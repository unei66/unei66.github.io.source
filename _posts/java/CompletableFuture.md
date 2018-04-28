---
layout: post
title:  "java.util.concurrent.CompletableFuture"
date:   2018-04-17
categories: java
---



### API

+ supplyAsync 向ForkJoinPool.commonPool（或者一个指定的Executor）提交一个异步执行的任务，并包含一个返回值，返回一个新的CompletableFuture
+ runAsync 向ForkJoinPool.commonPool（或者一个指定的Executor）提交一个异步执行的任务，无返回值，返回一个新的CompletableFuture




CompletableFuture包含几个关键属性

result ：Future执行的结果或错误（AltResult）

Completion stack : treiber stack ，多个操作，例如supplyAsync,thenApplyAsync，会组合为栈



实例：

1. stageA=CompletableFuture.supplyAsync(supplierA)
   supplyAsync返回后：
   
   stack：null
   
   result：如果supplierA执行完，则result为其返回值，否则为null

2. stageB=stageA.thenApplySync(functionB)

   stageA.stack UniApply

   ​	src=stageA

   ​	dep=stageB

3. stageC=stageB.thenApplySync(functionC)

   stageB.stack UniApply

   ​	src=stageB

   ​	dep=stageC

supplierA,functionB,functionC 均为耗时操作，如果supplierA执行很快，在第二部完成时，stagetA.stack可能为null，不需要初始化stageA的stack


1. supplyAsync调用asyncSupplyStage完成实现，实际上就是生成一个AsyncSupply对象，提交到线程池。当只调用supplyAsync而且没有后续操作时，该方法返回CompleteFuture，和普通的Future没有太大区别。



```java
	static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                     Supplier<U> f) {
        if (f == null) throw new NullPointerException();
        CompletableFuture<U> d = new CompletableFuture<U>();
        e.execute(new AsyncSupply<U>(d, f));
        return d;
    }

	static final class AsyncSupply<T> extends ForkJoinTask<Void>
            implements Runnable, AsynchronousCompletionTask {
        CompletableFuture<T> dep; Supplier<T> fn;
        AsyncSupply(CompletableFuture<T> dep, Supplier<T> fn) {
            this.dep = dep; this.fn = fn;
        }
		
		... 
		
        public void run() {
            CompletableFuture<T> d; Supplier<T> f;
            if ((d = dep) != null && (f = fn) != null) {
                dep = null; fn = null;
                if (d.result == null) {
                    try {
                    	//f.get supplyAsync初始化的Supplier
                    	//completeValue 会将执行结果赋值给d的result
                        d.completeValue(f.get());
                    } catch (Throwable ex) {
                    	//如果失败，completeThrowable会将结果复制给result，AltResult类型
                        d.completeThrowable(ex);
                    }
                }
                //在实例中，当supplierA完成后，会调用dep(即stageB)的postComplete,postComplete-->tryFile-->uniApply(或者biApply等方法)-->dep.fn.apply(传入stageA的返回值),同样在stageB执行完，会触发stageC.postComplete
                d.postComplete();
            }

		}
    }
    
    /**
     * Pops and tries to trigger all reachable dependents.  Call only
     * when known to be done.
     */
    final void postComplete() {
        /*
         * On each step, variable f holds current dependents to pop
         * and run.  It is extended along only one path at a time,
         * pushing others to avoid unbounded recursion.
         */
        CompletableFuture<?> f = this; Completion h;
        while ((h = f.stack) != null ||
               (f != this && (h = (f = this).stack) != null)) {
            CompletableFuture<?> d; Completion t;
            if (f.casStack(h, t = h.next)) {
                if (t != null) {
                    if (f != this) {
                        pushStack(h);
                        continue;
                    }
                    h.next = null;    // detach
                }
                //tryFire
                f = (d = h.tryFire(NESTED)) == null ? this : d;
            }
        }
    }
```



2. thenApplyAsync  实现，如果实例中A已经执行完成，即当前Completable.result有返回值，则尝试执行B。如果A没有执行完成，则返回一个CompletableFuture.

   ```java
    private <V> CompletableFuture<V> uniApplyStage(
           Executor e, Function<? super T,? extends V> f) {
           if (f == null) throw new NullPointerException();
           CompletableFuture<V> d =  new CompletableFuture<V>();
           // 判断是否要加入到stack中
           if (e != null || !d.uniApply(this, f, null)) {
               UniApply<T,V> c = new UniApply<T,V>(e, d, this, f);
               //this.stack=c
               //c.dep=d
               //c.src=this
               push(c);
               //尝试调用f
               c.tryFire(SYNC);
           }
           return d;
       }

       final void push(UniCompletion<?,?> c) {
           if (c != null) {
               //result==null this.result不为null，即当前CompletableFuture未执行完
               while (result == null && !tryPushStack(c))
                   lazySetNext(c, null); // clear on failure
           }
       }

   	final boolean tryPushStack(Completion c) {
           //当前CompletableFuture的stack，如果	
           //CompletableFuturesupplyAsync().thenApplyAsync,stack=null
           Completion h = stack;
           //将c的next设置为h
           lazySetNext(c, h);
           //将this.stack 设置为c
           return UNSAFE.compareAndSwapObject(this, STACK, h, c);
       }
   ```

   ​

3. ​AsyncSupply.run—>stageA.postComplete—>stageB.tryFire—>stageB.uniApply—>stageB.claim(提交到Executor异步处理)