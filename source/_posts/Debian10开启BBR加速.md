---
title: Debian10开启BBR加速
date: 2021-01-21 16:49:08
categories: 技术向
tags: Linux
---
[BBR](https://github.com/google/bbr)是谷歌的一个开源的解决网络拥塞算法的，需要Linux的内核版本大于4.9才能开启。
<!--more-->
查看Linux内核版本：
```shell
uname -a
```
修改系统变量：
```shell
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```
保存生效：
```shell
sysctl -p
```
查看是否开启成功：
```shell
sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = bbr cubic reno
```