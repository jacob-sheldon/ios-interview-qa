---
layout: default
title: 多线程
parent: iOS 相关
nav_order: 8
---

# 多线程

## 多线程的实现方式

pthread：类 Unix 系统提供的操作系统级多线程处理方式，基本用不到

NSThread：iOS 系统对 pthread 的封装，提供面向对象的操作方式，需要自己创建和开启线程

GCD：自动管理线程的生命周期，根据系统的 CPU 及其负载进行任务调度。

NSOperation、NSOperationQueue：分别对应 GCD 的任务和队列。**提供了 cancel 操作和设置依赖的功能。**

## 同步、异步 串行、并行

异步执行 - `dispatch_async` 同步执行 - `dispatch_sync`

串行队列 - `DISPATCH_QUEUE_SERIAL` 并行队列 - `DISPATCH_QUEUE_CONCURRENT`

异步任务会开启新的线程，同步任务会在当前线程。

异步串行多个任务会在一个开启的新线程中依次执行，异步执行并行任务会开启多个线程乱序执行。

只要是同步任务（不论是并行还是串行）都会按照添加顺序在当前线程依次执行；只要是异步任务（不管是并行还是串行）都不会阻塞当前线程。

*当在主线程同步在主队列执行任务是会死锁。*

## 多线程的弊端和解决方案

```
1. 占用内存，上下文需要更新寄存器等操作需要一定耗时
2. 产生线程竞争：多个线程共享资源可能会产生与预期不同的结果
3. 锁：为了解决线程竞争的问题需要加锁，但是加锁会有性能损失
4. 引起 crash：多线程操作集合对象时可能引起crash，可以通过多读单写来处理（dispatch_barrier）
```

## 实现线程同步的方法

    1. 加锁。OSSpinLock、pthread_mutex、NSLock、
            NSCondition、NSConditionLock、
            @synchronize、NSRecursiveLock
    2. 信号量。dispatch_semaphore_t
    3. 使用串行队列。dispatch_queue_create(Serial)

## 多线程锁

|         | 自旋锁                      | 互斥锁                                  |
| ------- | ------------------------ | ------------------------------------ |
| 概念      | 一直等待资源释放，死循环，处于忙等状态      | 当无法获取需要资源时会进入休眠状态，直到资源可用才会被唤起        |
| 优势      | 效率高，因为不会休眠，没有唤起和上下文切换的消耗 | 不需要一直占用CPU，在等待过程中 CPU 可以进行其他工作       |
| 劣势      | 一直占用 CPU，                | 需要进行唤起和上下文切换                         |
| 使用场景    | 被锁住的代码执行时间较长             | 被锁住的代码执行时间很短                         |
| iOS 中的锁 | OSSpinLock               | pthread_mutex/NSLock/NSRecursiveLock |

1. `pthread_mutux` 设置 PTHREAD_MUTEX_RECURSIVE 后就成了递归锁，`NSLock `是对 `pthread_mutux` 普通锁的封装，`NSRecusiveLock` 是对 `pthread_mutux` 递归锁的封装。

2. 递归锁：允许**同一线程**对同一把锁进行重复加锁。

## 多种锁的原理

1. `NSLock` 是对 `pthread_mutex` 的封装，所以 NSLock 是一个互斥锁。

2. `@synchronized()` ：runtime 维护了一个单向链表，链表的节点是 `SyncData`，加锁和解锁时会根据传入的对象遍历链表找到 `SyncData`，之后获取到 SyncData 中的 `recursive_mutex_t` 这样就可以使用这个递归锁进行加锁和解锁了。

3. `NSCondition` 是对`pthread_mutex` 和 `pthread_cond_t` 的封装；而 `NSConditionLock` 是对 `NSCondition` 的封装

## atomic 真的安全么？是怎么实现的？用的哪种锁？

原子性并不能保证线程安全. 只是相对运用了原子性keyword 的属性来说是线程安全的. 对于类来说则不一定.

**atomic所说的线程安全只是保证了getter和setter存取方法的线程安全，并不能保证整个对象是线程安全的**

比如对一个 MutableArray 设置了 atomic，只能确保多个线程在 set 和 get 这个属性的时候是线程安全的，但是如果有多个线程在同时操作这个数组（添加、删除元素），那就不是线程安全的了。

Runtime 中有一个全局数组保存了属性锁 —— 这个锁是 `spinlock_t` 的自旋锁，根据属性在实例中的位置获取到这把锁，加锁后才能操作设置或者获取值，之后再释放锁。

## 线程和队列的关系

- 线程是计算机的操作系统用来执行任务的单元
- 队列是用来存储并分发任务，队列是一种数据结构

## 信号量的使用

- 创建信号量 `dispatch_semaphore_create(0)` 参数是初始化的信号数量
- 减少信号量 `dispatch_semaphore_wait(sem, 10)` 超过了设定的时间会返回非0，并**继续往下执行**，如果减一后信号量是负数就会卡死在当前线程。
- 增加信号量 `dispatch_semaphore_signal(sem)` 

## 多线程的实际应用场景（几个需求的实现方法）

- 监听多个异步操作的完成，之后统一执行某操作
    - dispatch_group_async + dispatch_group_notify

    - dispatch_barrier_async

- 读写分离（多读单写）
    - pthread_rwlock_t（读写锁）

    - dispatch_barrier_async

- 控制并发数量
    - dispatch_semphore(n)

    - NSOperationQueue.maxConcurrent

- 多个异步操作串行执行 
    - dispatch_queue(Serial) 
    - dispatch_semaphore(1)
    - NSOperation 设置依赖


## 使用多线程的注意点

- `dispatch_barrier_async()` 必须传入自己创建的并发队列，如果使用全局并发队列就相当于使用的是 `dispatch_async()`

- 子线程默认不会开启 runloop，也就是说线程内任务串行执行完成后该线程就停止了，需要依赖 runloop 才能进行的任务不会被执行，比如 事件、定时器、界面刷新等

- `performSelector:` 方法相当于直接进行方法调用；`performSelector:afterDelay` 使用了定时器需要依赖 runloop，**哪怕 `afterDeley: 0`**，不会开新线程只是加了定时器，**selector 会在当前线程执行**。

- `NSThread` 取消任务需要调用 `- (void)cancel` 方法

- 只有主线程默认带有自动释放池，子线程需要手动添加。

- 多线程调用`nonatomic` 修饰的 OC 对象的 setter 方法时会 crash，因为会出现给僵尸对象发消息。引用计数中会在 OC 对象的 setter 方法中先释放旧值，这就可能出现一个线程把旧值释放了，另外一个线程又释放 `release` 的情况。
