---
title: Kafka学习记录
date: 2019-01-22 21:27:00
comments: true
categories: [消息队列]
tags: [消息队列, Kafka]
---



<!-- more -->

## 基本概念

- Producer：消息和数据的生产者，向 Kafka 的一个 Topic 发布消息的进程/代码/服务
- Consumer：消息和数据的消费者，向 Kafka 订阅数据（Topic）并且处理其发布的消息的进程/代码/服务
- Consumer Group：逻辑概念，对于同一个 Topic，会广播给不同的 Group，一个 Group 中，只有一个 Consumer 可以消费该消息
- Broker：物理概念，Kafka 集群中的每个 Kafka 节点
- Topic：逻辑概念，Kafka 消息的类别，对数据进行区分、隔离
- Partition：物理概念，Kafka 下数据存储的基本单元。一个 Topic 的数据会被分散存储到多个 Partition，每一个 Partition 是有序的。一个 Partition 只能存在于一个 Broker 上
- Replication：同一个 Partition 可能会有多个 Replica（副本），多个 Replica 之间数据是一样的
- Replication Leader：一个 Partition 的多个 Replica 上，需要一个 Leader 负责该 Partition 上与 Producer 和 Consumer 交互
- ReplicaManager：负责管理当前 Broker 所有分区和副本的信息，处理 KafkaController 发起的一些请求，副本状态的切换、添加/读取消息等

## 延伸概念

### Partition

- 每一个 Topic 被切分为多个 Partition
- 消费者数目少于或等于 Partition 数目
- Broker Group 中的每一个 Broker 保存 Topic 的一个或多个 Partition
- Consumer Group 中的仅有一个 Consumer 读取 Topic 的一个或多个 Partition，并且是唯一的 Consumer

### Replication

- 当集群中有 Broker 挂掉的情况，系统可以主动的使 Replicas 提供服务
- 系统默认设置每一个