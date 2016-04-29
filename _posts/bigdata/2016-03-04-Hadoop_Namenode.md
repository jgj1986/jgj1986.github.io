---
title:  Hadoop-Namenode
date: 2016-03-04 17:12:28 +0800
category: tools
tags: tools
- a
- b
---

* toc
{:toc}

## hdfs HA 配置

现在的hadoop集群支持高可用性(High Availability)的配置，主要表现在在支持两个namenode (和Secondary NameNode有什么不同呢？)
参考 [hadoop HA配置文件说明](http://www.itdadao.com/article/234754/){:target="_blank"}，可知在hdfs-site.xml 中，添加两个上dfs.ha.namenodes.mycluster，支持rpc和http的监听端口，如下：

    <property>
      <name>dfs.ha.namenodes.mycluster</name>
      <value>nn1,nn2</value>
    </property>

    <property>
      <name>dfs.namenode.rpc-address.mycluster.nn1</name>
      <value>machine1:8020</value>
    </property>
    <property>
      <name>dfs.namenode.rpc-address.mycluster.nn2</name>
      <value>machine2:8020</value>
    </property>

    <property>
      <name>dfs.namenode.http-address.mycluster.nn1</name>
      <value>machine1:50070</value>
    </property>
    <property>
      <name>dfs.namenode.http-address.mycluster.nn2</name>
      <value>machine2:50070</value>
    </property>

到时候在machine1 和machine2 上都启动namenode服务即可。在最开始启动服务的时候，多采用如下的执行过程

    machine1> hdfs  namenode  –format
    machine1> hadoop-daemon.sh  start namenode
    machine2> hdfs namenode  -bootstrapStandby
    machine2> hadoop-daemon.sh  start  namenode

正常启动的话，machine1:50070 和 machine2:50070 都可在浏览器下访问。

## 不同

那 HA 使用的namenode(备用的Standby NameNode) 和 Secondary NameNode 有什么不同呢， 看看
[这篇文章](https://www.quora.com/What-is-the-difference-between-a-standby-NameNodes-and-a-secondary-NameNode-Does-the-new-Hadoop-with-YARN-have-a-secondary-NameNode){:target="_blank"}。

Secondary Namenode是将日志和镜像文件合到一起(然后namenode加载到ram中)，不具有容错功能；如果Namenode一旦出错，可能就要人工干预了;   
而HA环境的Standby Namenode则提供了自动的容错功能。HA不是强制性的，但使用了HA，就不要在使用SecondayNamenode了。

原文：

Secondary Namenode: 
In Hadoop 1.x and 2.x, the secondary namenode means the same. It does CPU intensive tasks for Namenode. In more details, it combines the Edit log and fs_image and returns the consolidated file to Namenode. Namenode then loads that file into RAM. But, secondary namenode doesn't provide failover capabilities.  So, in case of Namenode failure, Hadoop admins have to manually recover the data from Secondary Namenode.

Now, what's Standby Namenode?
In Hadoop 2.0, with the introduction of HA, the Standby Namenode came into picture. The standby namenode is the node that removes the problem of SPOF (Single Point Of Failure) that was there in Hadoop 1.x. The standby namenode provides automatic failover in case Active Namenode (can be simply called 'Namenode' if HA is not enabled) fails. 

Moreover, enabling HA is not mandatory. But, when it is enabled, you can't use Secondary Namenode. So, either Secondary Namenode is enabled OR Standby Namenode is enabled. 

