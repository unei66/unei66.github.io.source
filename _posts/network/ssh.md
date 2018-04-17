---
layout: post
title:  "ssh"
date:   2018-04-17
categories: network
---

+ ssh 端口转发

ssh -f root@<192.168.1.2> -L 2001:<192.168.1.1>:3306 -N

