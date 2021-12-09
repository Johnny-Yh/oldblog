---
title: 给博客加上HTTPS
date: 2020-03-31 14:39:17
categories: 技术向
tags: Https
---
## 前言 ##
Trojan可以让自己的博客加上**Https**访问。
<!--more-->
下面这张图是部分文档的介绍：
![trojan](https://cdn.jsdelivr.net/gh/Johnny-Yh/ImageHosting//img/200331-4.jpg)
它可以仅仅是一个Https的服务器，监听系统的443端口，展开Tls握手，握手成功后，如果判断是其他流量，就会打开一个隧道，将流量转发到预设好的端口，这个端口就可以处理解密后的Http流量了。
下面这张图可以更好地理解这样的工作模式：
![work](https://cdn.jsdelivr.net/gh/Johnny-Yh/ImageHosting//img/200331-3.jpg)
当客户端访问服务器时，服务端监听443端口，如果是客户端的合法流量，那么就由Trojan处理，再访问互联网，如果不是客户端的合法流量（比如浏览器访问服务器上的Https网站），那么在进行Https流量解密之后，服务端会将Http流量转发给nginx监听的80端口，交给Nginx处理，从而使用户可以正常访问服务器上的网站。
## 开始操作 ##
 1. 首先确认自己的博客能够正常地通过Http访问
 2. 申请一张Https证书，并准备好证书和密钥文件,放在你的服务器的合适位置
 3. 安装Trojan，使用Github上的安装脚本
通过curl命令安装：
```shell
sudo bash -c "$(curl -fsSL https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"
```
或者通过wget命令安装：
```shell
sudo bash -c "$(wget -O- https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"
```
 4. 修改/usr/local/etc/trojan/config.json文件
![edit](https://cdn.jsdelivr.net/gh/Johnny-Yh/ImageHosting//img/200331-2.jpg)

 5. 设置trojan自动启动
```shell
systemctl start trojan
systemctl enable trojan
```
 6. 编辑Typecho站点根目录下的文件config.inc.php加入下面一行配置：
```shell
/** 开启HTTPS */
define('__TYPECHO_SECURE__',true);
```
    注意：Chrome内核的浏览器有不安全提示的时候，需要找到主题目录下面的comments.php文件再将其中的  
    $this>commentUrl()，替换为：
```shell        
echo str_replace("http","https",$this->commentUrl());
```
    最后保存。
![success](https://cdn.jsdelivr.net/gh/Johnny-Yh/ImageHosting//img/200331-1.jpg)