---
title: mongodb install
date: 2014-10-28 20:12:28 +0800
tags: bigdta
- a
- b
---

* toc
{:toc}

## mongodb 集群 安装

### 说明
**使用机器**   
192.168.1.159  
192.168.1.171  
192.168.1.189  
**准备工作**  
1、下载文件 

    $ wget http://downloads.mongodb.org/linux/mongodb-linux-x86_64-2.6.5.tgz  
    $ tar xzf mongodb-linux-x86_64-2.6.5.tgz 

2、创建相应的目录  
3、保证磁盘空间在4G以上   

    $ mkdir ~/mongodb/data  
    $ ./bin/mongod --dbpath /home/work/mongodb/data/ --replSet repset  
    3379MB available in /home/work/mongodb/data/journal or use --smallfiles  

考虑到操作的简单，演示中操作尽可能的时候配置文件实现      



### replica set  

192.168.159:27017  -  master  
192.168.171:27017  -  slaver  
192.168.189:27017  -  arbiter  
   
**master (192.168.1.159)**  

    #master.conf    
    dbpath=/home/work/mongodb/data/master  
    logpath=/home/work/mongodb/log/master.log   
    pidfilepath=/home/work/mongodb/master.pid  
    directoryperdb=true  
    logappend=true  
    replSet=testrs  
    bind_ip=192.168.1.159  
    port=27017  
    oplogSize=10000  
    fork=true  
    noprealloc=true

在有足够空间的前提下：  

    $ ./bin/mongod -f master.conf   
    note: noprealloc may hurt performance in many applications  
    about to fork child process, waiting until server is ready for connections.  
    forked process: 25120  
    child process started successfully, parent exiting  


**slaver (192.168.1.171)**  

    #slaver.conf    
    dbpath=/home/work/mongodb/data/slaver   
    logpath=/home/work/mongodb/log/slaver.log   
    pidfilepath=/home/work/mongodb/slaver.pid   
    directoryperdb=true   
    logappend=true   
    replSet=testrs   
    bind_ip=192.168.1.171  
    port=27017   
    oplogSize=3000    
    fork=true   
    noprealloc=true  

在有足够空间的前提下：  

    $ ./mongodb-linux-x86_64-2.6.5/bin/mongod -f ./slaver.conf   
    note: noprealloc may hurt performance in many applications  
    about to fork child process, waiting until server is ready for connections.  
    forked process: 10126  
    child process started successfully, parent exiting  


**arbiter (192.168.1.189)**  

    #arbiter.conf    
    dbpath=/home/work/mongodb/data/arbiter    
    logpath=/home/work/mongodb/log/arbiter.log    
    pidfilepath=/home/work/mongodb/arbiter.pid    
    directoryperdb=true    
    logappend=true    
    replSet=testrs    
    bind_ip=192.168.1.189  
    port=27017    
    oplogSize=3000    
    fork=true    
    noprealloc=true    

在有足够空间的前提下：  

    $ ./mongodb-linux-x86_64-2.6.5/bin/mongod -f arbiter.conf   
    note: noprealloc may hurt performance in many applications  
    about to fork child process, waiting until server is ready for connections.  
    forked process: 7164  
    child process started successfully, parent exiting  

相应的服务启动好之后，创建集群  

    ./bin/mongo 192.168.1.159:27017  
    > use admin  
    > cfg={ _id:"testrs", members:[ {_id:0,host:'192.168.1.159:27017',priority:2}, {_id:1,host:'192.168.1.171:27017',priority:1},{_id:2,host:'192.168.1.189:27017',arbiterOnly:true}] };  
    > rs.initiate(cfg);  
    { "ok" : 0, "errmsg" : "couldn't initiate : new file allocation failure" }  

查看集群的状态命令  

    > rs.status()  

此时停掉 192.168.1.159节点上的mongodb服务，在查看状态, 159的状态 "health" : 0, 171变为 primary节点    

    > rs.status()  
    {  
        "set" : "testrs",  
        "date" : ISODate("2014-10-28T06:26:52Z"),  
        "myState" : 1,  
        "members" : [  
                {  
                        "_id" : 0,  
                        "name" : "192.168.1.159:27017",  
                        "health" : 0,  
                        "state" : 8,  
                        "stateStr" : "(not reachable/healthy)",  
                        "uptime" : 0,  
                        "optime" : Timestamp(1414477402, 1),  
                        "optimeDate" : ISODate("2014-10-28T06:23:22Z"),  
                        "lastHeartbeat" : ISODate("2014-10-28T06:26:49Z"),  
                        "lastHeartbeatRecv" : ISODate("2014-10-28T06:26:18Z"),  
                        "pingMs" : 0  
                },  
                {  
                        "_id" : 1,  
                        "name" : "192.168.1.171:27017",  
                        "health" : 1,  
                        "state" : 1,  
                        "stateStr" : "PRIMARY",  
                        "uptime" : 325,  
                        "optime" : Timestamp(1414477402, 1),  
                        "optimeDate" : ISODate("2014-10-28T06:23:22Z"),  
                        "electionTime" : Timestamp(1414477588, 1),  
                        "electionDate" : ISODate("2014-10-28T06:26:28Z"),  
                        "self" : true  
                },  
                {  
                        "_id" : 2,  
                        "name" : "192.168.1.189:27017",  
                        "health" : 1,  
                        "state" : 7,  
                        "stateStr" : "ARBITER",  
                        "uptime" : 218,  
                        "lastHeartbeat" : ISODate("2014-10-28T06:26:50Z"),  
                        "lastHeartbeatRecv" : ISODate("2014-10-28T06:26:50Z"),  
                        "pingMs" : 7  
                }  
        ],  
        "ok" : 1  
    }  

