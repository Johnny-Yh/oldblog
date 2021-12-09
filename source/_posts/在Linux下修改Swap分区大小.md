---
title: 在Linux下修改Swap分区大小
date: 2020-03-18 11:46:55
categories: 技术向
tags: Linux
---
今天无意间看到主机的swap的大小只有131M，而且占用也快满了，想要修改swap分区大小，改为两倍系统物理内存的大小（1024M），以满足日常使用的需求。

<!--more-->

swap分区：通常被称为交换分区，这是一块特殊的硬盘空间，即当实际内存不够用的时候，操作系统会从内存中取出一部分暂时不用的数据，放在交换分区中，从而为当前运行的程序腾出足够的内存空间。

使用free命令查看系统内存的使用情况：

    [root@PC ~]# free -m
              total        used        free      shared  buff/cache   available
     Mem:       493         184         112          6         195         289                       
     Swap:      131           0         131                                    


可以看到我们这台主机内存有493M，而swap分区只有131M（我之前重新挂载过一次分区，所以当前使用为0M）

使用swapon命令查看swap分区：

    [root@PC ~]# swapon -s
     Filename				Type		Size	Used	Priority               
     /swap                   file      135164	   0	      -2

可以看到swap分区的位置为/swap

使用swapoff命令关闭swap分区：

    [root@PC ~]# swapoff -a

再次查看系统内存：

    [root@PC ~]# free -m
              total        used        free      shared  buff/cache   available
     Mem:       493         212          41           6         239         261
     Swap:        0           0           0

可以看到swap分区的空间已经变成了0
删除当前的swap分区文件：

    [root@PC ~]# rm /swap
     rm: remove regular file '/swap'? y

重新创建swap分区：
dd命令的部分参数如下：参数bs为一个块的大小，参数count为总的块数

    [root@PC ~]# dd if=/dev/zero of=/swap count=1024 bs=1M
     1024+0 records in
     1024+0 records out
     1073741824 bytes (1.1 GB, 1.0 GiB) copied, 3.59162 s, 299 MB/s

查看swap分区：

    [root@PC ~]# ls / | grep swap
     swap

可以看到swap已经成功重新创建

为swap文件赋予权限（文件所有者的读写权限）：

    [root@PC ~]# chmod 600 /swap

查看swap文件权限：

    [root@PC ~]# ls -lh /swap
     -rw------- 1 root root 1.0G Mar 17 23:41 /swap

挂载swap分区：

    [root@PC ~]# mkswap /swap
     Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
     no label, UUID=c3d2c1dc-adb9-468e-8ab4-fbdc8fead879

打开swap分区：

    [root@PC ~]# swapon /swap

设置开机启动：

    [root@PC ~]# vi /etc/fstab

在文件末尾添加如下代码并保存：

    /swap none swap sw 0 0

再次查看系统内存使用情况：

    [root@PC ~]# free -m
              total        used        free      shared  buff/cache   available
     Mem:       493         214           6           6         272         259
     Swap:     1023           0        1023

swap分区文件大小的修改就完成了！