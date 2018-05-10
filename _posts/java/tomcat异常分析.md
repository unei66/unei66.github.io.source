---
layout: post
title:  "tomcat异常分析"
date:   2018-04-26
categories: java
---

### 发现

半个月前发现，邮件报警，线上的tomcat一直在报监控接口超时，这个监控接口就是返回个hello world,没有依赖其他系统。这个报警，大约几个小时来一次。



### 线上监控

1. 监控接口请求耗时监控

   ![image003](/images/java-tomcat-gc/image003.png)

   

2. 网络监控

   ![image001](/images/java-tomcat-gc/image001.png)

   

3. cpu监控

   ![image002](/images/java-tomcat-gc/image002.png)

4. jstat

   ![gcutil](/images/java-tomcat-gc/gcutil.png)



### 推断

从上面三个可以看出，机器没死，活着呢，网络停止了，估计jvm在gc（最后证实确实如此）。



### 优化思路



看一下原始的jvm参数（这个参数不是我配的😂）：

```
-XX:+UseCompressedOops -Xms7G -Xmx7G -XX:MetaspaceSize=64M -XX:+MaxMetaspaceSize=384M -Xloggc:path/log.log -verbose:gc  -XX:HeapDumpPath=/tmp -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:+CMSClassUnloadingEnabled -XX:ParallelGCThreads=4 -XX:ConcGCThreads=2 -XX:+HeapDumpOnOutOfMemoryError -XX:SurvivorRatio=26 -XX:+CMSParallelRemarkEnabled -XX:CMSInitiatingOccupancyFraction=80
```

首先，我需要看一下gc log，结果运维找不到日志，尴尬，相对路径，日志就没输出。第二，我要看一下fgc前后的堆变化情况，然后就加了下面的参数：

-XX:+HeapDumpBeforeFullGC -XX:+HeadDumpAfterFullGC

这俩东西就一起搞到线上去了。

然后。。。等待问题复现。

抓到了，看gclog！

```
2018-04-25T09:58:55.273+0800: 39018.975: [GC (Allocation Failure) 39018.976: [ParNew (promotion failed): 328576K->328576K(328576K), 0.2897109 secs]39019.266: [CMS2018-04-25T09:59:37.021+0800: 39060.736: [CMS-concurrent-sweep: 61.437/80.407 secs] [Times: user=85.18 sys=9.30, real=80.42 secs]
 (concurrent mode failure): 5754419K->709193K(6999296K), 93.9618683 secs] 6066058K->709193K(7327872K), [Metaspace: 125046K->125046K(1167360K)], 94.2613781 secs] [Times: user=8.22 sys=3.47, real=94.26 secs]
```

看到这里，就发现问题不对劲了，concurrent mode failure，real 80.42, 94.26。

其实在这个阶段，我没仔细算gc前后的heap大小，又去看了一下heap dump，dump也没看出来啥，但是，fgc前后的文件大小差距就大了，一个6.5g，一个600m。这个提醒我去看一下gc log。

young大小：320M？

看一下gc log中的初始化参数吧，找到gc log的最开头位置：

```
Memory: 4k page, physical 8010824k(7578720k free), swap 2097148k(1996844k free)
CommandLine flags: -XX:+CMSClassUnloadingEnabled -XX:CMSInitiatingOccupancyFraction=80 -XX:+CMSParallelRemarkEnabled -XX:ConcGCThreads=2 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/ -XX:InitialHeapSize=7516192768 -XX:+ManagementServer -XX:MaxHeapSize=7516192768 -XX:MaxMetaspaceSize=402653184 -XX:MaxNewSize=348966912 -XX:MetaspaceSize=67108864 -XX:NewSize=348966912 -XX:OldPLABSize=16 -XX:OldSize=697933824 -XX:ParallelGCThreads=4 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:SurvivorRatio=26 -XX:+UseCMSCompactAtFullCollection -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:+UseParNewGC
2018-04-24T23:08:44.044+0800: 7.746: [GC (Allocation Failure) 7.746: [ParNew: 316416K->12160K(328576K), 0.0890565 secs] 316416K->24149K(7327872K), 0.0891831 secs] [Times: user=0.15 sys=0.01, real=0.09 secs]
2018-04-24T23:08:48.077+0800: 11.778: [GC (Allocation Failure) 11.778: [ParNew: 328576K->12159K(328576K), 0.0597998 secs] 340565K->48432K(7327872K), 0.0598772 secs] [Times: user=0.14 sys=0.01, real=0.06 secs]
2018-04-24T23:08:50.000+0800: 13.702: [GC (Allocation Failure) 13.702: [ParNew: 328575K->12160K(328576K), 0.0672809 secs] 364848K->69694K(7327872K), 0.0673553 secs] [Times: user=0.23 sys=0.01, real=0.07 secs]
2018-04-24T23:08:50.742+0800: 14.444: [GC (Allocation Failure) 14.444: [ParNew: 328576K->12160K(328576K), 0.0572250 secs] 386110K->101739K(7327872K), 0.0572988 secs] [Times: user=0.13 sys=0.00, real=0.06 secs]
```

