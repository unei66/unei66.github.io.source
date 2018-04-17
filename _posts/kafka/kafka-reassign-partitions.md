---
layout: post
title:  "kafka-reassign-partitions"
date:   2018-04-17
categories: kafka
---

### topic添加partition



+ 需操作的topic，定义一个json文件

➜  bin vi topic.json

	{
		"topics":[{"topics":"test"}],
		"version":1
	}
+ 生成reassign计划

➜  bin ./kafka-reassign-partitions.sh --zookeeper localhost:2181 --topics-to-move-json-file ./topic.json --broker-list '0,1,2' --generate
Current partition replica assignment
{"version":1,"partitions":[{"topic":"test","partition":4,"replicas":[0,2,1]},{"topic":"test","partition":1,"replicas":[0,2,1]},{"topic":"test","partition":2,"replicas":[1,0,2]},{"topic":"test","partition":5,"replicas":[1,0,2]},{"topic":"test","partition":0,"replicas":[2,1,0]},{"topic":"test","partition":3,"replicas":[2,1,0]}]}

Proposed partition reassignment configuration
{"version":1,"partitions":[{"topic":"test","partition":4,"replicas":[2,1,0]},{"topic":"test","partition":1,"replicas":[2,0,1]},{"topic":"test","partition":5,"replicas":[0,2,1]},{"topic":"test","partition":2,"replicas":[0,1,2]},{"topic":"test","partition":0,"replicas":[1,2,0]},{"topic":"test","partition":3,"replicas":[1,0,2]}]}

+ 保存reassign计划json

➜  bin vi reassignment.json

+ 执行reassign计划

➜  bin ./kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file ./reassignment.json --execute
Current partition replica assignment

{"version":1,"partitions":[{"topic":"test","partition":4,"replicas":[0,2,1]},{"topic":"test","partition":1,"replicas":[0,2,1]},{"topic":"test","partition":2,"replicas":[1,0,2]},{"topic":"test","partition":5,"replicas":[1,0,2]},{"topic":"test","partition":0,"replicas":[2,1,0]},{"topic":"test","partition":3,"replicas":[2,1,0]}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions.

+ 校验reassign计划

➜  bin ./kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file ./reassignment.json --verify
Status of partition reassignment:
Reassignment of partition [test,4] completed successfully
Reassignment of partition [test,0] completed successfully
Reassignment of partition [test,3] completed successfully
Reassignment of partition [test,2] completed successfully
Reassignment of partition [test,1] completed successfully
Reassignment of partition [test,5] completed successfully

增加partition完成，但是老的partition中的数据不会迁移到新的partition