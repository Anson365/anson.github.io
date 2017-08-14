---
title: kafka常用指令集
date: 2017-08-14 22:39:53
categories: study
tags: [学习,后端,中间件,大数据,kafka]
---

  在学习kafka的过程中，经常会用到一些console命令行。现在对这些命令行进行一个简单的汇总，方便今后查阅。  
  
### 启动zookeeper  

```
bin/zookeeper-server-start.sh config/zookeeper.properties  
```



### 启动broker  

```
bin/kafka-server-start.sh config/server.properties
```




### 创建TOPIC  

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```




### 查看topic list 

```
bin/kafka-topics.sh --list --zookeeper localhost:2181
```


   
### console producer  

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```



 
### console producer with output info  
 
```
bin/kafka-verifiable-producer.sh --topic test --max-messages 200000 --broker-list localhost:9092
```