1. 启动参数：NewSize，MaxNewSize 340788K **这么大的堆，这么小的新生代？**
2. gc 日志：

[ParNew: 328576K->12160K(328576K)  一个survivor区占用12160K，328576K=1个survivor+eden，survivor:eden=2:26 (12160*2):(328576-12160)=2:26  **-XX:SurvivorRatio=26**不理解这个参数为何要配这么多

3. -XX:CMSInitiatingOccupancyFraction=80 

   配置了启动的时机，但是没有配置-XX:+UseCMSInitiatingOccupancyOnly，导致这个配置成为摆设。

4. gc log中发现了System.gc

   那就把这个加上：-XX:+DisableExplicitGC，忽略系统的gc调用。



最终参数：

```
-Xms7G -Xmx7G -Xmn2G 
-XX:MetaspaceSize=64M 
-XX:MaxMetaspaceSize=384M 
-Xloggc:xxx 
-verbose:gc 
-XX:+PrintGCDateStamps 
-XX:+PrintGCDetails 
-XX:+UseConcMarkSweepGC 
-XX:+UseCMSCompactAtFullCollection 
-XX:CMSInitiatingOccupancyFraction=60
-XX:+UseCMSInitiatingOccupancyOnly 
-XX:+CMSClassUnloadingEnabled 
-XX:+CMSParallelRemarkEnabled
-XX:ParallelGCThreads=4 
-XX:ConcGCThreads=2 
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/tmp/ 
-XX:+DisableExplicitGC 
```

考虑加上的参数：

CMSFullGCsBeforeCompaction  ：在上一次CMS并发GC执行过后，到底还要再执行多少次full GC才会做压缩。默认是0，也就是在默认配置下每次CMS GC顶不住了而要转入full GC的时候都会做压缩。 把CMSFullGCsBeforeCompaction配置为10，就会让上面说的第一个条件变成每隔10次真正的full GC才做一次压缩（**而不是每10次CMS并发GC就做一次压缩，目前VM里没有这样的参数**）。这会使full GC更少做压缩，也就更容易使CMS的old gen受碎片化问题的困扰。 本来这个参数就是用来配置降低full GC压缩的频率，以期减少某些full GC的暂停时间。CMS回退到full GC时用的算法是**mark-sweep-compact**，但compaction是可选的，不做的话碎片化会严重些但这次full GC的暂停时间会短些；这是个取舍。



CMSScavengeBeforeRemark : CMS前进行一次MinorGC



调整后大约24h ，gc情况：

![gc-after-24h](/images/java-tomcat-gc/gc-after-24h.png)

平均一个小时两次fgc，每次的时间不到0.5s，大体上可以接受。



### 总结

1. cms流程


   2018-04-28T15:48:13.167+0800: 1341.458: [GC (CMS Initial Mark) [1 CMS-initial-mark: 4076417K(5242880K)] 4093238K(7130368K), 0.0064435 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]

   2018-04-28T15:48:13.173+0800: 1341.465: [CMS-concurrent-mark-start]
   2018-04-28T15:48:13.416+0800: 1341.708: [CMS-concurrent-mark: 0.240/0.243 secs] [Times: user=0.88 sys=0.07, real=0.24 secs]

   2018-04-28T15:48:13.416+0800: 1341.708: [CMS-concurrent-preclean-start]

   2018-04-28T15:48:13.436+0800: 1341.728: [CMS-concurrent-preclean: 0.019/0.020 secs] [Times: user=0.07 sys=0.00, real=0.02 secs]

   2018-04-28T15:48:13.436+0800: 1341.728: [CMS-concurrent-abortable-preclean-start]

   2018-04-28T15:48:15.145+0800: 1343.437: [GC (Allocation Failure) 1343.437: [ParNew: 1693259K->189383K(1887488K), 0.3191849 secs] 5769676K->4265801K(7130368K), 0.3194583 secs] [Times: user=0.40 sys=0.00, real=0.32 secs]

   2018-04-28T15:48:17.424+0800: 1345.716: [CMS-concurrent-abortable-preclean: 2.845/3.988 secs] [Times: user=10.56 sys=0.64, real=3.99 secs]

   2018-04-28T15:48:17.427+0800: 1345.719: [GC (CMS Final Remark) [YG occupancy: 1685779 K (1887488 K)]1345.719: [Rescan (parallel) , 0.3523662 secs]1346.071: [weak refs processing, 0.3003263 secs]1346.372: [class unloading, 0.1228579 secs]1346.495: [scrub symbol table, 0.0134209 secs]1346.508: [scrub string table, 0.0023638 secs][1 CMS-remark: 4076417K(5242880K)] 5762197K(7130368K), 0.7958359 secs] [Times: user=1.85 sys=0.00, real=0.80 secs]

   2018-04-28T15:48:18.238+0800: 1346.530: [CMS-concurrent-sweep-start]

   2018-04-28T15:48:18.419+0800: 1346.711: [GC (Allocation Failure) 1346.711: [ParNew: 1867207K->199251K(1887488K), 0.3801573 secs] 5812075K->4163268K(7130368K), 0.3804263 secs] [Times: user=0.52 sys=0.00, real=0.38 secs]

   2018-04-28T15:48:20.851+0800: 1349.143: [GC (Allocation Failure) 1349.143: [ParNew: 1877075K->159618K(1887488K), 0.2674537 secs] 3508407K->1803795K(7130368K), 0.2677172 secs] [Times: user=0.35 sys=0.00, real=0.27 secs]

   2018-04-28T15:48:22.040+0800: 1350.331: [CMS-concurrent-sweep: 3.127/3.801 secs] [Times: user=11.04 sys=0.50, real=3.80secs]

   2018-04-28T15:48:22.040+0800: 1350.331: [CMS-concurrent-reset-start]
   2018-04-28T15:48:22.051+0800: 1350.343: [CMS-concurrent-reset: 0.011/0.011 secs] [Times: user=0.04 sys=0.00, real=0.01secs]

   2018-04-28T15:48:22.683+0800: 1350.975: [GC (Allocation Failure) 1350.975: [ParNew: 1837442K->204471K(1887488K),0.4063115 secs] 2327686K->697104K(7130368K), 0.4065654 secs] [Times: user=0.47 sys=0.00, real=0.40 secs]

   

   + CMS Initial Mark 

     初始标记。STW。old size：4076417K，old capacity:5242880K，heap size(不包括perm):4093238K,heap size:7130368K

   + CMS-concurrent-preclean

   + CMS Final Remark

     最终标记。STW。

     [YG occupancy: 1685779 K (1887488 K)]1345.719: [Rescan (parallel) , 0.3523662 secs]1346.071: [weak refs processing, 0.3003263 secs]1346.372: [class unloading, 0.1228579 secs]1346.495: [scrub symbol table, 0.0134209 secs]1346.508: [scrub string table, 0.0023638 secs][1 CMS-remark: 4076417K(5242880K)] 5762197K(7130368K), 0.7958359 secs] [Times: user=1.85 sys=0.00, real=0.80 secs]

     young size:1685779K

     young capacity:1887488 K

     heap size:5762197K

     heap capacity:7130368K

   + 分析一下CMS-concurrent-sweep-start 以后的几次ParNew：

     

     | 次数 | young size before MinorGC | young size after MinorGC | heap size before MinorGC | heap size after MinorGC |
     | ---- | ------------------------- | ------------------------ | ------------------------ | ----------------------- |
     | 1    | 1867207K                  | 199251K                  | 5812075K                 | 4163268K                |
     | 2    | 1877075K                  | 159618K                  | 3508407K                 | 1803795K                |
     | 3    | 1837442K                  | 204471K                  | 2327686K                 | 697104K                 |

     

