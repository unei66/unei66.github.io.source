---
layout: post
title:  "linux问题排查命令"
date:   2018-04-17
categories: linux
---



+ 查看进程打开的文件

  lsof -a -p pid

+ 统计进程信息

  prtstat pid  

+ 查看执行栈

  pstack pid

  cat /proc/pid/stack

