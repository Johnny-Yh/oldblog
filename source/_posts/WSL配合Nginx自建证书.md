---
title: WSL配合Nginx自建证书
date: 2021-12-09 19:00:56
categories: 技术向
tags: Https
---
在学校的信息安全实践课上，第一个实验就是需要自建CA证书搭建HTTPS服务器，为了方便，我使用[WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux)(Windows Subsystem for Linux)配合Nginx和OpenSSL来自建CA证书。
<!--more-->
WSL系统：[Ubuntu 20.04.3 LTS](https://www.microsoft.com/store/productId/9NBLGGH4MSV6)

## 安装依赖软件

首先更新系统软件

```shell
sudo apt-get update
sudo apt-get upgrade
```

然后安装Nginx，并启动

```shell
sudo apt-get install -y nginx
sudo service nginx start
```

之后在Windows浏览器输入`http:localhost`就可以看到Nginx的默认页面

![安装成功](https://cdn.jsdelivr.net/gh/Johnny-Yh/ImageHosting/img/211208-1.png)

然后安装OpenSSL依赖

```shell
sudo apt-get install -y openssl libssl-dev
```

## 创建CA（Certificate authority）证书

下面的命令将会创建RootCA.pem、RootCA.key&RootCA.crt

```shell
openssl req -x509 -nodes -new -sha256 -days 1024 -newkey rsa:2048 -keyout RootCA.key -out RootCA.pem -subj "/C=US/CN=Root-CA"
openssl x509 -outform pem -in RootCA.pem -out RootCA.crt
```

## 创建域名（Localhost）证书

首先创建一个带有所有本地域名的文件domains.ext，这里我们只需要创建localhost的证书，所以只有一行，如果需要添加其他本地域名，可以在后面添加 `DNS.2=domain.local`，依此类推。

```shell
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = localhost
```

然后生成域名证书。其中C、ST、L、O分别代表国家、省份、城市和名称，后面的参数可以改成自己的。

```shell
openssl req -new -nodes -newkey rsa:2048 -keyout localhost.key -out localhost.csr -subj "/C=US/ST=YourState/L=YourCity/O=Example-Certificates/CN=localhost.local"
openssl x509 -req -sha256 -days 1024 -in localhost.csr -CA RootCA.pem -CAkey RootCA.key -CAcreateserial -extfile domains.ext -out localhost.crt
```

## 在Nginx中配置HTTPS

上面的证书我都放在了`/etc/ssl/`下，直接修改配置文件`/etc/nginx/sites-available/default`即可。

首先备份一下default文件，再进行修改

```shell
cd /etc/nginx/sites-available/
mv default default.bak
vim default
```

其实就是添加了下面两行

`ssl_certificate /etc/ssl/localhost.crt;`
`ssl_certificate_key /etc/ssl/localhost.key;`

修改后的配置文件如下

```nginx
server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;

        ssl_certificate /etc/ssl/localhost.crt;
        ssl_certificate_key /etc/ssl/localhost.key;

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

}
```

保存之后测试一下Nginx配置是否正确，再启动Nginx

```shell
nginx -t
service nginx start
```

## 在浏览器中信任本地CA

我在Windows中使用的是Edge浏览器，需要在Edge的设置->隐私、搜索和服务->安全性->管理证书中，将前文创建的`RootCA.crt`文件导入受信任的根证书颁发机构中，成功导入之后就可以看到自己的证书啦。

![成功导入](https://cdn.jsdelivr.net/gh/Johnny-Yh/ImageHosting/img/211208-2.png)

## 测试HTTPS

打开Edge浏览器，输入`https://localhost`，发现已经成功开启HTTPS访问，并且域名左边有一把🔒，说明证书是有效的，至此实验完成。

![](https://cdn.jsdelivr.net/gh/Johnny-Yh/ImageHosting/img/211208-3.png)



### 参考链接：

- [How to create an HTTPS certificate for localhost domains (github.com)](https://gist.github.com/cecilemuller/9492b848eb8fe46d462abeb26656c4f8)
- [How To Create a Self-Signed SSL Cert for Nginx in Ubuntu 18.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-18-04)