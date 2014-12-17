---
title:  一些可能会用到的bash shell
date: 2014-12-17 18:12:28 +0800
tags: Linux
- a
- b
---

* toc 
{:toc}

> 本文记录在实际工作中用到的一些Bash Shell，可能比较偏门，但是方便以后使用

###文件重命名

参考 [Linux批量重命名文件 ](http://snailwarrior.blog.51cto.com/680306/139706){:target="_blank"}。  
我想把它们的名字的第一个1个字母变为"q"，其它的不变： 

    for i in `ls`; do mv -f $i `echo $i | sed 's/^./q/'`; done
    
修改前面5个字母为zhaozh：

    for i in `ls`; do mv -f $i `echo $i | sed 's/^...../zhaozh/'`; done
    
修改后面5个字母为snail：
    
    for i in `ls`; do mv -f $i `echo $i | sed 's/.....$/snail/'`; done
    
在前面添加 _hoho_：
    
    for i in `ls`; do mv -f $i `echo "_hoho_"$i`; done
    
所有的小写字母变大写字母：

    for i in `ls`; do mv -f $i `echo $i | tr a-z A-Z`; done
    
多层目录下的文件，将TC开头的文件换乘MJ

    for i in `find ./ -name TC*`; do mv -f $i `echo $i | sed 's/\/TC/\/MJ/'`; done
    
将done改为update
    
    for i in `find ./ -name done`; do mv $i `echo $i | sed 's/done/update/'` ; done
    
###文件内容修改

第一种比较笨的办法，是将修改后的内容存到另一文件中，然后替换

    for i in `find ./ -type f`; do cat $i |  sed -e "s/AAAA/BBBBB/g" -e "s/CCCCC/DDDDD/g" > bak ; mv bak $i; done

也可以直接用 `sed -i` 命令  

###删除比xx更新的文件   
    
    for i in `find ./ \! -newer 1390534720.snapshot ` ; do rm $i ; done   ##older with \! -newer
    
find用户 -prune是除去这个目录
    
###Shell中一些单引号

sed中单引号

    echo "dds'fa" | sed -e "s#'##g"
    
awk中单引号

    echo "ddd'fs" | awk -F"'" '{print $1}'
    
awk打印单引号

    echo "ddd'fs" | awk -F"'" '{printf("%s \047 \n",$1) }'
    
### 正则表达式反向引用

在前面匹配的时候，一定要用()表示出来

    echo "optional string v = 1;" | sed -n -r 's/optional (int32|int64) (.*)/optional \1 \2 \/\/ \1  /p'
    
vim中的()要用反斜线 

    :'<,'>s/get_\(.*\);/get_\1() { return \1; }/g        
    :'<,'>s/set_\(.*\);/set_\1(int m_\1) { this->\1 = m_\1; }/g
    :%s/ok|\([0-9]\)|\([0-9.]*\)/ok|\1,price|\2,imei/g
    
###一些指定的文件大小和

    du -sh 21_20131001* | awk -F'M' '{sum+=$1} END {print sum}'

这儿使用M做awk的分隔符

###打包scp操作

参考 [ssh tar](http://www.thingy-ma-jig.co.uk/blog/03-09-2008/using-tar-and-ssh-improve-scp-speeds){:target="_blank"}  

    (cd /lib/modules; tar zcvf - 2.6.19 ) | ssh -p 60001 root@192.168.0.254 tar zxvf - -C /lib/modules/
    ssh 192.168.1.2 -T -c arcfour -o Compression=no -x "tar cf - /remote/path" | tar xf - -C .
    ssh -p 60020 work@ck -T -c arcfour -o Compression=no -x "cd /home/work/pb/base_pb/201311/; tar cf - 06" | tar xf - -C .
    tar czf - www.example.com/ | ssh joebloggs@otherserver.com tar xzf - -C ~/

一些说明    
T: turn off pseudo-tty to decrease cpu load on destination.  
c: arcfour: use the weakest but fastest SSH encryption. Must specify "Ciphers arcfour" in sshd_config on destination.  
o: Compression=no: Turn off SSH compression.
x: turn off X forwarding if it is on by default.

###sed拆分

    sed -n '10001,10200'p stats_21log > stats_11
    for i in {1..11}; do start_line=`echo $i*1000-999 | bc`; end_line=`echo $i*1000 | bc`; sed -n "$start_line,${end_line}p" stats_21log > stats_$i ; done
    
注意，计算时使用bc，sed中使用双引号

###获取字符串长度

    log_path_len=`expr length $log_path`
    datetime_path=${log_dir:${log_path_len}:50}  

后者可以将字符串后面的字段获取到， log_dir开头的内容时log_path  

###增加虚拟ip

    ifconfig eth0:1 192.168.100.11 netmask 255.255.255.0
    ifconfig eth0:2 192.168.100.12 netmask 255.255.255.0
    
###awk 输出时改变分隔符

    echo "ab cd ds" | awk '{OFS=":"; $2="*";  print $0}' //如果没有$2="*" 这条，OFS=":"就不起作用
    echo a b c d|awk '{OFS=":";print $1,$2,$3,$4}'    //这样也可以
    echo a b c d|awk '{OFS=":";print $1 OFS $2,$3,$4}' //也可以
    echo "ab:bc:d" | awk -F':'  '{print | "cut -d : -f -"(NF-1)}'      #delete the last field
    
前N个：cut -d 分隔符 -f -N
第X个：cut -d 分隔符 -f X
最后一个：awk -F 分隔符 '{pint $NF}'
去除最后一个：

    awk -F 分隔符 '{print | "cut -d 分隔符 -f -"(NF-1)}'
    
去除指定字段X：

    awk -F 分隔符 ’{print | "cut -d 分隔符 -f 1-(X-1),(X+1)-"(NF)}' 
    
###grep的正则表达式

    grep -o "device=.*&" file   --- 会时贪婪搜索
    grep -o "device=.*?&" file  --- shell的正则中不支持？的懒惰
    grep -o "device=[^&]*" file --- 得到预期的到&之间的数据
    
###awk 几个例子

   awk '{a[$1]+=$2;}END{for(i in a){if(a[i]>8){print i" "a[i];}}}' aa

   cat */result_201* | awk -F' ' '{tmp=$1; $1="";gsub(/ /, "-"); val[$0]+=tmp}END{for(i in val) {print val[i]" "i};}' | sort -n -r > total 

多个文件中包含设备类型及个数，合并求和 (大写转为小写)

   cat *_201406 | tr "[:upper:]" "[:lower:]" | awk '{names[$2]+=$1} END {for (i in names) {print names[i] " " i} }'

###grep 非ascii

打印行数，并高亮显示非ascii码字符

    grep --color='auto' -P -n "[\x80-\xFF]" file.xml
    

###bzip2 -d文件失败的修复

    bzip2recover file.bz2
    bzip2 -d rec*.bz2 > file    
