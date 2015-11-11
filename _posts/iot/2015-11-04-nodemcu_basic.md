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

## 使用mqtt协议远程操控

[Introduction to the MQTT Protocol on NodeMCU](http://www.allaboutcircuits.com/projects/introduction-to-the-mqtt-protocol-on-nodemcu/){:target="_blank"} 
介绍了使用使用开源的mqtt项目[mosquitto](http://mosquitto.org/){:target="_blank"}做服务端，[Paho Python Client](https://www.eclipse.org/paho/clients/python/)做客户端，
达到控制LED等的效果。

## 固件编译、烧写

在前面提到，烧写固件时，选用老版本固件会出现一些问题，于是决定使用 [github](https://github.com/nodemcu/nodemcu-firmware){:target="_blank"}上的代码，重新编译一份。

要使用相应编译工具链，tools/esp-open-sdk.tar.gz，解析得到xtensa-lx106-elf，

    $ export PATH=$PATH:/path/to/tools/esp-open-sdk/xtensa-lx106-elf/bin
    $ make
    $ ls bin
    0x00000.bin  0x10000.bin

这两个bin是刚生成的。使用esptools.py 命名烧写到板子上，参考 [esptool的github](https://github.com/themadinventor/esptool){:target="_blank"}
    
    $ sudo ./tools/esptool.py -p /dev/ttyUSB0 -b 9600 write_flash 0x00000 ./bin/0x00000.bin
    $ sudo ./tools/esptool.py -p /dev/ttyUSB0 -b 9600 write_flash 0x10000 ./bin/0x10000.bin

结果不好使。尝试使用flasher工具中自带的固件(git仓库下载后的 nodemcu-flasher-master/Resources/Binaries/nodemcu_integer_0.9.5_20150318.bin)，使用相同的方式写入，也是同样的问题：  
乱码显示后没有等待输入的界面。  
改变烧写模式，默认的是 `gio`，换为`dio`:

    $ ./tools/esptool.py write_flash -h
    ...
    --flash_mode {qio,qout,dio,dout}, -fm {qio,qout,dio,dout}
    ...
    $ sudo ./tools/esptool.py -p /dev/ttyUSB0 115200 write_flash -fm dio 0x10000 /path/to/nodemcu_integer_0.9.5_20150318.bin

烧写完成后，使用minicom连接，RTS重启下，就进入 输入界面，可以正常交互了。发现上面-b 设置不同的波特率，会影响烧写的速度，除此之外没有看到别的影响。使用这种方式，烧写编译好的两个bin文件

    $ sudo ./esptool.py write_flash -fm dio 0x00000 ../nodemcu-firmware/bin/0x00000.bin
    $ sudo ./esptool.py write_flash -fm dio 0x10000 ../nodemcu-firmware/bin/0x10000.bin

烧写完成后，第一次进入，有个提示  Self adjust flash size，然后没有反应。重新使用minicom连接，就可以正常交互了。

## 代码加载与执行

在前面的 `minicom 控制`和`使用mqtt协议远程操控`章节，都应用了 `http://www.allaboutcircuits.com/` 上面的文章，里面用到了工具[luatool](https://github.com/4refr0nt/luatool){:target="_blank"}。

    $ ./luatool.py -h
    $ ./luatool.py -f net.lua -t net.lua -v 
        # net.lua文件写到板子上，也命名为net.lua,-v显示过程
    $ ./luatool.py -f net.lua -t net.lua -d -v
        # -d 加载完成后执行该文件 dofile

## 总结

到此，在Linux下可以完成Nodemcu的一些最基本操作了。用到了工具 esptool、minicom、luatool等，因为ttyUSB0端口是单操作的，所以在使用执行esptool 或 luatool等命令是，minicom要断开。
