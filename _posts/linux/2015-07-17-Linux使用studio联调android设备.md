---
title: Linux下使用Android Studio联调Android设备
date: 2015-07-17 14:12:28
tags: web
- a
- b
---

* toc 
{:toc}

> Windows 下IDE连接android设备做联调好像很容易；其实如果在Linux下能完成这个过程，发现其实也不是件很困难的事情。

## 手机设置
手机要处于USB可调试状态。不同手机上的设置不一样，此处不赘述。

## Linux识别设备

usb线连接移动设备和Linux电脑后，确保如下命令能看到设备
    
    $ adb devices
    List of devices attached 
    50536cf1    device

关于adb的安装，此处不介绍。本人使用的ubuntu系统，所需要的udev驱动等，系统默认的应该都已经有了。
而在识别设备时，遇到了一些问题，参看博客 [在Linux下adb连接不上android手机的终极解决方案](http://blog.csdn.net/liuqz2009/article/details/7942569){:target="_blank"}。

我在操作过程中，修改了博客中说到的两个地方

    $ lsusb
    Bus 001 Device 009: ID 2717:0368   # 安卓设备
    ... 
    $ cat /etc/udev/rules.d/51-android.rules
    SUBSYSTEMS=="usb", ATTRS{idVendor=="2717", ATTRS{idProduct}=="0368", MODE="0666", OWNER="jgj"} 
    $ cat ~/.android/adb_usb.ini
    0x2717


重新启动adb server
    
    
## 展示

在Android Studio 中选择一个项目，下面选择 “android”，然后选择连接的设备。
这个时候输出的日志会非常多，可以在做过滤。
    
