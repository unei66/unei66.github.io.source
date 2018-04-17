---
layout: post
title:  "zookeeper注意点"
date:   2018-04-17
categories: zookeeper
---

+ 节点不要太多

  全内存的节点，很容易fgc，stw，client端就不断reconnect

+ watch不要太多

  太多会导致包太大

+ server上下线要优先修改配置

  ​

### Rolling Restart in Apache ZooKeeper to dynamically add servers to the Ensemble

As of the writing of this post there is currently no way in Apache ZooKeeper to dynamically add a master ZooKeeper server to the ZooKeeper Ensemble.  That is until <https://issues.apache.org/jira/browse/ZOOKEEPER-107> is resolved.

As a work around there is a process known as the ZooKeeper ensemble rolling restart to get around this issue.

When you specify your zookeeper ensemble you setup a config file that holds the other zookeeper servers in the ensemble that it should attach to.  By modifying this file, according to the following steps, will allow you to add a zookeeper server to the ensemble without needing to take the service as a whole offline.  This could be due to a server replacement if a server goes down (and has to account for a DNS or IP address change) or if you want to expand the zookeeper cluster.

1. Modify your /config/zoo.cfg file to hold the new list of servers with your new addition.  If you wanted to grow from 7 servers to 9 servers then you would be adding in information about server 8 and 9.
   1. server.8=host:port:electionport
   2. server.9=host:port:electionport
   3. Ensure that server 8 and 9 are turned on and ready to go.  Start the ZooKeeper process on those servers.  They will follow the currently elected leader.
   4. Go through zookeeper servers 1-7 one by one and update the /config/zoo.cfg file so that they have information on zookeeper servers 8 and 9.  Restart the ZooKeeper service on that server.
   5. As you restart each server in step 3 above – ensure to watch your traffic and connection graphs on the other servers.  This is to ensure that the dropped connections from the other master get properly reassigned to other servers in the cluster.

While this is not an officially supported method for adding servers to the cluster – this is a method that will work,.  In fact it has been performed in a production environment with over 10,000 client connections attached to the ZooKeeper ensemble.  This has worked for 

1) needing to update DNS entries or IP address,

2) replacing failed hardware and 

3) expanding a cluster to be a larger size.

