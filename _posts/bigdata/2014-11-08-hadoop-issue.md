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
