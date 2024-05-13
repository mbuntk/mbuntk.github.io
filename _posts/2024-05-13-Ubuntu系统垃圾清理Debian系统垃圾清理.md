---
layout:     post
title:      Linux(甲骨文 GCP等)修改时区
subtitle:   Linux(甲骨文 GCP等)修改时区
date:       2024-05-13
author:     sundys
header-img: img/bg-linux.jpg
catalog: true
tags:
    - Linux
---


## **一、清理下载的软件包**

按快捷键 **Ctrl + Alt + T** 打开终端，输入命令：

```bash
du -h /var/cache/apt/archives
```

回车之后，就可以看到软件安装包所占用的空间。

#### 1、删除已卸载软件的安装包：

```bash
sudo apt-get autoclean
```

#### 2、删除所有的软件安装包：

```bash
sudo apt-get clean
```

#### 3、卸载孤立无用软件包：

```bash
sudo apt-get autoremove
```

___

## 二、删除不用的老旧内核

#### 1、查看当前使用的内核命令：

```bash
uname -r
```

#### 2、查看系统已安装过的内核：

```bash
$ dpkg --get-selections | grep linux

binutils-x86-64-linux-gnu install

console-setup-linux install

libselinux1:amd64 install

linux-base install

linux-firmware install

linux-generic install

linux-headers-4.15.0-74 install

linux-headers-4.15.0-74-generic install

linux-headers-generic install

linux-image-4.15.0-20-generic deinstall

linux-image-4.15.0-23-generic deinstall

linux-image-4.15.0-24-generic deinstall

linux-image-4.15.0-28-generic install

linux-image-4.15.0-74-generic install

linux-image-generic install

linux-libc-dev:amd64 install

linux-modules-4.15.0-20-generic deinstall

linux-modules-4.15.0-23-generic deinstall

linux-modules-4.15.0-24-generic deinstall

linux-modules-4.15.0-28-generic install

linux-modules-4.15.0-74-generic install

linux-modules-extra-4.15.0-20-generic deinstall

linux-modules-extra-4.15.0-23-generic deinstall

linux-modules-extra-4.15.0-24-generic deinstall

linux-modules-extra-4.15.0-28-generic install

linux-modules-extra-4.15.0-74-generic install

linux-signed-generic install

linux-sound-base install

pptp-linux install

syslinux install

syslinux-common install

syslinux-legacy install

util-linux install
```

**显示为“deinstall”的可以放心卸载掉。**

可以把不用的老旧内核文件image、头文件headers、模块modules卸载掉。

如果目前使用的内核非常不错，可以把之前的所有更新的内核都给卸载掉（建议留个上一版本或者稳定版本，以防万一！）。

#### 3、卸载内核文件命令：

```bash
sudo apt-get purge linux-image-4.15.0-20-generic linux-modules-4.15.0-20-generic linux-modules-extra-4.15.0-20-generic
```  
