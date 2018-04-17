---
layout: post
title:  "wireshark https"
date:   2018-04-17
categories: network
---

wireshark 抓https包，chrome浏览器

1. chrome启动：./Google\ Chrome --ssl-key-log-file=～/ssl/ssl.log
2. wireshark配置：Preferences—Protocols—ssl—(Pre)-Master-Secret log filename 和步骤一的地址一致
3. wireshark start capturing...
4. wireshark分析pcap使用上述log文件无效。

