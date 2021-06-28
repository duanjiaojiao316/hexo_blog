---
title: 初始Kafka
categories:
  - 后端
  - 大数据
tags:
  - Kafka
translate_title: initial-kafka
date: 2021-01-08 21:56:47
---


1、Kafka的定义

首先了解Kafka是什么，在哪些地方扮演着什么角色？

`kafka`是一个多分区多副本基于`ZooKeeper`协调的发布/订阅模式分布式消息系统，具有高吞吐量，可持久化，可水平扩展，支持流数据处理特性，`kafka`主要用于：消息系统，存储系统，流式处理平台。

Kafka与其他的消息中间件类似，具备系统解耦、冗余存储、流量削峰、缓冲、异步通信、扩展性等功能，同时还实现了消息顺序性保障以及消息回溯功能。

Kafka在存储系统，将消息持久化到磁盘，相比于其他的内存存储的系统降低数据丢失的风险，主要得益于Kafka的持久化功能多副本机制。

在流式处理平台，为流式处理框架提供可靠的数据来源，提供流式处理类库。

2、Kafka的体系架构

Kafka主要由三个部分：生产者`Producer`、消费者`Consumer`、服务代理节点`Broker`。

生产者创建消息，将消息发送到Kafka；消费者从Kafka拉取消息，接收消息；broker是Kafka的服务节点，如果Kafka中只有一个broker，也可以看做Kafka服务器。多个broker组成Kafka集群。

Kafka中以主题为单元进行存储，一个主题可以划分为多个分区，一个分区只能属于一个主题，分区在存储层面可以看做Log日志文件，把消息附加到分区中会分配到一个offset偏移量，Kafka通过偏移量保证消息的顺序性。注意offset不跨越分区。所以Kafka中保证分区内消息有序而不是主题有序。

![](%E5%88%9D%E5%A7%8BKafka/Kafka%E4%BD%93%E7%B3%BB%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

从图中可以看出分区副本，Kafka引入分区的多副本机制，可以提高容错能力，但是同一时刻不同副本之间的消息并非完全相同，副本之间是一主多从的关系，leader负责读写请求，follower副本只负责和leader消息同步。不同副本处于不同的broker，如果leader出现故障，可以从follower中重新选举新的leader。

在分区中所有的副本统称为AR，与leader保持一定程度同步的副本组成ISR，一定程度表示可以忍受的容错范围，这个指标可以通过参数指定，与leader具有过多之后的副本组成OSR。所以`AR=ISR+OSR`

同时leader负责维护和跟踪ISR中所有副本的滞后状态，如果follower滞后较多，leader会将他从ISR中剔除，如果OSR中副本没有较多的滞后，会把它转移到ISR中。如果leader发生故障，只会从ISR中选举leader，OSR中的follower没有机会选举为新的leader。

为了进一步了解ISR和OSR，需要了解HW和LEO。

HW（ High Watermark）高水位。

LEO（Log End Offset）当前日志下一条消息的offset。

![image-20210112221809096](%E5%88%9D%E5%A7%8BKafka/image-20210112221809096.png)

LEO是分区中最后一条消息offset的值+1，ISR集合中最小的LEO为HW，消费者只能消费HW之前的消息。