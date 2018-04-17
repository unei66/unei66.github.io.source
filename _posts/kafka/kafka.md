---
layout: post
title:  "kafka-command"
date:   2018-04-17
categories: kafka
---

## 常用命令
####启动

bin/kafka-server-start.sh config/server.propertis &

####创建topic

bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic



#### controller

当有节点加入或离开集群时，controller负责在partition和replicas中选举leader。controller使用epoch number来防止脑裂问题（两个节点都认为自己是controller）。

集群中第一个在zk中建立/controller  ephemeral节点的broker会成为controller。其他的broker会watch /controller。



replication

leader replica，follower replica



​				       Fetch Request

Follower Replica +--------------------------------> Leader Replica

Fetch Request包含要接收的下一个消息的offset，一直是有序的。

如果follower 十秒钟(replica.lag.time.max.ms控制)没有请求消息，或者在十秒钟内数据没有跟上leader，这个节点被认为out of sync。如果一个replica无法跟上leader，在leader异常时，该replica不能成为新的leader。



send acknowledge

acks=0 means that a message is considered to be written successfully to Kafka if
the producer managed to send it over the network

acks=1 means that the leader will send either an acknowledgment or an error the
moment it got the message and wrote it to the partition data file (but not neces‐
sarily synced to disk)

acks=all means that the leader will wait until all in-sync replicas got the message
before sending back an acknowledgment or an error



ISR：In-Sync Replicas



Exactly-once delivery

+ writing results to a system that has some support for unique keys
+ The idea is to write the records and their offsets in the same transaction so they will be in-sync.When starting up, retrieve the offsets of the latest records written to the external store and then use consumer.seek() to start consuming again from those offsets. 