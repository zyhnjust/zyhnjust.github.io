---
title: arch_ms
date: 2018-09-22 15:00:08
tags:
- 面试
categories: 
- 面试
- 架构
---
消息，hashtable非，分布式缓存， 缓存穿透
<!-- more -->


# 列举一个常用的消息中间件， 消息保序如何实现

比如kafka消息队列， 我们用的三个节点的。 zk来控制他们的集群状态，还有offside。 
？ 怎么个流程呢。 
- producer到broker的过程是push，也就是有数据就推送到broker，而consumer到broker的过程是pull，是通过consumer主动去拉数据的，而不是broker把数据主动发送到consumer端的。

简单说下整个系统运行的顺序：

1. 启动zookeeper的server
2. 启动kafka的server
3. Producer如果生产了数据，会先通过zookeeper找到broker，然后将数据存放进broker
4. Consumer如果要消费数据，会先通过zookeeper找对应的broker，然后消费。

- Kafka：天然的‘Leader-Slave’无状态集群，每台服务器既是Master也是Slave。分区首领均匀地分布在不同的kafka服务器上，分区副本也均匀地分布在不同的kafka服务器上，所以每一台kafka服务器既含有分区首领，同时又含有分区副本，每一台kafka服务器是某一台kafka服务器的Slave，同时也是某一台kafka服务器的leader。kafka的集群依赖于zookeeper，zookeeper支持热扩展，所有的broker、消费者、分区都可以动态加入移除，而无需关闭服务，与不依靠zookeeper集群的mq相比，这是最大的优势。


## 如何保序
- 问题由来： 就是不一个分区自然。 

用一个partition。 一个分区。 但是在多个Partition时，不能保证Topic级别的数据有序性。
肯定一个topic， 这个没说的。 

## 其他呢
- redis 也可以消息队列
- redis 消息推送（基于分布式 pub/sub）多用于实时性较高的消息推送，并不保证可靠。

- MQ

## kafka
首先， 消息 kafka。 消息队列。 
是一个集群， 它是用zk的集群来
- 可以看到，三个节点上的消费者都能正常的接收到其中一个节点上发送的消息。这说明kafka集群基本上已经超过部署。
- zk 可以在里面的也可以外面独立不熟的
- Broker（代理）

Kafka集群通常由多个代理组成以保持负载平衡。 Kafka代理是无状态的，所以他们使用ZooKeeper来维护它们的集群状态。 一个Kafka代理实例可以每秒处理数十万次读取和写入，每个Broker可以处理TB的消息，而没有性能影响。 Kafka经纪人领导选举可以由ZooKeeper完成。

http://www.open-open.com/lib/view/open1354277579741.html

# history
0920 需要给一个简短的update， 下面可以继续安排一下。 估计两个小时。 


下面 
1. 需要读这本书 - 大型网站技术架构
2. https://zh.wikipedia.org/wiki/%E4%B8%80%E8%87%B4%E5%93%88%E5%B8%8C

