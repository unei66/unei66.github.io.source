---
layout: post
title:  "perf"
date:   2018-04-17
categories: linux
---



1. sudo perf record -a -e cycles -o cycle.perf -g sleep 10
2. sudo perf report -i cycle.perf |more

