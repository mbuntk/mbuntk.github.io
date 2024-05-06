---
layout:     post
title:      Debian 挂载 WebDAV,实现本机备份到网盘
subtitle:   利用rsync备份到Debian 挂载 WebDAV的网盘中
date:       2024-04-30
author:     sundys
header-img: img/ns-sj.png
catalog: true
tags:
    - WebDAV
    - Docker
    - N1盒子
---


## 一、Debian 挂载 WebDAV

1\. 安装 davfs2 软件包

```bash
apt install davfs2
```
2\. 创建一个用来挂载 WebDAV 的目录

```bash
mkdir /mnt/alist
```
3\. 编辑 davfs2.conf 配置文件，将 use\_locks 的 1 改为 0

```bash
sed -i 's/# use_locks 1/use_locks 0/g' /etc/davfs2/davfs2.conf
或手动修改
nano /etc/davfs2/davfs2.conf
```
4\. 保存用户名密码，以后可以直接免密码挂载

```bash
echo "你的WebDAV地址 用户名 密码" >> /etc/davfs2/secrets  
或手动添加
nano /etc/davfs2/secrets
```

5\. 开机自动挂载

方案一：

编辑 /etc/fstab 文件

> /etc/fstab 是 Linux 系统中的一个配置文件，用于定义文件系统的挂载点和相关的选项。它的主要作用是在系统启动时自动挂载文件系统。fstab 的全名是 "file systems table"，它记录了系统上所有可用的文件系统和它们的挂载配置。
```bash
echo "你的WebDAV地址 /mnt/alist davfs rw,user,file_mode=0600,dir_mode=0700,_netdev 0 0" >> /etc/fstab
```
重启即可自动挂载,注意此方法有一定风险,挂载不成功会导致无法进入系统.推荐方案二.

方案二：

编辑 crontab 文件
```bash
crontab -e
```
添加一个开机执行的任务
```bash
@reboot sleep 180 && mount -t davfs 你的WebDAV地址 /mnt/alist -o rw,user,file_mode=0600,dir_mode=0700,_netdev
```
注意定时挂载时间,如果是本机docker运行alist要设置时间长些,等待alist启动成功,重启系统后查看是否挂载成功：
```bash
df -h
```
![df - h 截图](/img/de-av.png)


## 二、定时备份数据到 WebDAV

安装 rsync
```bash
apt install rsync 
```
编辑 crontab 文件
```bash
crontab -e
```
添加一个定时执行的任务,每天凌晨四点备份 /www/data 到 WebDAV 的 data-backup 文件夹
```bash
0 4 * * * rsync -av --delete /www/data /mnt/alist/data-backup
```
每天凌晨四点备份指定文件到 WebDAV挂载的网盘中的文件夹
```bash
0 4 * * *  rsync -av --remove-source-files /root/ql_backup.tar.gz /mnt/alist/天翼云盘/qlbk/
```
结合使用 rsync 定时自动备份数据食用再也不用担心丢数据,rsync 真是太好用辣！
