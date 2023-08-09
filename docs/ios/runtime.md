---
layout: default
title: Runtime
parent: iOS 相关
nav_order: 3
---

# Runtime

## 什么是 runtime, runtime 用来做什么的？

runtime 是一套底层的 C 语言 API，所有 OC 的元素都是通过这套 API 来执行的。比如说 类的定义、方法的调用等。Runtime 给 OC 赋予了非常强大、灵活的能力，使 OC 可以在运行阶段实现很多功能。

## Objective-C 方法的调用过程

OC方法调用可以分为三大阶段：**消息发送、消息动态解析、消息转发。**

1. 消息发送。OC 调用发送会被转变成 objc_msgSend() 函数的调用，这个调用过程是：
   
   1. 在消息接收着的类对象中的方法缓存中寻找是否有对应方法，为了提高搜索效率缓存使用的是散列表的数据结构。
   
   2. 如果在缓存中没有找到方法就会进入消息接收着的类对象的方法列表中寻找，如果能找到就会把方法缓存起来并且进行调用。
   
   3. 如果消息接收着的类对象的方法列表中也没有找到，就会沿着继承链（superClass）向上寻找，直到 NSObject，如果在这个过程中找到了方法就会缓存起来并调用。
   
   4. 通过上面的步骤如果仍然没有找到方法就会进入消息动态解析过程。

2. 消息动态解析。通过重写`(BOOL)resolveInstanceMethod:(SEL)sel` 在里面给消息接收着的类对象在运行时添加需要的方法。**当动态解析过程执行完成后会重新回到消息发送流程执行。** 

3. 消息转发。如果在动态解析阶段仍然不能处理这个方法，就会进入消息转发流程。转发过程中有两次机会处理消息。
   
   1. 指定一个可以处理消息的对象。`(id)forwardingTargetForSelector:(SEL)aSelector`
   
   2. 直接处理消息。给 `(NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`  方法返回一个**不为空**的方法签名，就可以在 `(void)forwardInvocation:(NSInvocation *)anInvocation` 方法中任意处理逻辑了。

## 消息缓存机制

*   Runtime为每个类(不是每个类实例)缓存了一个方法列表，该方法列表采用hash表实现，hash表的优点是查找速度快，时间为O(1)。
*   父类方法的缓存只存在父类么，还是子类也会缓存父类的方法？
    子类会缓存父类的方法。
*   类的方法缓存大小有没有限制？
    在objc-cache.mm有一个变量_class_slow_grow定义如下：

```
/* When _class_slow_grow is non-zero, any given cache is actually grown
 * only on the odd-numbered times it becomes full; on the even-numbered
 * times, it is simply emptied and re-used.  When this flag is zero,
 * caches are grown every time. */
static const int _class_slow_grow = 1;

```

注释中说明，当_class_slow_grow是非0值的时候，只有当方法缓存第奇数次满（使用的槽位超过3/4）的时候，方法缓存的大小才会增长（会清空缓存，否则hash值就不对了）；当第偶数次满的时候，方法缓存会被清空并重新利用。 如果_class_slow_grow值为0，那么每一次方法缓存满的时候，其大小都会增长。
所以单就问题而言，答案是没有限制，虽然这个值被设置为1，方法缓存的大小增速会慢一点，但是确实是没有上限的。

