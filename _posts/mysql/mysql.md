---
layout: post
title:  "mysql 索引"
date:   2018-04-17
categories: mysql
---

## 索引
1. 一个文件可以有多个索引，分别基于不同的搜索码。如果包含记录的文件按照某个搜索码指定的顺序排序，那么该搜索码对应的索引成为聚集索引(clustering index)，聚集索引的搜索码常常是主码，不是必须如此。
2. 索引项(index entry)或索引记录(index record)由一个搜索码和指向具有该搜索码值的一条或者多条记录的指针组成。指向记录的指针包括磁盘块的标识和标识磁盘块内记录的块内偏移量。
+ 稠密索引（dense index):在稠密索引中，文件中的每个搜索码值都有一个索引项。在稠密聚集索引中，索引项包括索引码值以及指向具有该搜索码值的第一条数据记录的指针。具有相同搜索码值的其余记录顺序地存储在第一条数据记录之后，由于该索引是聚集索引，因此记录根据相同的索引码值排序。在稠密非聚集索引中，索引必须存储指向所有具有相同索引码值的记录的指针列表。


# set autocommit=0

set autocommit=0;
update tba set name=1 where id=1;
update tbb set name=2 where id=1;
commit;
set autocommit=1;

commit以上的sql相当于一个事务，在不commit时，其他会话时看不到结果，如果此时其他会话更新相同的数据，会堵塞等待。