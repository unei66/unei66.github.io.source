---
layout: post
title:  "gc"
date:   2018-04-17
categories: java
---



#### minor gc

minor gc都会stw。



#### gc log

-XX:+PrintGC

-XX:+PrintGCDetails

-XX:+PrintGCTimeStamps

-Xloggc:/tmp/gc-test.log

-XX:+HeapDumpOnOutOfMemoryError

-XX:+UseGCLogfileRotation -XX:NumberOfGCLogfiles=N -XX:GCLogfileSize=N  日志文件循环

## garbage collector

### serial

-XX:+UseSerialGC

### throughput

-XX:+UseParallelGC

-XX:+UseParallelOldGC

### cms

cms默认使用Parnew作为young收集器，cms停顿时间更短，应用线程只在minorGC以及后台线程扫描老年代时发生短暂的停顿。

#### 缺点

+ 更高的CPU使用。
+ 堆容易碎片化。
+ 如果cms后台线程没有足够的cpu资源，或者碎片化过于严重或者处理速度过慢导致无法找到连续空间分配对象（ Concurrent Mode Failure），cms会蜕变为serial收集器的行为：暂停所有线程，使用单线程回收，整理老年代空间。

#### 过程

1. 初始标记 暂停所有应用线程

   89.976: [GC [1 CMS-initial-mark: 702254K(1398144K)]
   772530K(2027264K), 0.0830120 secs]
   [Times: user=0.08 sys=0.00, real=0.08 secs]

   702254K 老年代中使用大小

   1398144K 老年代总大小

   772530K 被占用的堆大小

   2027264K 堆大小

   0.08 secs 暂停时间

2. 并发标记阶段 应用线程不暂停

   90.059: [CMS-concurrent-mark-start]
   90.887: [CMS-concurrent-mark: 0.823/0.828 secs]
   [Times: user=1.11 sys=0.00, real=0.83 secs]

3. 预清理阶段  应用线程不暂停

   90.887: [CMS-concurrent-preclean-start]
   90.892: [CMS-concurrent-preclean: 0.005/0.005 secs]
   [Times: user=0.01 sys=0.00, real=0.01 secs]

4.  重新标记阶段  可中断预清理

   90.892: [CMS-concurrent-abortable-preclean-start]
   92.392: [GC 92.393: [ParNew: 629120K->69888K(629120K), 0.1289040 secs]
   1331374K->803967K(2027264K), 0.1290200 secs]
   [Times: user=0.44 sys=0.01, real=0.12 secs]
   94.473: [CMS-concurrent-abortable-preclean: 3.451/3.581 secs]
   [Times: user=5.03 sys=0.03, real=3.58 secs]
   94.474: [GC[YG occupancy: 466937 K (629120 K)]
   94.474: [Rescan (parallel) , 0.1850000 secs]
   94.659: [weak refs processing, 0.0000370 secs]
   94.659: [scrub string table, 0.0011530 secs]
   [1 CMS-remark: 734079K(1398144K)]
   1201017K(2027264K), 0.1863430 secs]
   [Times: user=0.60 sys=0.01, real=0.18 secs]

5. 清除阶段 并发

   94.661: [CMS-concurrent-sweep-start]
   95.223: [GC 95.223: [ParNew: 629120K->69888K(629120K), 0.1322530 secs]
   999428K->472094K(2027264K), 0.1323690 secs]
   [Times: user=0.43 sys=0.00, real=0.13 secs]
   95.474: [CMS-concurrent-sweep: 0.680/0.813 secs]
   [Times: user=1.45 sys=0.00, real=0.82 secs]

   4.5中的Parnew说明新生代和老年代的回收可以并发进行。

#### cms可能碰到的问题

1. Concurrent mode failure

   增加老年代

   2) 

2. 老年代有足够的空间容纳晋升对象，但由于碎片化严重，导致晋升失败。

   cms在新生代收集时，对象晋升失败。然后对整个老年代进行整理压缩。

   6043.903: [GC 6043.903:
   [ParNew (promotion failed): 614254K->629120K(629120K), 0.1619839 secs]
   6044.217: [CMS: 1342523K->1336533K(2027264K), 30.7884210 secs]
   2004251K->1336533K(1398144K),
   [CMS Perm : 57231K->57231K(95548K)], 28.1361340 secs]
   [Times: user=28.13 sys=0.38, real=28.13 secs]

