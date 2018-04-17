---
layout: post
title:  "Golang new make"
date:   2016-10-15 11:00
categories: GoLang
---

## new
+ new分配内存，用零初始化内存，返回新建类型的<b>地址</b>，*T
+ eg:new(File)和&File{}等同

## make
+ make只能用来新建slice，map，channel，它返回一个初始化完成的<b>值类型</b>T，不是用0初始化，返回的不是地址。

## 参考资料
+ [effective go](https://golang.org/doc/effective_go.html)
