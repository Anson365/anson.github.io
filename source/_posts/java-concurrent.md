---
title: java concurrent解析
date: 2017-06-07 22:51:59
category: summarize
tags: [java,concurrent,多线程,Executor]
---
***Executor*** :抽象的Runnable任务的执行者，提供一个excute方法  
***ExecutorService*** :抽象的线程池管理者，包含三种状态：运行、关闭、终止。当调用了shutdown()方法时，便进入了关闭状态，ExcecutorService
不在接受新的任务，当已经提交的任务完成后便达到终止状态。  
***ReentrantLock*** :一个可重入得Lock代码层面的并发锁，需手动释放
***Future*** :一个Callable线程执行结束后返回的结果，提供isDone检查call方法是否执行完毕，cancel终止线程
***BlockingQueue*** :阻塞队列，常见ArrayBlockQueue,LinkedBlockQueue  
***CompletionService*** :ExecutorService的扩展，可以获得线程执行结果的 
***CountDownLatch*** :一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待  
***CyclicBarrier*** :一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 
***ScheduledExecutorService*** :一个 ExecutorService，可安排在给定的延迟后运行或定期执行的命令  
***Semaphore*** :一个计数信号量 
***Executors*** :提供了一系列工厂方法用于创先线程池，返回的线程池都实现了ExecutorService接口

#### Executors中几种方法的对比  

| 方法名称 |  缓存队列 | 方法解析 |
| :------- | :------- | :------|
|newCachedThreadPool() | SynchronousQueue | 1.缓存型池子，先查看池中有没有以前建立的线程，如果有，就 reuse.如果没有，就建一个新的线程加入池中  2.缓存型池子通常用于执行一些生存期很短的异步型任务因此在一些面向连接的daemon型SERVER中用得不多。但对于生存期短的异步任务，它是Executor的首选。3.能reuse的线程，必须是timeout IDLE内的池中线程，缺省timeout是60s,超过这个IDLE时长，线程实例将被终止及移出池。注意，放入CachedThreadPool的线程不必担心其结束，超过TIMEOUT不活动，其会自动被终止。|
|newFixedThreadPool(int) | LinkedBlockingQueue |1.newFixedThreadPool与cacheThreadPool差不多，也是能reuse就用，但不能随时建新的线程 2.其独特之处:任意时间点，最多只能有固定数目的活动线程存在，此时如果有新的线程要建立，只能放在另外的队列中等待，直到当前的线程中某个线程终止直接被移出池子 3.和cacheThreadPool不同，FixedThreadPool没有IDLE机制（可能也有，但既然文档没提，肯定非常长，类似依赖上层的TCP或UDP IDLE机制之类的），所以FixedThreadPool多数针对一些很稳定很固定的正规并发线程，多用于服务器 4.从方法的源代码看，cache池和fixed 池调用的是同一个底层 池，只不过参数不同:fixed池线程数固定，并且是0秒IDLE（无IDLE）cache池线程数支持0-Integer.MAX_VALUE(显然完全没考虑主机的资源承受能力），60秒IDLE|
|newScheduledThreadPool(int) | DelayedWorkQueue |1.调度型线程池 2.这个池子里的线程可以按schedule依次delay执行，或周期执行|
|newSingleThreadPool() | LinkedBlockingQueue |1.单例线程，任意时间池中只能有一个线程 2.用的是和cache池和fixed池相同的底层池，但线程数目是1-1,0秒IDLE（无IDLE）|

#### [BlockingQueue几种实现类](http://www.infoq.com/cn/articles/java-blocking-queue)

| 类名称 | 功能描述|
| :------| :------|
|ArrayBlockingQueue|由数组结构组成的有界阻塞队列,其构造函数必须带一个int参数来指明其大小.其所含的对象是以FIFO(先入先出)顺序排序的|
|LinkedBlockingQueue|一个由链表结构组成的有界阻塞队列，如果没指定队列大小，则默认为Integer.Max,遵从FIFO|
|PriorityBlockingQueue|一个支持优先级排序的无界阻塞队列,PriorityBlockingQueue是对PriorityQueue的再次包装，是基于堆数据结构的，而PriorityQueue是没有容量限制的,可能会导致 OutOfMemoryError,往入该队列中的元素要具有比较能力|
|DelayQueue|一个使用优先级队列实现的存放Delayed 元素的无界阻塞队列,只有在延迟期满时才能从中提取元素。该队列的头部是延迟期满后保存时间最长的 Delayed 元素|
|SynchronousQueue|不存储元素，特殊的阻塞队列,对其的操作必须是放和取交替完成的|
|LinkedBlockingQueue|一个由链表结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法.transfer方法。如果当前有消费者正在等待接收元素（消费者使用take()方法或带时间限制的poll()方法时），transfer方法可以把生产者传入的元素立刻transfer（传输）给消费者。如果没有消费者在等待接收元素，transfer方法会将元素存放在队列的tail节点，并等到该元素被消费者消费了才返回|
|LinkedBlockingDeque|是一个由链表结构组成的双向阻塞队列,所谓双向队列指的你可以从队列的两端插入和移出元素。双端队列因为多了一个操作队列的入口，在多线程同时入队时，也就减少了一半的竞争。相比其他的阻塞队列，LinkedBlockingDeque多了addFirst，addLast，offerFirst，offerLast，peekFirst，peekLast等方法|

***BlockingQueue的常用方法***：

> add        增加一个元索                     如果队列已满，则抛出一个IIIegaISlabEepeplian异常
>remove   移除并返回队列头部的元素    如果队列为空，则抛出一个NoSuchElementException异常
>element  返回队列头部的元素             如果队列为空，则抛出一个NoSuchElementException异常
>offer       添加一个元素并返回true       如果队列已满，则返回false
>poll(time)         移除并返问队列头部的元素    如果队列为空，则返回null
>peek       返回队列头部的元素             如果队列为空，则返回null
>put         添加一个元素                      如果队列满，则阻塞
>take        移除并返回队列头部的元素     如果队列为空，则阻塞






