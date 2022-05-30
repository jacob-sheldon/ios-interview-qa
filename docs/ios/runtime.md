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

