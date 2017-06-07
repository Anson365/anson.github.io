---
title: java concurrent解析
date: 2017-06-07 22:51:59
tags:
---
***Executor*** :抽象的Runnable任务的执行者，提供一个excute方法  
***ExecutorService*** :抽象的线程池管理者，包含三种状态：运行、关闭、终止。当调用了shutdown()方法时，便进入了关闭状态，ExcecutorService
不在接受新的任务，当已经提交的任务完成后便达到终止状态。  
***ReentrantLock*** :一个可重入得Lock代码层面的并发锁，需手动释放
***Future*** :一个Callable线程执行结束后返回的结果，提供isDone检查call方法是否执行完毕，cancel终止线程
***BlockingQueue*** :阻塞队列，常见ArrayBlockQueue,LinkedBlockQueue前者有容量限制，后者没有限制  