3. ​

#### 选项

- -XX:+UseParNewGC
- -XX:+UseConcMarkSweepGC
- -XX:CMSInitiatingOccupancyFraction=N,-XX:+CMSUseInitiatingOccupancyOnly
- ​

##### g1

g1将堆划分为若干region（默认2048），仍属于分带收集器。首先收集垃圾最多的分区。新生代的垃圾收集仍然采用暂停所有应用线程
的方式，将存活对象移动到老年代或者 Survivor 空间。同其他的收集算法一样，这些操作
也利用多线程的方式完成。

缺点：

+ 更多的cpu消耗。负责gc的多个thread，必须获得足够的cpu运行周期。



过程：

1. 初始标记阶段 initial-mark 暂停应用线程

   50.541: [GC pause (young) (initial-mark), 0.27767100 secs]
   [Eden: 1220M(1220M)->0B(1220M)
   Survivors: 144M->144M Heap: 3242M(4096M)->2093M(4096M)]
   [Times: user=1.02 sys=0.04, real=0.28 secs]

2. 扫描根分区 root region

   50.819: [GC concurrent-root-region-scan-start]
   51.408: [GC concurrent-root-region-scan-end, 0.5890230]

   这个阶段中不能发生新生代垃圾收集，因此预留足够的 CPU 周期给
   后台线程运行是非常重要的。如果扫描根分区时，新生代空间刚巧用尽，新生代垃圾收集
   （会暂停所有的应用线程）必须等待根扫描结束才能完成。效果上，这意味着新生代垃圾
   收集的停顿时间会更长（远超过正常的耗时）

   350.994: [GC pause (young)
   351.093: [GC concurrent-root-region-scan-end, 0.6100090]
   351.093: [GC concurrent-mark-start],
   0.37559600 secs]

3. 并发标记  并发标记阶段是可以中断的，所以这个阶段中可能发生新生代垃圾收集。紧接在标记阶段之后的是重新标记（ remarking）阶段和正常的清理阶段  暂停应用线程

   111.382: [GC concurrent-mark-start]
   ....
   120.905: [GC concurrent-mark-end, 9.5225160 sec]

   ​

   120.910: [GC remark 120.959:
   [GC ref-PRC, 0.0000890 secs], 0.0718990 secs]
   [Times: user=0.23 sys=0.01, real=0.08 secs]
   120.985: [GC cleanup 3510M->3434M(4096M), 0.0111040 secs]
   [Times: user=0.04 sys=0.00, real=0.01 secs]

4. 并发清理

   120.996: [GC concurrent-cleanup-start]
   120.996: [GC concurrent-cleanup-end, 0.0004520]

   至此，G1的垃圾定位结束，定位出X region。

5. mixed gc

   79.826: [GC pause (mixed), 0.26161600 secs]
   ....
   [Eden: 1222M(1222M)->0B(1220M)
   Survivors: 142M->144M Heap: 3200M(4096M)->1964M(4096M)]
   [Times: user=1.01 sys=0.00, real=0.26 secs]

-XX:+UseG1GC



#####控制并发

-XX:ParallelGCThreads=N gc线程数，影响以下参数：

+ 使用 -XX:+UseParallelGC 收集新生代空间
+ 使用 -XX:+UseParallelOldGC 收集老年代空间


+  使用 -XX:+UseParNewGC 收集新生代空间
+  使用 -XX:+UseG1GC 收集新生代空间
+  CMS 收集器的stw阶段（但并非 Full GC）
+ G1 收集器的stw阶段（但并非 Full GC）



-XX:+UseComparessedOops







-XX:+PrintTLAB	输出TLAB信息
-XX:+UseTLAB	启用TLAB
-XX:TLABSize=N	TLAB默认大小
-XX:+ResizeTLAB	是否启动动态修改TLAB，每次gc动态修改

-XX:G1HeapRegionSize=N 标志设置（正常情况下，默认值是 0，意味着使用刚才描述的动态算法计算，分区大小 = 1 << log(初始堆的大小 /2048);,分区的大小最小是 1 MB，最大不能超过 32 MB