*   为什么类的方法列表不直接做成散列表呢，做成list，还要单独缓存？
    1、散列表是没有顺序的，Objective-C的方法列表是一个list，是有顺序的；Objective-C在查找方法的时候会顺着list依次寻找，并且category的方法在原始方法list的前面，需要先被找到，如果直接用hash存方法，方法的顺序就没法保证。
    2、list的方法还保存了除了selector和imp之外其他很多属性
    3、散列表是有空槽的，会浪费空间
    参考资料：[深入理解 Objective-C：方法缓存](https://tech.meituan.com/2015/08/12/deep-understanding-object-c-of-method-caching.html)
    
## 关于 super 关键字

使用 `super` 调用方法本质上还是给当前类对象发送消息，只不过在搜索方法时是从父类的方法列表开始搜索而不是当前类的方法列表.

如果当前类和父类都没有对应方法的实现，而是到更高的父类中才找到实现那么通过 self 调用方法就和通过 super 调用方法没有区别了。

```objectivec
// ViewController : UIViewController
NSLog(@"[self class] = %@", [self class]); // ViewController
NSLog(@"[super class] = %@", [super class]); // ViewController
    
NSLog(@"[self superclass] = %@", [self superclass]); // UIViewController
NSLog(@"[super superclass] = %@", [super superclass]); // UIViewController
```

<u>因为 class 和 superclass 方法都是在 NSObject 中实现的，而实现的方法都是通过方法接收者的 isa 或者 superclass 指针来获取返回值，同时 super 调用方法的接受者也是 self，所以才会出现上面的结果。</u>

## 扩展
1.编译时决定   
2.以时声决明议形式存在，多数情况寄生于宿主类的.m中   
3.不能为系统类添加扩展。   

## 分类
1.运行时决定。  
2.可以为系统添加分类。  
3.分类中可以添加一下内容：    
  &emsp;&emsp;1.实例方法  
  &emsp;&emsp;2.类方法   
  &emsp;&emsp;3.协议   
  &emsp;&emsp;4.属性（自己实现get、set方法，并通过关联对象添加）
  
## 分类加载时机

*   在App加载时（将 Mach-0 相关 sections 都加载到内存之后），Runtime会把Category的实例方法、协议以及属性添加到类上；把Category的类方法添加到类的metaclass上。
*   category的方法没有“完全替换掉”原来类已经有的方法，如果category和原来类都有methodA，那么category附加完成之后，类的方法列表里会有两个methodA。
*   category的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的category的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的，它只要一找到对应名字的方法，就会停止查找，殊不知后面可能还有一样名字的方法。
  
##分类的实现机制和原理
1通过runtime加载某个类的所有Category数据, 底层是通过 unattachedCategoriesForClass 方法，获取到所属类的分类集合。
2.调用attachCategories，初始化方法列表、属性列表、协议列表的二维数组。
3.倒序遍历分类集合，取出分类的方法列表添加到二维数组中。所以后编译的分类方法会先被找到。
4.获取所属类的 rw 信息，取出所属类的方法列表，将二维数组插入到所属类的方法列表的第0位。
   ```底层代码 C++
static void attachCategories(Class cls, category_list *cats, bool flush_caches)
{
    // 判断是类对象还是元类对象
    bool isMeta = cls->isMetaClass();
    // 方法数组
    method_list_t **mlists = (method_list_t **)
        malloc(cats->count * sizeof(*mlists));
    // 属性数组
    property_list_t **proplists = (property_list_t **)
        malloc(cats->count * sizeof(*proplists));
    // 协议数组
    protocol_list_t **protolists = (protocol_list_t **)
        malloc(cats->count * sizeof(*protolists));
    // Count backwards through cats to get newest categories first
    int mcount = 0;
    int propcount = 0;
    int protocount = 0;
    int i = cats->count;
    bool fromBundle = NO;
    while (i--) {
        // 取出某个分类
        auto& entry = cats->list[i];
        // 取出分类中的对象方法或者类方法
        method_list_t *mlist = entry.cat->methodsForMeta(isMeta);
        if (mlist) {
            mlists[mcount++] = mlist;
            fromBundle |= entry.hi->isBundle();
        }
        property_list_t *proplist = 
            entry.cat->propertiesForMeta(isMeta, entry.hi);
        if (proplist) {
            proplists[propcount++] = proplist;
        }
 
        protocol_list_t *protolist = entry.cat->protocols;
        if (protolist) {
            protolists[protocount++] = protolist;
        }
    }
    // 得到类对象或者元类对象里面的数据
    auto rw = cls->data();
    prepareMethodLists(cls, mlists, mcount, NO, fromBundle);
    // 将所有分类的对象方法（类方法）附加到类对象（元类对象）的方法列表中
    rw->methods.attachLists(mlists, mcount);
    free(mlists);
    if (flush_caches  &&  mcount > 0) flushCaches(cls);
    // 将所有分类的属性附加到类对象的属性列表中
    rw->properties.attachLists(proplists, propcount);
    free(proplists);
    // 将所有分类的协议附加到类对象的协议列表中
    rw->protocols.attachLists(protolists, protocount);
    free(protolists);
}
   ```
## 类别添加属性、方法

*   在类别中不能直接以@property的方式定义属性，OC不会主动给类别属性生成setter和getter方法；需要通过objc_setAssociatedObject来实现。

## 关联对象
关联对象由全局AssociationsManager管理，储存在AssociationsHashMap中，不同类添加关联对象都是由AssociationsManager管理。
AssociationsHashMap 是双层嵌套结构。

![](https://pic.imgdb.cn/item/64d333881ddac507cc6cdbc2.jpg)

