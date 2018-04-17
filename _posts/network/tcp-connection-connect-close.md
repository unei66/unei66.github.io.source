---
layout: post
title:  "TCP连接的建立和终止"
date:   2015-10-31 13:00
categories: 网络
---

##TCP连接建立
(1).服务器必须准备好接受外来的连接。通常通过调用socket，bind和listen方法完成，称之为被动打开（passive open）。

(2).客户端通过调用connect发起主动打开（active open）。这导致客户TCP发送一个SYN(同步)分节，告诉服务器客户端将在(待建立的)
连接中发送的数据的初始序列号。通常SYN节不携带数据，其所在的IP数据报只含有一个IP首部，一个TCP首部及可能有的TCP选项。

(3).服务器必须确认（ACK）客户的SYN，同时自己也得发送SYN分节，它含有服务器将在同一连接中发送的数据的初始序列号。服务器在单个分节中发送SYN和对客户SYN的ACK。

(4).客户端必须确认服务器的SYN。

这种交换至少需要3个分组，因此被称为TCP的三路握手(three-way handshake).
连接过程如下图所示
<img style="width:400px;height:200px" src="images/unp/tcp-three-way-handshake.png"/>

##TCP连接终止
TCP建立一个连接需要3个分组，终止一个连接需要4个分组
(1).某个应用首先调用close，我们称该端执行主动关闭(active close).该端的TCP发送一个FIN分节，表示数据发送完毕。

(2).接收到FIN的对端执行被动关闭（passive close).这个FIN由TCP确认（ACK）。它的接受也作为一个文件结束符(end-of-file)传递给接收端应用进程(放在已排队等候该应用进程接受的任何其他数据之后)，因为FIN的接收意味着接收端应用进程在相应连接上再无额外数据可接收。

(3).一段时间后，接收到这个文件结束符的应用进程将调用close关闭它的套接字。这导致它的TCP也发送一个FIN。

(4).接收这个最终FIN的原发送端TCP（即执行主动关闭的那一端)确认这个FIN。

在步骤2,3之间，从执行被动关闭一端到执行主动关闭一端流动数据是可能的，这称为半关闭(half-close).

TCP终止的过程如下图所示

<img style="width:400px;height:200px" src="images/unp/tcp-close.png"/>

一个完整的TCP连接所发生的实际分组交换情况，包括建立连接，数据传送和连接终止3个阶段。如下图所示：

<img style="width:500px;height:500px" src="images/unp/tcp-conn.png"/>

##参考资料
1. 《UNIX网络编程 卷1：套接字联网API》