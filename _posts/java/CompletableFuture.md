---
layout: post
title:  "java.util.concurrent.CompletableFuture"
date:   2018-04-17
categories: java
---

### API

+ supplyAsync 向ForkJoinPool.commonPool（或者一个指定的Executor）提交一个异步执行的任务，并包含一个返回值，返回一个新的CompletableFuture
+ runAsync 向ForkJoinPool.commonPool（或者一个指定的Executor）提交一个异步执行的任务，无返回值，返回一个新的CompletableFuture

