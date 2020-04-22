---
title: "Centos8安装Docker CE"
date: 2020-02-20T01:53:55+08:00
draft: false
tags: [
    "centos",
    "docker",
]
---

## 添加软件源
为了方便添加软件源，支持 devicemapper 存储类型，安装如下软件包
```shell
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
 添加 Docker 稳定版本的 yum 软件源
```shell
$ sudo yum-config-manager --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
## 安装Docker CE
官方给出的安装命令是
```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```
 运行后报错：
```shell
错误：
 问题: package docker-ce-3:19.03.6-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
  - cannot install the best candidate for the job
  - package containerd.io-1.2.10-3.2.el7.x86_64 is excluded
  - package containerd.io-1.2.2-3.3.el7.x86_64 is excluded
  - package containerd.io-1.2.2-3.el7.x86_64 is excluded
  - package containerd.io-1.2.4-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.5-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.6-3.3.el7.x86_64 is excluded
(尝试添加 '--skip-broken' 来跳过无法安装的软件包 或 '--nobest' 来不只使用最佳选择的软件包)
```
 解决办法是下载rpm包来安装,Docker提供的下载地址是[这里](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)。
安装containerd.io-1.2.6-3.3.el7.x86_64.rpm时报错:
```shell
$ rpm -ivh containerd.io-1.2.6-3.3.el7.x86_64.rpm 
错误：依赖检测失败：
	runc 与 containerd.io-1.2.6-3.3.el7.x86_64 冲突
	runc 被 containerd.io-1.2.6-3.3.el7.x86_64 取代
```
删除runc:
```shell
$ yum remove runc
```
再次安装containerd.io-1.2.6-3.3.el7.x86_64.rpm继续报错:
```shell
错误：依赖检测失败：
	container-selinux >= 2:2.74 被 containerd.io-1.2.6-3.3.el7.x86_64 需要
```
安装container-selinux:
```shell
$ sudo yum install container-selinux
```
再次安装containerd.io-1.2.6-3.3.el7.x86_64.rpm,成功。
最后通过yum命令安装Docker成功。

```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```


