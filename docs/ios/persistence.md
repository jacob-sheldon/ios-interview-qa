---
layout: default
title: 持久化
parent: iOS 相关
nav_order: 6
---

# 持久化    

## NSCache

通常使用 `NSCache` 对象临时存放创建起来非常昂贵的临时对象。

- **包含自动淘汰机制可以确保不会使用太多的系统内存**。当系统内存紧张可以利用这些策略来移除一些 cache，最小化内存峰值
- **线程安全**。可以在多个线程添加、删除、查询缓存内容。
- 与`NSMutableDictionay`不同，cache 不会复制放入其中的 key objects。

## FMDB 线程安全（读、写），数据库当需要给一个表增加、减少字段时怎么做，版本迁移。

- `SQLite`可以配置为是否线程安全，与`FMDB`多线程安全是两回事
- `SQLite`默认的线程模式是串行模式，是线程安全的
- `FMDatabase`多线程不安全，单个`FMDatabaseQueue`是多线程安全的。
- `FMDatabaseQueue`是一个串行队列，所以可以做到线程安全。但是也不能在多个线程使用同一个连接，一个线程使用完后要调用 close 关闭这个连接之后，另一个线程才能使用。

## `SQLite`的线程配置
- 0 - Single-thread, 编译时所有互斥锁代码都会被删除掉，SQLite不是线程安全的。
- 1 - Serialized，串行，是线程安全的，默认选项
- 2 - Multi-thread, 大多数情况下线程安全，不安全情况：有多个线程同时尝试使用相同的数据库连接。

除了在编译时指定 threading mode，还可以在 start time 进行配置或者在run time 配置.

**系统自带的 SQLite 在不做任何配置的情况下不是完全线程安全的。**

