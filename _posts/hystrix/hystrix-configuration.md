---
layout: post
title:  "hystrix configuration"
date:   2018-04-17
categories: hystrix
---

# Command Properties

## Execution

+ execution.isolation.strategy  

  default:THREAD

  HystrixCommand.run 执行的隔离策略。

  THREAD：在线程池的线程中执行。

  SEMAPHORE：在调用线程中执行，最大并发数受信号量数量控制。

  HystrixCommand推荐THREAD，HystrixObservableCommand推荐SEMAPHORE

+ execution.isolation.thread.timeoutInMilliseconds

  default:1000

  调用超时时间。Hystrix会将该Command置为TIMEOUT，调用fallback逻辑。

+ execution.timeout.enabled

  default:true

  Command.run是否有超时

+ execution.isolation.thread.interruptOnTimeout

  default:true

  超时时，是否中断Command

+ execution.isolation.thread.interruptOnCancel

  default:false

  当Cancel时，是否中断Command

+ execution.isolation.semaphore.maxConcurrentRequests

  default:10

  当使用ExecutionIsolationStrategy.SEMAPHORE时，command最大请求数量



## Fallback

+ fallback.isolation.semaphore.maxConcurrentRequests

  default:10

  fallback并发数，超过返回Exception

+ fallback.enabled

  default:true

## Circuit Breaker

+ circuitBreaker.enabled

  default:true

+ circuitBreaker.requestVolumeThreshold

  default:20

  在一个窗口中，可以触发circuitBreaker的最小请求数，19个请求，即使全部失败也不会触发circuitBreaker

+ circuitBreaker.sleepWindowInMilliseconds

  default:5000

  circuitBreaker被触发后，circuitBreaker会在该时间段内拒绝所有请求，该时间过后，会尝试执行run，如果成功执行，会将circuitBreaker关闭，否则一直打开。

+ circuitBreaker.errorThresholdPercentage

  default:50

  在窗口内，失败请求的比例

+ circuitBreaker.forceOpen

  default:false

  打开circuitBreaker，拒绝所有请求

+ circuitBreaker.forceClosed

  default:false

  关闭circuitBreaker

## Metrics

+ metrics.rollingStats.timeInMilliseconds

  default:10000

  窗口大小，保留多长时间的统计信息来决定是否使用circuitBreaker。

+ metrics.rollingStats.numBuckets

  default:10

  窗口包含的bucket数量

+ metrics.rollingPercentile.enabled

  default:true

  延迟执行是否被跟踪并按百分比展示

  ​

  + metrics.rollingPercentile.timeInMilliseconds


  + ​

## Request Context



# Collapser Properties



# Thread Pool Properties

