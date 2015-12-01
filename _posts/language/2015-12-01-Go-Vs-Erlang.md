---
title:  Go-VS-Erlang
date: 2015-12-01 11:12:28 +0800
tags: iot
- a
- b
---

* toc
{:toc}


## 国内的一些讨论

[Erlang和Golang的几点粗浅的比较](http://www.douban.com/note/233256219/):target="_blank"}、
[给自己一条退路，再次比较Erlang和Golang](http://blog.sina.com.cn/s/blog_6e1bd8350102uwgl.html):target="_blank"}、
[为什么我要选择erlang+go进行服务器架构(2)](http://studygolang.com/articles/912):target="_blank"}。

基本验证如下观点，语言各种各的特点，但是没有哪个语言能胜任各个场景，且比其他的都好。

1. Erlang 语法比较小众，入门时间会相对长一些；而从常见的语言切换到Go则比较方便  
2. Erlang 非常适用于高并发、高稳定环境，成熟度比较高，计算密集型则表现的相对差一些；而Go在计算上更突出  
3. Erlang成熟，但库相对少一些，Go在蓬勃发展  
4. 就像最后一个介绍的，两者也可以结合适用！  

## 国外的一些比较

[Concurrency Models: Go vs Erlang](http://joneisen.tumblr.com/post/38188396218/concurrency-models-go-vs-erlang){:target="_blank"}; 
[Some Thoughts on Go and Erlang](http://blog.erlware.org/some-thoughts-on-go-and-erlang/){:target="_blank"}。

## 现实中的比较

Mosquitto 开发组的成员使用Go、C、D、Erlang等语言实现了MQTT Broker的核心功能，并做了一些基本的测试，将这个语言在实际使用做了比较，[GO VS D VS ERLANG VS C IN REAL LIFE: MQTT BROKER IMPLEMENTATION SHOOTOUT](https://atilanevesoncode.wordpress.com/2013/12/05/go-vs-d-vs-erlang-vs-c-in-real-life-mqtt-broker-implementation-shootout/){:target="_blank"}。

    loadtest (throughput - bigger is better)
    Connections:   100            500            750            1k
    D + vibe.d:    121.7 +/- 1.5  166.9 +/- 1.5  171.1 +/- 3.3  167.9 +/- 1.3
    C (Mosquitto): 106.1 +/- 0.8  122.4 +/- 0.4   95.2 +/- 1.3   74.7 +/- 0.4
    Erlang:        104.1 +/- 2.2  124.2 +/- 5.9  117.6 +/- 4.6  117.7 +/- 3.2
    Go:             90.9 +/- 11   100.1 +/- 0.1   99.3 +/- 0.2   98.8 +/- 0.3

    pingtest (latency - bigger is better)
    parameters:    400p 20w       200p 200w      100p 400w
    D + vibe.d:    50.9 +/- 0.3   38.3 +/- 0.2   20.1 +/- 0.1
    C (Mosquitto): 65.4 +/- 4.4   45.2 +/- 0.2   20.0 +/- 0.0
    Erlang:        49.1 +/- 0.8   30.9 +/- 0.3   15.6 +/- 0.1
    Go:            45.2 +/- 0.2   27.5 +/- 0.1   16.0 +/- 0.1

文中谈到，对Go开发时使用的V1.1的版本，而测试用的是V1.2的版本，预计改善因版本问题而导致的bug后，会有10%的提升，和Erlang的新能几乎一样。

当然，文中隐含的提到一点，对Erlang不懂，无法评价评论。在一定程度上说明Erlang 语法确实有些小众。

## 总结

不要停留在语言层面了，遇到实际场景，什么合适用什么吧。

