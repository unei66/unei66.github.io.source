---
layout: post
title:  "network"
date:   2018-04-17
categories: network
---

### 状态

<img src='images/tcp-connect-close-state-change.png' style="zoom:50%"/>

+ TIME_WAIT

  客户端收到服务端发送的最后一个FIN，会进入TIME_WAIT状态，TIME_WAIT状态会持续2MSL，原因是，客户端进入TIME_WAIT后会发送给服务端一个ACK（1MSL），如果服务端没收到这个ACK，需要重发FIN（1MSL)，所以共2MSL。TIME_WAIT期间，任何迟到的报文都会被丢弃。



### RST

+ 到不存在的端口的连接请求
+ 异常终止一个连接

### 呼入连接请求队列

+ 正等待连接请求的一端有一个固定长度的请求队列，该队列中的连接已被TCP接受（三次握手完成），但还没有被应用层接受。TCP接受一个连接是将其放入队列，应用层接受连接是将其从队列中取出。
+ 通过backlog配置，一般0-5




# Nagle

```
Nagle用于解决小包过多问题。发送条件：一个就是缓 冲区中的字节数达到了一定量，另一个就是等待了一定的时间（一般的Nagle算法都是等待200ms）；
这两个门槛的任何一个达到都必须发送数据了。一般情况下，如果数据流量很大，第二个条件是永远不会起作用的，但当发送小的数据包时，第二个门槛就发
挥作用了，防止数据被无限的缓存在缓冲区。
```

- 关闭Nagle:TCP_NODELAY

  ​

拥塞窗口 congestion window

通告窗口


发送窗口
接收窗口

MSS：网络传输数据最大值
MTU：网络传输最大报文包
	mss加包头数据就等于mtu.
	简单说拿TCP包做例子。
	报文传输1400字节的数据的话，那么mss就是1400，再加上20字节IP包头，20字节tcp包头，那么mtu就是1400+20+20.
	当然传输的时候其他的协议还要加些包头在前面，总之mtu就是总的最后发出去的报文大小。mss就是你需要发出去的数据大小。

win:接收窗口大小

01:47:43.351148 IP 192.168.35.100.52666 > 192.168.34.38.distinct: Flags [S], seq 3841046457, win 65535, options [mss 1460,nop,wscale 5,nop,nop,TS val 178469865 ecr 0,sackOK,eol], length 0
01:47:43.351388 IP 192.168.34.38.distinct > 192.168.35.100.52666: Flags [S.], seq 3853031824, ack 3841046458, win 28960, options [mss 1460,sackOK,TS val 2565700 ecr 178469865,nop,wscale 6], length 0
01:47:43.351429 IP 192.168.35.100.52666 > 192.168.34.38.distinct: Flags [.], ack 1, win 4117, options [nop,nop,TS val 178469865 ecr 2565700], length 0
01:47:43.351749 IP 192.168.34.38.distinct > 192.168.35.100.52666: Flags [P.], seq 1:6, ack 1, win 453, options [nop,nop,TS val 2565700 ecr 178469865], length 5
01:47:43.351797 IP 192.168.35.100.52666 > 192.168.34.38.distinct: Flags [.], ack 6, win 4117, options [nop,nop,TS val 178469865 ecr 2565700], length 0
01:47:43.352114 IP 192.168.34.38.ssh > 192.168.35.100.52199: Flags [P.], seq 2214141269:2214141345, ack 4170104506, win 582, options [nop,nop,TS val 2565700 ecr 178460565], length 76
01:47:43.352158 IP 192.168.35.100.52199 > 192.168.34.38.ssh: Flags [.], ack 76, win 4093, options [nop,nop,TS val 178469866 ecr 2565700], length 0

1. 从最简单的UDP协议开始，可以用C去写些简单的UDP发送/接受程序，然后记得会用抓包软件Wireshark去抓取程序运行时发送的数据包，逐层逐层的去看去学习。由于网络是分层设计，所以你对网络的了解最好也按照层次递进关系来学习，先不管路由，不管网络层，从传输层入手，一层一层的征服。    
2. 用raw socket实现步骤1中的程序，自己去构造UDP报文，根据协议手动的往里面塞数据，这样你自己“抚摸”过这些01之后对协议的感觉 *绝对* 不一样。    
3. 更深一步，用raw socket去构造整个网络层及以上的报文，进而去了解IP协议各字段协议。这个过程通过Wireshark去debug抓包分析。    
4. 经过上面三个步骤配合书本，我想你自己应该会对网络提起兴致，接下来就可以专注的看些理论了解路由算法，隧道技术等等。

tcpdump -ieth0 port 3306 -w /home/localadmin/tcpdump -G 600 -s0

tcpdump -ieth0 port 80 -w /root/tcpdump -G 600 -s0
