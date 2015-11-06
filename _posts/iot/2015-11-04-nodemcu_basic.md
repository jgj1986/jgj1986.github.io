---
title: nodemcu学习笔记
date: 2015-11-04 10:12:28
tags: iot
- a
- b
---

* toc 
{:toc}

## 入门
NodeMcu论坛上有一篇比较好的入门文章[NodeMcu三小时入门](http://bbs.nodemcu.com/t/nodemcu/104){:target="_blank"}，开始时按照里面的步骤，烧写固件。
这个过程中遇到了两个问题：
1、USB驱动，提示的的CP2102 usb to uart bridge controller 的驱动有问题，那就不要在使用CH340的驱动了。Windows7系统自己不能识别，则网上搜集CP2102的驱动下载安装。
2、固件烧写，曾经尝试烧写了两个版本，2014xx和20150122的，但是之后在连接时没有输出，或者输出不能输入，是因为固件版本还是太低，而硬件已经是高级别的硬件了。到github上去下载最近的[flasher](https://github.com/nodemcu/nodemcu-flasher){:target="_blank"}和[固件](https://github.com/nodemcu/nodemcu-firmware){:target="_blank"}。使用较新版本烧写之后连接，则可以测试lua的helloworld了。

后续测试连接wifi，获取网络地址的时候，可能要等待一段时间才得到ip地址。

## minicom控制

Linux上使用minicom也可以控制NodeMcu，将已经烧写过固件的板子连接到Linux系统上，会有 /dev/ttyUSB0。
minicom中的一些配置如下: 

    Serial Device: /dev/ttyUSB0
    Bps/Par/Bits    : 9600 8N1
    Hardware Flow Control                       : No
    Software Flow Control : No

之后 RST 键重启下开发板，
        
    NodeMCU 0.9.5 build 20150318  powered by Lua 5.1.4                                                                                                     
    lua: cannot open init.lua                                                                                                                          
    > print("helloworld world") 
    hello world

参考: [How to Make an Interactive TCP Server    with NodeMCU on the ESP8266](http://www.allaboutcircuits.com/projects/how-to-make-an-interactive-tcp-server-nodemcu-on-the-esp8266/){:target="_blank"}，

## Arduino

还可以使用Arduino来操控nodemcu，[Getting Started with Noduino on Windows](http://wiki.jackslab.org/Getting_Started_with_Noduino_on_Windows){:target="_blank"}

