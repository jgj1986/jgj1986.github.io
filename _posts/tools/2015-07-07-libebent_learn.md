---
title: libevent使用学习
date: 2015-07-07 15:12:28
tags: tools 
- a
- b
---

* toc 
{:toc}

>

> 最近接收的项目中用到了libevent的机制，为了更好的理解项目，也就学习了一下libevent。

## 事件处理机制

[libevent源码深度剖析](blog.csdn.net/sparkliang/article/category/660506){:target="_blank"}系列文章好好读一遍，简单的记录比较有启发的一些知识点。

应用程序需要提供相应的接口并注册到Reactor上，作为“回调函数”。  
其执行顺序:  
* 初始化libevent库、初始化事件event  
* 设置event从属的event_base  
* 添加事件  
* 进入无限循环 event_base_dispatch(base);

  
## 简单例子

参考 [libevent简介和使用](http://blog.csdn.net/yyyiran/article/details/12219737){:target="_blank"}中简单入门的例子，

    #include <stdio.h>  
    #include <iostream>   
    #include <event.h>   
      
    // 定时事件回调函数  
    void onTime(int sock, short event, void *arg) {  
        cout << "sub process!" << endl;      
        struct timeval tv;  
        tv.tv_sec = 1;  
        tv.tv_usec = 0;  
        // 重新添加定时事件（定时事件触发后默认自动删除）  
        event_add((struct event*)arg, &tv);  
    }  
      
    int main() {  
        // 初始化  
        event_init();  
      
        struct event evTime;  
        // 设置定时事件  
        evtimer_set(&evTime, onTime, &evTime);  
      
        struct timeval tv;  
        tv.tv_sec = 1;  
        tv.tv_usec = 0;  
        // 添加定时事件  
        event_add(&evTime, &tv);  
       
        event_dispatch();     
        return 0;  
    }  
    
实际过程中更常用的方式是：

    base = event_base_new();     
    event_base_set(base, &evListen);  
    // 添加事件  
    event_add(&evListen, NULL);        
    // 事件循环  
    event_base_dispatch(base);
    
## 连接监听器
系统这样设计的：有多个进程(10)个，对外提供接口服务，一个进程做监控，当服务的某个进程挂掉后，监控进程会将对应(端口)的服务在启动起来。
在线上服务运行时遇到这样一个现象，每个服务进程在执行完成之后，都就没有了，监控进程又重新启动起来了。
因为使用了libevent中的 `evconnlistener_new_bind` 来创建这样的服务，好好研究下怎么使用。
最后发现，在服务代码中两次释放一个内存地址，导致core dump。但是在这个过程中还是好好的了解了一下连接监听器的使用。
博客[Libevent源码分析-----连接监听器evconnlistener](http://blog.csdn.net/luotuo44/article/details/38800363){:target="_blank"}  
从如何使用到代码的实现等方面对此作了比较详细的介绍，这里只简单的列出一些自己觉得如果比较有用的代码。

    event_base *base = event_base_new();  
    evconnlistener *listener  
            = evconnlistener_new_bind(base, listener_cb, base,  
                LEV_OPT_REUSEABLE|LEV_OPT_CLOSE_ON_FREE | LEV_OPT_THREADSAFE,  
                10, (struct sockaddr*)&sin,  
                sizeof(struct sockaddr_in));  
  
    event_base_dispatch(base);  
    evconnlistener_free(listener);  
    event_base_free(base);
    
创建好连接之后，使用 `event_base_dispatch` 进入到了循环等待。 `listener_cb`可以如下的形式实现：

    void listener_cb(evconnlistener *listener, evutil_socket_t fd,
        struct sockaddr *sock, int socklen, void *arg) {
        event_base *base = (event_base*)arg;
        bufferevent *bev =  bufferevent_socket_new(base, fd, BEV_OPT_CLOSE_ON_FREE);    
        
    bufferevent_setcb(bev, socket_read_cb, NULL, socket_error_cb, NULL);
        bufferevent_enable(bev, EV_READ | EV_PERSIST);
    }

    
