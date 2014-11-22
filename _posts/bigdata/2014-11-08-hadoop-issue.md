---
title: hadoop 问题汇总
date: 2014-11-08 20:12:28 +0800
tags: bigdta
- a
- b
---

* toc 
{:toc}


hadoop 使用过程中遇到的问题汇总（版本2.2） 

### map执行100%，reduce任务不执行  
* 定位  

    `yarn-work-resourcemanage-host-151.log`日志文件一直在输出： 

        capacity.LeafQueue (LeafQueue.java:assignToQueue(946)) - default usedResources: <memory:2048, vCores:1> clusterResources: <memory:4096, vCores:2> currentCapacity 0.5 required <memory:4096, vCores:1> potentialNewCapacity: 1.5 (  max-capacity: 1.0)

* 处理  

    ` yarn-site.xml `中` yarn.nodemanager.resource.memory-mb `设置为了` 4096 `, 如果改为2048的话，则map也不执行了。查看执行mapreduce的任务，有个启动参数 ` -D 'mapred.job.reduce.memory.mb='4096''  `，将其改为2048则就可以执行了。
    
* 分析  
  ok，可以看到在hadoop的配置中，特别是关于分配的内存大小有很多的选项，这么多的选项都是做什么的，在什么地方使用呢。 

###reduce返回255

* 定位

运行mapreduce时，出现如下的错误：

    14/11/22 14:45:02 INFO mapreduce.Job:  map 100% reduce 0%
    14/11/22 14:45:10 INFO mapreduce.Job: Task Id : attempt_1406010920358_60903_r_000000_0, Status : FAILED
    Error: java.lang.RuntimeException: PipeMapRed.waitOutputThreads(): subprocess failed with code 255
        at org.apache.hadoop.streaming.PipeMapRed.waitOutputThreads(PipeMapRed.java:320)
        at org.apache.hadoop.streaming.PipeMapRed.mapRedFinished(PipeMapRed.java:533)
        at org.apache.hadoop.streaming.PipeReducer.close(PipeReducer.java:134)
        at org.apache.hadoop.io.IOUtils.cleanup(IOUtils.java:237)
        at org.apache.hadoop.mapred.ReduceTask.runOldReducer(ReduceTask.java:477)
        at org.apache.hadoop.mapred.ReduceTask.run(ReduceTask.java:408)
        at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:162)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:415)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1491)
        at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:157)

搜索`failed with code 255`，得到信息是reduce运行错误，返回 `-1` 。看hadoop的reduce的日志，发现时解析应用列表文件出错。


注：hadoop的mapreduce历史记录通过mapreduce接口网站查看，进到每个任务的`logs`中，不只是看 `Note` 的内容。如果不是通过网页，而是直接在机器上看日志，或者hadoop上的文件，目前还没有找到在什么地方存放！


* 处理

发现时解析应用列表的json文件出错，而实际上这个文件的json内容是正确的，解析不应该出错。启动mapreduce的时候，配置项如下：

    -file $DIR_ROOT/conf/app.conf \
    ... ...
    -cmdenv APP_CONFIG="$DIR_ROOT/conf/app.conf" \

经咨询，原来是 -cmdenv指定使用哪个文件是，已经用file指定路径和名称了，就不需要在指定路径了，所以做如下修改即可了

    -file $DIR_ROOT/conf/app.conf \
    ... ...
    -cmdenv APP_CONFIG="app.conf" \

