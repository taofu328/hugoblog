---
title: "LINUX下使用WoeUSB制作 WINDOWS启动U盘"
date: 2020-03-26T23:48:26+08:00
draft: false
tags: [
    "windows",
]
---
前不久把自己电脑换成深度操作系统体验了一段时间，想换回win10的时候发现要做启动盘还挺麻烦。网上不少方法都是要用分区工具分区，创建分区表，设置boot标示，等等等等，复杂得都没耐心看完。后来发现有一款叫[WoeUSB](https://github.com/slacka/WoeUSB)的开源工具，使用起来特别方便。WoeUSB是一个创建Windows可引导U盘工具，支持从 ISO 镜像文件及 DVD 来创建，同时提供图形界面及命令行界面。<br />**Deepin系统从商店安装WoeUSB**<br />   深度系统可以从应用商店中直接安装WoeUSB，版本号是3.3.1，但是看截图图形界面不支持选择NTFS格式。现在win10的install.wim大小超过4G，所以如果用这个版本的话，可以用命令行还制作U盘
```bash
# /dev/sdb 换成自己u盘，不用带分区数字，可以使用 sudo fdisk -l 查看到
sudo woeusb --device /path/to/filename.iso /dev/sdb --target-filesystem NTFS
```
**从源码安装WoeUSB**

1. 下载源码。从git上克隆WoeUSB到本地目录 `git clone https://github.com/slacka/WoeUSB.git`
1.  生成应用版本号
```bash
$ ./setup-development-environment.bash
```

3. 安装编译依赖项。以下操作适用于基于Debian的Linux发行版
```bash
$ sudo apt-get install devscripts equivs gdebi-core
$ mk-build-deps
$ sudo gdebi woeusb-build-deps_<version>_all.deb #version是第二步的版本号
```

4. 安装 WoeUSB
```bash
$ dpkg-buildpackage -uc -b
$ sudo gdebi ../woeusb_<version>_<architecture>.deb #deb文件在WoeUSB源码的上一层目录
```

5. 制作启动U盘可以用前面的命令来制作，用图形界面跟方便。WoeUSB运行后的界面如下

![WoeUSB.png](https://cdn.nlark.com/yuque/0/2020/png/846429/1585237233708-6df18a09-3868-4d62-a165-99a731f6d5c2.png#align=left&display=inline&height=540&name=WoeUSB.png&originHeight=540&originWidth=400&size=21329&status=done&style=none&width=400)<br />选择Windows的iso文件，File System选择NTFS,然后点install开始制作启动U盘
