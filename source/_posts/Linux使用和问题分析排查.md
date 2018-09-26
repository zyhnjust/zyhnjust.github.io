---
title: Linux使用和问题分析排查
date: 2018-09-26 15:45:15
tags:
- 面试
---


# 两种命令创建一个文件

touch

create

# 硬链接和软连接的区别

ln 默认是硬链接， 指定-s 就是软链接。 

如果硬链接删除， 软连接删除， 对本身都没有影响。 
删除本身， 对硬链接没有影响， 软连无效。 

硬连接文件f2与原文件f1的inode节点相同，均为9797648，然而符号连接文件的inode节点不同。



# linux 常用命令

很多

# 如何看java 线程的资源耗用

top可以么

ps ux  # 进程

# load 过高的可能性哪些

确实过高， 内存计算。 

也许死循环


# etc hosts 用途

DNS 服务

用hostname 也可以。 不过是临时性的。 

# 如何快速把文本abc 替换成xyz

- vim 可以用 :s/aaa/bbb/g

- 外面 sed
- sed -e ‘s/foo/bar/’ myfile.txt

# 如何log中搜索error的日志

- grep -i "Error" xx.file

# 空间不够如何找到空间最大的文件

du : 计算出单个文件或者文件夹的磁盘空间占用.
    -  -a              write counts for all files, not just directories
    -  -h  human readable 
sort : 对文件行或者标准输出行记录排序后输出.
    -r               reverse the result of comparisons
    
head : 输出文件内容的前面部分.

- du -a /var | sort -n -r | head -n 10


# java 服务端问题排查 （OOM CPU 高， load 高， 类冲突）

- top 看内存 cpu
- 看log。 
- 打dump。 可以用jmap， 

# java 常用问题排查以及用法 - top, iostat, vmstat, star, tcpdump, jvisualvm, jmap, jconsole

- top process
- iostat
-  - ubuntu need install.  参数 -d 表示，显示设备（磁盘）使用状态；-k某些使用block为单位的列强制使用Kilobytes为单位；2表示，数据显示每隔2秒刷新一次。

- vmstat
- 可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况。这个命令是我查看Linux/Unix最喜爱的命令，一个是Linux/Unix都支持，二是相比top，我可以看到整个机器的CPU,内存,IO的使用情况，而不是单单看到各个进程的CPU使用率和内存使用率(使用场景不一样)。
- vmstat 2 1    第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数
- 2表示每个两秒采集一次服务器状态，1表示只采集一次。
- https://www.cnblogs.com/ggjucheng/archive/2012/01/05/2312625.html

-  使用iostat 命令

查看r/s(读请求),w/s(写请求),avgrq-sz(平均请求大小),await(IO等待), svctm(IO响应时间)
    r/s ,w/s是每秒读/写请求的次数。

   util是设备的利用率。如果它接近100%，通常说明设备能力趋于饱和(并不绝对，比如设备有写缓存)。有时候可能会出现大于100%的情况，这多半是计算时四舍五入引起的。
    svctm是平均每次请求的服务时间。这里有一个公式：(r/s+w/s)*(svctm/1000)=util。举例子：如果util达到100%，那么此时  svctm=1000/(r/s+w/s)，假设IOPS是1000，则svctm大概在1毫秒左右，如果长时间大于这个数值，说明系统出了问题。
   await是平均每次请求的等待时间。这个时间包括了队列时间和服务时间，也就是说，一般情况下，await大于svctm，它们的差值越小，队列时间越短，反之差值越大，队列时间越长，说明系统出了问题。
avgqu-sz是平均请求队列的长度。毫无疑问，队列长度越短越好。


其他 jvisualvm 要显示的 

jmap 是 map

jconsole

# Thread dump 如何分析

用MAT可以看 。 之前看过内存过大的问题。 

# 如何看java 应用的线程信息。 

TODO
1. 一个监控脚本 thread 。 https://github.com/oldratlee/useful-scripts/blob/master/docs/java.md#-show-busy-java-threads
2. 

