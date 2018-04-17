---
layout: post
title:  "Centos7 更新yum source为163"
date:   2016-10-27 00:48
categories: linux
---

```
#/bin/bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bk
cd /etc/yum.repos.d/
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
yum clean all
yum makecache
```