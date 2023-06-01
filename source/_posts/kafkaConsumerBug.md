---
title: kafka-clients 0.10.2.1 空poll导致频繁youngGC的问题剖析
date: 2023-06-01 17:20:50
tags:
---

## 一、问题复现
自研应用消费客户实时kafka信令消息， 发现eden区128M 数秒就会消耗完并频繁触发youngGC， 并加剧fullGC，
本文排查下到底kafkaConsumer 占用eden和什么相关
编写demo kafka用0.10.2.1版本，单线程单consumer while true poll(1000)下， 128M年轻代 平均60s 一次 youngGC， 并且总是突然的大量内存堆使用。  现象基本对的上。

![gc](./p1-jvmgc.png)

## 二、问题原因
首先我们google下类似问题，发现以下相关：https://issues.apache.org/jira/browse/KAFKA-5512 

概括bug：poll线程拉取消息时，为避免影响heartbeat线程，会将本次poll timeout时间设置为 Min(timeout, timeToNextHeartbeat) , 当timeToNextHeartbeat=0 时 client.poll(0)会非阻塞无停顿的读取数据。此时，若heartbeat线程仍处在wait( retryBackOff)(上一次检查还不需要发送心跳，则wait retryBackOffMs 等待下一次check)的情况下，这时一次poll过程会 while remaining>0 发起大量pollOnce 且 创建并遍历大量空数组导致频繁youngGC。 此bug被记录到github issue-3442, 并合入0.11.0版本代码

为了进一步证明问题，使用新老版本kafkaClients包测试相同问题, while(true) {empty poll(1000ms) }

![compare](./p2-compare.png)

结论：0.10.2.1 的bug， 已与 0.11.0+ 版本修复， 虽然得到了结论确实是低版本有bug， 但是我们没有自己排查到这个问题， 接下来我们根据提示， 自己尝试定位到问题。

## 三、JProfiler根因定位&源码分析
首先我们安装社区定位问题使用的 JProfiler，JProfiler通过javaagent对class进行了插桩，我们继续运行有问题的demo代码，开启cpu统计，查看org.apache.kafka包内所有方法调用次数

![jprofile1](./p3.png)

发现确实，业务层代码仅仅调用了5次consumer.poll，但是pollOnce方法执行了1446次，与社区反馈的问题现象一致。
heap中确实是出现了大量的Kafka内部对象及Collection Map等集合类，手动触发gc可全部被gc回收

![jprofile2](./p4.png)

poll & pollOnce问题源码定位：

![code1](./p5.png)

源码中可以看到，pollOnce在一个while循环中，只要没有到poll入参给定的timeout，则会无限重复执行pollOnce方法，也就是说一次poll调用，确实可能多次调用pollOnce方法，但pollOnce的入参传入的也是remaining，我们继续看pollOnce内部实现

![code2](./p6.png)

pollOnce方法中，有一行修改client pollTimeout的代码，它会将NetworkClient 向kafka集群 poll数据的阻塞时间 设置为 入参timeout 和 距离coordinator下一次发送心跳间隔时间 中更小的那个值，如果调用至此行代码，已经到了要发送heartbeat的时间的话，pollTimeout会被设为0， 最终在nio层面 执行 selector.poll(0) , 也就是selector 直接返回nio channel就绪数据， 不阻塞。此方法也就会瞬间返回结果， 并进入下一次while循环，如果heartbeat线程 还未发送heartbeat的话， 再次立即执行完进入while循环，继续pollOnce。 直至remaining结束 或者 heartbeat线程发送 heartbeat请求到 对应coordinator broker。

看到这，其实可以理解这行问题的代码用意，防止poll timeout阻塞过程中影响了 heartbeat线程发送心跳请求。

但是Heartbeat心跳线程的这个逻辑导致了bug的出现：

![code3](./p7.png)

Heartbeat线程负责consumerGroup中的consumer client和 kafka coordinator broker定期心跳，心跳间隔通过参数 heartbeatIntervalMs 配置， 线程while true判断是否到达interval并需要发送Heartbeat请求， 没到发请求时间就 wait(retryBackOffMs) 后重试。也就是在这个retryBackOffMs 内，会导致pollOnce一直poll(0)， 并发起大量pollOnce调用， 从而导致eden区usage突然陡增，导致频繁youngGC。

接下来我们通过示意图来说明bug出现的具体原因。

![consumer](./p8.png)

首先， 上边是一个kafka consumer的主要涵盖内容， 我们启动consumer消费线程的同时，如果使用了consumerGroup，kafka client会为consumerGroup分配一个coordinator broker，并将consumerGroup中的第一个consuemr作为leader 负责partition的assign、rebalance等工作。consumer和coordinator broker需要维持心跳来帮助判断每个consumerGroup中的consumer是否还存活，这个是靠后台Heartbeat线程来进行的。
当触发以下条件时，会发生上述BUG：

![bug](./p9.png)

## 四、问题修复
彻底修复：升级kafka-clients库 0.11+，社区修复方案： 当发现timeToNextHeartbeat=0时，会调用coordinator.notifyAll 唤醒Heartbeat立即发送心跳

无法升级kafka-clients的话，调参优化：
  + 如果没有大量的kafkaConsumer线程，则youngGC问题不会很明显，无影响则可以不修复。  
  + 如果你的生产者稳定生产消息，消费者每次poll过程基本都能消费到消息，则无影响，可以不修复。因为一担pollOnce拉到了数据就返回了。
  + 如果youngGC确实影响了你的应用，可以减少retryBackOffMS的大小，较小周期冲突时长。
  + poll消息如果结果为空，sleep一段时间，减小poll频率，如果业务可接受的话。

