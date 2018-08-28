---
layout: post
title:  "java.util.concurrent.CountDownLatch"
date:   2018-04-17
categories: java
---



## 主要方法

1. await()  线程等待，直到CountDownLatch计数变为0
2. countDown() CountDownLatch计数减1



## 典型场景

1. 一个thread持有两个latch，一个用于作为开始信号，一个作为结束信号
2. 将一个任务切分为N份，latch最终等待所有任务完成

详细demo见java doc



## 问题

1. CountDownLatch为一次性的，不能reset。如果需要重复使用，考虑CyclicBarrier



## 分析

1. new

   完成Sync的创建。Sync为AbstractQueuedSynchronizer (AQS)的实现。

2. await

   await—>AQS.acquireSharedInterruptibly(1)

   		—>Sync.tryAcquireShared(1)

   		—>AQS.doAcquireSharedInterruptibly(1)



   ```
   private void doAcquireSharedInterruptibly(int arg)
           throws InterruptedException {
           //将当前node加入到CLH队列中，第一次加入时，会初始化head，tail
           final Node node = addWaiter(Node.SHARED);
           boolean failed = true;
           try {
               for (;;) {
               	//node的prev节点
                   final Node p = node.predecessor();
                   if (p == head) {
                   	//检查state是否为0，如果>=0，将node设置为head，
                   	//将node.waitStatus设置为PROPAGATE
                   	//然后返回
                       int r = tryAcquireShared(arg);
                       if (r >= 0) {
                           setHeadAndPropagate(node, r);
                           p.next = null; // help GC
                           failed = false;
                           return;
                       }
                   }
                   if (shouldParkAfterFailedAcquire(p, node) &&
                       parkAndCheckInterrupt())
                       throw new InterruptedException();
               }
           } finally {
               if (failed)
                   cancelAcquire(node);
           }
       }
   ```

   addWaiter

   ```java
   private Node addWaiter(Node mode) {
           Node node = new Node(Thread.currentThread(), mode);
           // Try the fast path of enq; backup to full enq on failure
           Node pred = tail;
           if (pred != null) {
               node.prev = pred;
               if (compareAndSetTail(pred, node)) {
                   pred.next = node;
                   return node;
               }
           }
           enq(node);
           return node;
       }
   ```



   ​

3. countDown


AQS.releaseShared(1)—>Sync.tryReleasedShared(1) 计数减1

				—>AQS.doReleasedShare()

