ParallelScavenge和ParNew都是并行GC，主要是并行收集young gen，目的和性能其实都差不多。最明显的区别有下面几点：  

1、PS以前是广度优先顺序来遍历对象图的，JDK6的时候改为默认用深度优先顺序遍历，并留有一个UseDepthFirstScavengeOrder参数来选择是用深度还是广度优先。在JDK6u18之后这个参数被去掉，PS变为只用深度优先遍历。ParNew则是一直都只用广度优先顺序来遍历 

 2、PS完整实现了adaptive size policy，而ParNew及“分代式GC框架”内的其它GC都没有实现**完**（倒不是不能实现，就是麻烦+没人力资源去做）。所以千万千万别在用ParNew+CMS的组合下用UseAdaptiveSizePolicy，请只在使用UseParallelGC或UseParallelOldGC的时候用它。  

3、由于在“分代式GC框架”内，ParNew可以跟CMS搭配使用，而ParallelScavenge不能。当时ParNew GC被从Exact VM移植到HotSpot VM的最大原因就是为了跟CMS搭配使用。  

4、在PS成为主要的throughput GC之后，它还实现了针对NUMA的优化；而ParNew一直没有得到NUMA优化的实现。 



from: http://hllvm.group.iteye.com/group/topic/37095#post-242695