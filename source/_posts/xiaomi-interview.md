---
title: 小米面试总结
date: 2017-05-28 13:25:30
category: summarize
tags: [工作,面试,总结]
---

上周面了一下小米的金融，不管结果如何，觉得还是有必要对其中涉及到知识点及问题进行总结。

### Object类中的方法  
* +equals():Boolean   
 原生的equals比较的是两个对象的引用是否相同，换而言之就是比较对象的内存地址是否一致
 
* +hashcode():int
 通过散列算法计算对象的hash值  
 拓展：解决hash冲突的方式有哪些？
 
* +toString():String  
 返回对象的字符串表示方式
 
* +wait()/wait(long time)/wait(long time,int nanos):void
 使当前线程让出CPU资源进入等待状态，
 当其他线程调用该对象的notify/notifyAll方法时跳出等待状态
 
* +notify():void
 唤醒监听这个对象的一个线程

* +notifyAll():void  
 唤醒监听这个对象的所有线程

* +getClass():Class
 返回当前对象的runtime class
 
* clone:Object
 返回当前对象的副本，使浅克隆。深克隆的话可以用序列化及反序列化方式
 
* finalize:void
 当对象将被垃圾收集器回收的时候，阻止垃圾回收器回收，该方法只能使对象存活一次 
 
### 保证线程安全的方式有哪些以及他们之间的区别？  
 目前保证线程安全的三种方式  
    1.用volatile修饰的变量  
    2.用Reetrantlock 即lock和unlock修饰的代码块
    3.用synchronized修饰的方法或者代码块  
 volatile:  
    使用该关键字修饰的，访问变量时告诉jvm需要从主内存中获取，保证可见性，不会造成线程阻塞；
 synchronized:
    是jvm级的安全锁，保证可见性及原子性，使用时如果发生中断异常会主动释放当前锁，会造成线程阻塞；
 Reetrantlock:
    代码级安全锁，保证可见性及原子性，使用时不会主动释放当前锁，需要手动释放，会造成线程阻塞
    
 1）用法区别
     synchronized(隐式锁)：在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。
     lock（显示锁）：需要显示指定起始位置和终止位置。一般使用ReentrantLock类做为锁，多个线程中必须要使用一个ReentrantLock类做为对 象才能保证锁的生效。且在加锁和解锁处需要通过lock()和unlock()显示指出。所以一般会在finally块中写unlock()以防死锁。
 2）synchronized和lock性能区别
     synchronized是托管给JVM执行的，而lock是java写的控制锁的代码。
     synchronized采用的是CPU悲观锁机制，即线程获得的是独占锁。独占锁意味着其 他线程只能依靠阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。
     Lock用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就 是CAS操作（Compare and Swap）。
 3）synchronized和lock用途区别
     synchronized原语和ReentrantLock在一般情况下没有什么区别，但是在非常复杂的同步应用中，请考虑使用ReentrantLock，特别是遇到下面2种需求的时候。
     1.某个线程在等待一个锁的控制权的这段时间需要中断
     2.需要分开处理一些wait-notify，ReentrantLock里面的Condition应用，能够控制notify哪个线程
     3.具有公平锁功能，每个到来的线程都将排队等候、
     
### 写一个生产消费模型

### 二分查找递归及非递归写法

### 用两个stack模拟一个queue

### 简单的谈谈jvm

### 项目中遇到的难点及解决方式

## 总结
 整个面试的过程中，发现对基础的知识的提问相对于自己参与项目的提问整体时间所占比例大幅下降
 由于更加专业化，面试官在提问的过程中也着重于他们自己的业务。所以整体感觉，
 如果他们问的不是你自己感兴趣或者职业侧重点，不管是不是当前阶段的知识盲区，
 最后能不能拿到offer都可以在某种程度上作为自己选择接不接offer的一个参考点。