---
title: "Centos和Debian下编译安装nginx"
date: 2020-03-10T0:20:26+08:00
draft: false
tags: [
    "centos",
    "linux",
	"nginx",
]
---

<a name="IHIVQ"></a>
## 1.下载nginx
到[官网](https://nginx.org/en/download.html)下载nginx的源码并解压。
<a name="7DBfh"></a>
## 2.安装依赖模库
nginx的默认配置需要PCRE库和zlib库。PCRE库支持正则表达式，zlib库用于对HTTP包的内容做gzip格式的压缩。如果不需要可以在执行 `./configure` 通过参数 `--without-http_rewrite_module` 和 `--without-http_gzip_module` 来忽略。要使用https的话，还需要安装openssl。<br />Centos
```shell
yum -y install pcre-devel
yum -y install zlib-devel
yum -y install openssl openssl-devel
```
Debian
```shell
sudo apt-get install libpcre3 libpcre3-dev
sudo apt-get install zlib1g-dev
sudo apt-get install openssl libssl-dev
```
<a name="ar70I"></a>
## 3.安装
```shell
./configure --with-http_ssl_module
make 
make install
```
<a name="3I3TU"></a>
## 4.设置nginx为服务并开机自动启动
`nano /lib/systemd/system/nginx.service`,填入以下内容
```shell
[Unit]
Description=nginx
After=network.target
 
[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
```
更改文件权限 `chmod 745 /lib/systemd/system/nginx.service` <br />启动服务`systemctl start nginx.service`<br />设置开机自启动 `systemctl enable nginx.service` 