​     从表格中计算，cms前后，old size：从4163268K-199251K  大约将为697104K-204471K。

2. cms参数调整怎么算优秀？

   标准：以对象从新生代提升到老年代的速度对老年代中的对象进行收集。

   达不到这个标准则称之为失速(Lost the Race)。失速会导致Stop-The-World 压缩式垃圾收集(单线程的Serial Old FullGC)。避免失速的关键是要结合足够大的老年代和足够快地初始化CMS垃圾收集周期，让它以比提升速率更快的速度回收空间。

3. concurrent mode failure

   + 当CMS GC进行过程中，Minor GC向老年代提升的速度快于收集器收集的速度。
   + CMS 和MinorGC同时进行时，新生代Survivor空间放不下，需要提升到老年代。
   + CMS进行时，业务对象直接分配到老年代，老年代放不下。

   上述三种会导致该现象发生。

   

   发生压缩式垃圾收集时，cms gc log会出现并发模式失效(Concurrent Mode Failure)日志：

   ```
   2018-04-30T08:27:57.376+0800: 147363.218: [GC (Allocation Failure) 147363.223: [ParNew: 2831168K->2831168K(2831168K), 0.0018961 secs]147363.225: [CMS2018-04-30T08:28:01.910+0800: 147367.747: [CMS-concurrent-sweep: 45.882/77.866 secs] [Times: user=138.72 sys=18.07, real=77.88 secs](concurrent mode failure): 4045869K->745862K(4194304K), 38.8136017 secs] 6877037K->745862K(7025472K), [Metaspace: 125954K->125954K(1167360K)], 38.8679322 secs] [Times: user=4.18 sys=1.47, real=38.86 secs]
   ```

   MinorGC 前后：young size都是2831168K ，说明MinorGC收集不了，而cms启动时，old区的可用空间为5G*30%，放不下young空间的数据，导致STW，然后转化为单线程的FullGC。例子中FullGC耗时38.86s。

4. promotion failed

   MinorGC时，Sruvivor space放不下，只能放到老年代，而此时老年代也放不下，此时会导致STW 使用Serial Old收集器进行Full GC.



参考资料：

http://itindex.net/detail/47030-cms-gc-%E9%97%AE%E9%A2%98

http://www.oracle.com/technetwork/java/javase/tech/index.html

http://www.oracle.com/technetwork/java/javase/gc-tuning-6-140523.html

https://www.jianshu.com/p/ca1b0d4107c5