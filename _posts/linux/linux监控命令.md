---
layout: post
title:  "linux 监控命令"
date:   2018-04-17
categories: linux
---

## linux

+ 每5秒查看磁盘io情况

  iostat -xm 5

+ 查看系统内存，io速度，系统中断，上下文切换等信息

  vmstat 1

+ 网络配置文件

  /etc/sysconfig/network-scripts/

+ sar

  -A：所有报告的总和
  -u：输出CPU使用情况的统计信息
  -v：输出inode、文件和其他内核表的统计信息
  -d：输出每一个块设备的活动信息
  -r：输出内存和交换空间的统计信息
  -b：显示I/O和传送速率的统计信息
  -a：文件读写情况
  -c：输出进程统计信息，每秒创建的进程数
  -R：输出内存页面的统计信息
  -y：终端设备活动情况
  -w：输出系统交换活动信息


​     sar -P ALL 打印所有cpu监控信息



### TCP

- 定位服务器丢包问题

  1. netstat -i 查看RX-DRP,TX-DRP

  2. dropwatch 可以查看哪个内核方法丢的数据包

     ```shell
     dropwatch -l kas
     start
     ```

  3. perf 监视kfree_skb 事件

     ```shell
     sudo perf record -g -a -e skb:kfree_skb
     sudo perf script
     ```

  4. tcpdump 两端抓包

  ​

- tcpdump 抓包

  1. tcpdump -pni eth0 host 172.16.0.3 and port 3306-s65535 -G 3600 -w 'trace_%Y-%m-%d_%H:%M:%S.pcap'

- dns 配置文件

  /etc/resolv.conf

- 查询dns

  nslookup,dig

- MSS 默认536字节，以太网MSS可达1460

- IP数据报头部40字节：20字节TCP首部，20字节IP首部

- wireshark

  1. ip.src==192.168.0.2 and ip.dst==192.168.0.233 and tcp.port==965  
  2. tcp contains "http"
  3. http.request.uri contains "online"

- lsof -i

- netstat -anlp