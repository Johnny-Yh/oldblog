---
title: 在Openwrt下使用Kms激活
date: 2021-02-20 00:19:24
tags: Openwrt
---
自己使用的路由器是Openwrt系统的，最近重装了系统发现Windows和Office都变成了未激活状态，突然想起来路由器里自带Kms服务器可以激活，顺便将自己的操作记录下来，以后再重装就不需要去百度了。
<!--more-->

Windows激活：

1. 在Powershell输入slmgr /upk卸载原来自带密钥
2. 输入slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX，使用的是Win10专业版的Kms
3. 输入slmgr /skms 192.168.1.1，将Kms的服务器地址设为自己的路由器地址
4. 输入slmgr /ato，激活Windows

Office激活：

1. 在Powershell进入安装Office的目录，cd “C:\Program Files\Microsoft Office\Office16”
2. 输入cscript ospp.vbs /sethst:192.168.50.1
3. 输入cscript ospp.vbs /act

现在Windows和Office都激活成功啦！

参考链接：

- http://www.vjsun.com/629.html
- https://github.com/netnr/kms