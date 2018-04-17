---
layout: post
title:  "用过tcpdump观察TCP状态"
date:   2016-11-07 23:33
categories: 网络
---

# 准备工作：
+ 服务器：192.168.1.104 监听端口9999
+ 客户端：192.168.1.103 

# TCP状态解析
## 连接过程
+ 客户端向服务器端发送SYN请求，seq为X
+ 服务器向客户端返回SYN+ACK,seq为Y，ack为X+1
+ 客户端向服务器返回ACK，seq为Y+1,默认显示为偏移量1
因为一个SYN包会占用一个序列号，所以确认号加1

## 断开过程
实验中服务器端先断开

+ 服务器端向客户端发送FIN请求，seq为A，ack为B
+ 客户端向服务器发送ACK，seq为A+1
+ 客户端向服务器发送FIN，seq为B，ack为A+1
+ 服务器向客户端发送ACK，seq为B+1

tcpdump观察到的packet如下：

```
~$ tcpdump -ien0 -S  host 192.168.1.104 and port 9999
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on en0, link-type EN10MB (Ethernet), capture size 262144 bytes
--连接建立
1. 23:59:20.542060 IP 192.168.1.103.59850 > 192.168.1.104.distinct: Flags [S], seq 107490516, win 65535, options [mss 1460,nop,wscale 5,nop,nop,TS val 1508416415 ecr 0,sackOK,eol], length 0
2. 23:59:20.542277 IP 192.168.1.104.distinct > 192.168.1.103.59850: Flags [S.], seq 2963356419, ack 107490517, win 28960, options [mss 1460,sackOK,TS val 1833338 ecr 1508416415,nop,wscale 7], length 0
3. 23:59:20.542315 IP 192.168.1.103.59850 > 192.168.1.104.distinct: Flags [.], ack 2963356420, win 4117, options [nop,nop,TS val 1508416415 ecr 1833338], length 0
--服务器向客户端发送数据
4. 23:59:20.542616 IP 192.168.1.104.distinct > 192.168.1.103.59850: Flags [P.], seq 2963356420:2963356425, ack 107490517, win 227, options [nop,nop,TS val 1833338 ecr 1508416415], length 5
--客户端确认收到数据，ack
5. 23:59:20.542661 IP 192.168.1.103.59850 > 192.168.1.104.distinct: Flags [.], ack 2963356425, win 4117, options [nop,nop,TS val 1508416415 ecr 1833338], length 0
--客户端向服务器发送数据
6. 23:59:21.677798 IP 192.168.1.103.59850 > 192.168.1.104.distinct: Flags [P.], seq 107490517:107490519, ack 2963356425, win 4117, options [nop,nop,TS val 1508417548 ecr 1833338], length 2
--服务器向客户端发送ack
7. 23:59:21.677990 IP 192.168.1.104.distinct > 192.168.1.103.59850: Flags [.], ack 107490519, win 227, options [nop,nop,TS val 1834473 ecr 1508417548], length 0
--客户端向服务器发送数据
8. 23:59:25.171574 IP 192.168.1.103.59850 > 192.168.1.104.distinct: Flags [P.], seq 107490519:107490530, ack 2963356425, win 4117, options [nop,nop,TS val 1508421036 ecr 1834473], length 11
--服务器向客户端发送ack
9. 23:59:25.171797 IP 192.168.1.104.distinct > 192.168.1.103.59850: Flags [.], ack 107490530, win 227, options [nop,nop,TS val 1837967 ecr 1508421036], length 0
--连接断开阶段，服务器(104)发起
--seq为104上一个收到的数据包的ack值，ack为104上一个收到的数据包的序列号＋该包携带的数据大小
10. 23:59:27.469414 IP 192.168.1.104.distinct > 192.168.1.103.59850: Flags [F.], seq 2963356425, ack 107490530, win 227, options [nop,nop,TS val 1840264 ecr 1508421036], length 0
11. 23:59:27.469473 IP 192.168.1.103.59850 > 192.168.1.104.distinct: Flags [.], ack 2963356426, win 4117, options [nop,nop,TS val 1508423330 ecr 1840264], length 0
12. 23:59:27.469603 IP 192.168.1.103.59850 > 192.168.1.104.distinct: Flags [F.], seq 107490530, ack 2963356426, win 4117, options [nop,nop,TS val 1508423330 ecr 1840264], length 0
13. 23:59:27.470063 IP 192.168.1.104.distinct > 192.168.1.103.59850: Flags [.], ack 107490531, win 227, options [nop,nop,TS val 1840265 ecr 1508423330], length 0
```

## 关于序列号
一个SYN/FIN/PSH包的seq为该机器收到的上一个SYN/FIN/PSH包的ack，ack为上一个收到的包的序列号＋该包携带的数据大小。
SYN/FIN包占用一个序列化，所以在确认的ack中需要加1。