重启159上的mongdb，则159又为primary节点。  
  
参考博客： http://blog.csdn.net/luonanqin/article/details/8497860  

**问题：**  
使用上面的执行过程，在服务启动的时候，有个提示：  
"note: noprealloc may hurt performance in many applications  ", 看下mongodb的官网说明，  
--noprealloc  
Disables the preallocation of data files. This shortens the start up time in some cases and can cause significant performance penalties during normal operations.  


### sharding
参考：  
集群搭建: http://www.cnblogs.com/magialmoon/archive/2013/04/10/3013121.html   
sharding: http://www.cnblogs.com/magialmoon/archive/2013/04/11/3015394.html   
性能与优化: http://www.cnblogs.com/magialmoon/archive/2013/04/12/3017387.html   
  
路由节点 Query Routers:   
            192.168.1.159:27017   
配置节点 Config servers:   
            192.168.1.159:20001  192.168.1.171:20001  
数据节点 shards:     
            192.168.1.159:20002   192.168.1.171:20002   192.168.1.189:20002  
补充，官网上说明：The shard can be either a replica set or a standalone mongod instance  

  
**配置节点：**    

    #config.conf    
    dbpath=/data1/mongodb/data/config  
    logpath=/data1/mongodb/log/config.log    
    pidfilepath=/home/work/mongodb/config.pid  
    logappend=true    
    bind_ip=192.168.1.159    
    port=20001    
    oplogSize=3000    
    fork=true    
    noprealloc=true   


    $ ./mongod -f config.conf 

**路由节点：**  

    $ ./mongos --configdb 192.168.1.159:20001,192.168.1.171:20001 --port 27017 --fork --logpath /data1/mongodb/log/route.log 

使用两个配置节点的时候，提示：   

    BadValue need either 1 or 3 configdbs  

为了测试的简单，选择只启动一个config  

    $./mongos --configdb 192.168.1.159:20001 --port 27017 --fork --logpath /data1/mongodb/log/route.log  

**各个shard启动**

    #slave.conf    
    dbpath=/home/work/mongodb/data/shard  
    logpath=/home/work/mongodb/log/shard.log    
    pidfilepath=/home/work/mongodb/shard.pid    
    bind_ip=192.168.1.189  
    port=20002   
    oplogSize=3000   
    fork=true   
    noprealloc=true  


    $ numactl --interleave=all ./bin/mongod -f shard.conf  

**添加分片**  

    $ ./mongodb-linux-x86_64-2.6.5/bin/mongo 192.168.1.159:27017  
    > use admin  
    > db.runCommand({addshard:"192.168.1.159:20002", allowLocal:true})  
    { "shardAdded" : "shard0000", "ok" : 1 }   
    > db.runCommand({addshard:"192.168.1.171:20002"})  
    > db.runCommand({addshard:"192.168.1.189:20002"})   

**查看状态**  

    > use config  
    switched to db config  
    > db.shards.find()  
    { "_id" : "shard0000", "host" : "192.168.1.159:20002" }  
    { "_id" : "shard0001", "host" : "192.168.1.171:20002" }  
    { "_id" : "shard0002", "host" : "192.168.1.189:20002" }  

**移除shard**  

    > db.runCommand({removeShard: "192.168.1.189:20002"})  
    {     
        "msg" : "draining started successfully",  
        "state" : "started",  
        "shard" : "shard0002",  
        "ok" : 1  
    }  
    > db.runCommand({removeShard: "192.168.1.171:20002"})  
    { "ok" : 0, "errmsg" : "Can't have more than one draining shard at a time" }  

提示至少要有一个以上的shard, 移除后这个shard的状态   

    > db.shards.find()  
    { "_id" : "shard0000", "host" : "192.168.1.159:20002" }  
    { "_id" : "shard0001", "host" : "192.168.1.171:20002" }  
    { "_id" : "shard0002", "host" : "192.168.1.189:20002", "draining" : true } 

再执行一次remove操作，  

    > db.runCommand({removeShard: "192.168.1.189:20002"})  
    {  
        "msg" : "removeshard completed successfully",  
        "state" : "completed",  
        "shard" : "shard0002",  
        "ok" : 1  
    }  
  
如果是删除主节点的shard，则要执行"moveprimary".   
  
网上还有中方法，是将shard的三个节点搭建 Replica Set，然后  

    sh.addShard("test/192.168.1.159:20002")  #192.168.159:20002 是master的ip和端口  

从性能上考虑，replica set是一个节点写，另外节点复制信息。当写操作的数据量比较大的时候，性能会偏低些。  

**数据库使用shard**   

    > use admin  
    > db.runCommand({"enablesharding":"dbname1"})  

