---
layout:     post
title:      面试问答(二)
subtitle:   面经 问答
date:       2024-05-10
author:     Belial
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - 面试
    - Android
    - flutter
---

# 面试相关题

#### 1. 深拷贝与浅拷贝

浅拷贝：基本数据类型是值传递拷贝，对引用数据是引用地址传递，指针指向共同的地址。
深拷贝：基本数据类型与浅拷贝一致值传递，对于引用数据类型，创建一个新的对象，内容拷贝

#### 2. Error 与 Exception  

Error：是程序运行中不可预料的异常情况，这种异常会直接导致 jvm不可处理或不可恢复，这种异常是不可被抓取，**OOM，ANR**(`5s`没有响应输入的事件或`BordercastReceiver`在 `10s`内有没执行完毕)，`InputDispatching Timeout`: 输入事件分发超时`5s`，包括按键和触摸事件。
Exception：在项目中是可预料的异常情况，可以别捕获，并根据业务情况处理异常。

#### 3.Java线程池的好处

1.线程的重用：线程的创建与销毁都是比较消耗性能的操作，而减少了这些开销，线程执行也就会有很大的优化流畅
2.控制管理线程的并发，避免出现死锁

#### 4.android Activity 如何保存状态

以下几种情况会是`activity` 状态丢失

1. 手机翻转，配置发生修改
2. 系统自动销毁 `Activity`，内存不足时
3. 手动销毁 `Activity` 返回按键

保存数据方式：Android 可以使用 `ViewModel` 和 `saveInstanceState()`来保存数据.
`ViewModel`会将数据存储在内存中，`ViewModel` 与 `Activity` 绑定，当 `activity`，配置更改后产生新的 `activity` 实例系统自动与它的 ViewModel关联。
当系统销毁 `Activity` 调用 `onSaveInstanceState`，将`ViewModel`存储在 Bundle 中，当 `activity` 创建时，通过`Bundle` 对象中的数据恢复原来的值

#### 5.HashMap

hashMap 是 Map的实现类，用来存储key-value 键值对的集合，hashMap不保证数据由有序性，hashmap 允许键和值为null，hashMap 是基于 hash表实现的数据结构，具有快速高效的插入、查找、删除的特性。
hash表数组是有限的，所以会出现哈希冲突的情况，处理哈希冲突的方式是使用 数组+链表的方式存储，当链表长度大于 `8`（默认）时，将转换成红黑树数据格式存储，以便提高查找效率
解决哈希冲突有三种方式

- **扩容方式**
- **开放寻址**：线性探测 平方探测 多次哈希
- **链式地址**
> 扩容与链式地址都是属于扩容方式，链式变为红黑树大大增加查询效率

#### 6.volatile

多线程并发中操作的三大特性

- 原子性 对数据类型读写操作，不可中断，要么执行完成要么不执行
- 可见性 用 `volatile` 修饰变量时，强制将变量数值刷新到主内存中，使其他线程实时可见
- 有序性 `JMM` 中只要结果一直，允许指令重排

保证操作的可见性，不保证原子性， `synchronized` 关键字可以同时满足上面三种特性
当只有一个线程时，其他线程只读，可以使用 **volatile** 修饰，当多个线程并发不严重时可以使用 synchronized，
什么场景适用 `volatile`： 1. 状态量标记使用 `volatile` ，比 `synchronized` 高效 2. 单列模式，双重锁检查锁定

#### 7.POST 与 GET

1. 这两种网络请求方式再实现功能上是一样的，在技术上给 Get 请求添加 `Requestbody` 和 Post 在 Url 添加参数都是可行的，只是有些服务器浏览器的框架都约定俗成，Post 参数放在Body Get 参数放在 URL，http 本身时没有这样的限制的
2. 效率与安全方面：Get 执行请求效率搞但是安全性很低，而 `Post` 相反
3. Get 向服务端一次性发送`Header` 与 `data`，响应 200，而 `Post` 时先向服务端发生 `Header`，服务端返回 100，然后发送 `data`
4. 本质意义上，`Get` 不会对服务端内容做出修改覆盖等内容，更安全，而 `Post` 则会，即 `Get` 具有幂等性，而 `Post` 不具有

#### 8.Handler

Handler是 Android 开发中实现线程间通信的重要的工具，或切换线程使用
Handler机制是Android中基于单线消息队列模式的一套线程消息机制。

- 切换代码执行的线程
- 按顺序规则地处理消息，避免并发
- 阻塞线程，避免让线程结束
- 延迟处理消息

如何使用Handler 每个线程只有一个`Looper`，通常主线程已经创建好了，追溯应用程序启动流程可以知道启动过程中调用了`Looper.prepareMainLooper`，而在子线程就必须使用如下方法来初始化`Looper`：

```java
Looper.prepare();
```

1. 创建Looper
2. 使用Looper创建Handler
3. 启动Looper
4. 使用Handler发送信息

`Handler`机制内部有*三大关键角色*：**Handler**，**Looper**，**MessageQueue**。其中`MessageQueue`是`Looper`内部的一个对象，MessageQueue和Looper每个线程有且只有一个，而Handler是可以有很多个的。他们的工作流程是：

用户使用线程的Looper构建Handler之后，通过`Handler`的send和post方法发送消息消息会加入到`MessageQueue`中，等待Looper获取处理Looper会不断地从`MessageQueue`中获取`Message`然后交付给对应的Handler处理。

#### 9.glide 三级缓存

Glide的缓存机制，主要分为2种缓存，

- 一种是内存缓存
- 一种是磁盘缓存

> 1. 之所以使用内存缓存的原因是：防止应用重复将图片读入到内存，造成内存资源浪费。
> 2. 之所以使用磁盘缓存的原因是：防止应用重复的从网络或者其他地方下载和读取数据。正是因为有着这两种缓存的结合，才构成了Glide极佳的缓存效果。

读取一张图片的时候，获取顺序： **Lru算法缓存->弱引用缓存->磁盘缓存**

当我们的APP中想要加载某张图片时，先去`LruCache`中寻找图片，如果`LruCache`中有，则直接取出来使用，并将该图片放入WeakReference中，如果LruCache中没有，则去`WeakReference`中寻找，如果`WeakReference`中有，则从`WeakReference`中取出图片使用，如果`WeakReference`中也没有图片，则从磁盘缓存/网络中加载图片。
> 注：图片正在使用时存在于 `activeResources` 弱引用map中