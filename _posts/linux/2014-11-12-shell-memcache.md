---
title: memcache
date: 2014-11-12 16:12:28 +0800
tags: linux 
- a
- b
---

* toc 
{:toc}



本文是memcache的一些基本操作，以及引申出来一些小观点。

## Shell操作memcache

###use printf  

张宴的博客  [利用shell命令操作Memcached](http://zyan.cc/post/384/){:target="_blank"}给出几个例子，如

    $ printf "set jgj 0 0 3\r\n123\r\n" | nc 192.168.1.135 11211
    STORED
    $ printf "get jgj\r\n" | nc 192.168.1.135 11211
    VALUE jgj 0 3
    123
    END
    $ printf "delete jgj\r\n" | nc 192.168.1.135 11211
    DELETED

###use echo  
    
既然`printf`可以操作，`echo` 应该也是可以操作，经过一些测试，得到

    $ echo -e "set jgj 0 0 3\n123\r" | nc 192.168.1.135 11211
    STORED
    $ echo "get jgj" |nc  192.168.1.135 11211
    VALUE jgj 0 3
    123
    END
    $ echo "delete jgj" | nc 192.168.1.135 11211
    DELETED

基于此，就可以类推memcache的其他命令的使用了。

###/r和/n有和不同

上面使用echo的操作时，如果使用命令

    $ echo -e "set jgj 0 0 3\r\n123\r\n" | nc 192.168.1.135 11211
    STORED
    ERROR

虽然也执行成功，但是有个错误。所以\r和\n有什么不同呢？觉得 “回车(CR)与换行(LF)， '\r'和'\n'的区别” 这篇博客说的比较清楚（网上互相转来转去，也不知道是谁原创了）。其中的几个关键点  
1. 回车CR-将光标移动到当前行的开头；换行LF-将光标“垂直”移动到下一行。（而并不移动到下一行的开头，即不改变光标水平位置）  
2. CR用符号'\r'表示, 十进制ASCII代码是13, 十六进制代码为0x0D；LF使用'\n'符号表示, ASCII代码是10, 十六制为0x0A   
3. Dos和windows采用回车+换行CR/LF表示下一行；而UNIX/Linux采用换行符LF表示下一行；苹果机(MAC OS系统)则采用回车符CR表示下一行  
4. 有兴趣的可以看看这两个符号的来源(电传打字机时代开始的)


###nc命令

如其名字所表示的，牛叉的意思，网络界的“瑞士军刀”。  
**打开特定端口**

    nc -l 81   ## 之后进入等待状态，1234是本机（192.168.1.130）未使用的一个端口
    telent 192.168.1.130 81 ##使用telent通过81端口登录，此时输入的数据，在上面的终端同样会输出


**文件传输**   

    nc -l 1234 > received_file          # 1234是本机（192.168.1.130）未使用的一个端口
    nc 192.168.1.130 1234 < test_file  # 在要传输的机器上执行
    
执行完之后，received_file中拥有和test_file同样的内容，上面的操作可以使用命令：  

    ssh work@192.168.1.130 “( nc -l 1234 > received_file 2>/dev/null & )” && cat test_file | nc 192.168.1.130 1234

    
**扫描端口**   

    nc -v -z 192.168.1.130 70-90
    
想到了之前常用的扫面端口和机器的命令，nmap  
    
    nmap 192.168.1.130
    nmap -sP 192.168.1.1/24
    
更多功能，期待挖掘！

###关于博客

搜一些linux命令的时候，经常在一遍显示其他人还搜索 “张宴”，今天看到他的博客才注意起来。85年的，竟有如此不菲的成就。应该也是平时的一点一滴积累下来的。


##查看memcache中的key

想查看memcache中都已经有了些什么样的key，类似于redis那样keys *，参考 [Memcached: List all keys](http://www.darkcoding.net/software/memcached-list-all-keys/){:target="_blank"}
    
    $ telnet 192.168.1.135 11211
    Connected to 192.168.1.135.
    
    stats items
    
    STAT items:1:number 4
    ...
    STAT items:4:number 1
    ...
    END
    
    stats cachedump 1 0  ## 前面的1是items的标号，0，是要查询的item格式，当为0时，表示查询全部
    ITEM jgj [3 b; 1415763334 s]
    ITEM jin [3 b; 1415763334 s]
    ITEM key1 [3 b; 1415763334 s]
    END
    
    stats cachedump 4 0
    ITEM 127.0.0.1 [88 b; 1415763334 s]
    END

