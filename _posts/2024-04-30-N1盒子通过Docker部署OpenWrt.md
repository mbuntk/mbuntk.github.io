---
layout:     post
title:      N1盒子通过Docker部署OpenWrt
subtitle:   参与教程N1盒子通过Docker部署OpenWrt
date:       2024-04-30
author:     sundys
header-img: img/ns-sj.png
catalog: true
tags:
    - N1盒子
    - Docker
    - Openwrt
---

## N1盒子通过Docker部署OpenWrt

我的N1盒子刷了F大的Armbian系统，u盘启动，刷机过程不在冗述，百度一搜就有很多，使用镜像的名称是Armbian\_20.10\_Aml-s9xxx\_buster\_5.10.26-flippy-56+，刷完以后，需要把u盘插到电脑上，进入BOOT盘，把根目录里的u-boot-n1.bin，复制一份，改名u-boot.ext即可正常启动。  
下面开始通过Docker部署OpenWrt，安装Docker也不说了，网上一搜也到处都是。

##### 1、打开网卡混杂模式

```csharp
ssh登陆n1后使用如下命令：  
ip link set eth0 promisc on  
然后执行 ifconfig 看输出结果的eth0后的描述内是否包含PROMISC N1重启后这个设置会失效，所以最好把这条命令加到/etc/rc.local中
```

##### 2、创建 Docker 虚拟网络

```php
docker network create -d macvlan --subnet=192.168.25.0/24 --gateway=192.168.25.1 -o parent=eth0 macnet  
这一条命令需要根据N1所处的网络环境来做修改，可以使用ifconfig命令来查看N1 eth0 网卡获得的IP地址，如果N1获得的IP地址为192.168.2.154，那么说明N1处在192.168.2.x网段，相应的，命令中的192.168.25.0和192.168.25.1需要被替换成 192.168.2.0和192.168.2.1
```

##### 3、拉取镜像

```undefined
docker pull sulinggg/openwrt:armv8
```

##### 4、运行OpenWRT容器

```kotlin
docker run --restart always --name openwrt -d --network macnet --privileged sulinggg/openwrt:armv8 /sbin/init
```

##### 5、配置 OpenWRT 容器网络

```bash

进入容器命令行  
docker exec -it openwrt bash  


编辑网络配置文件  
nano /etc/config/network  
config interface 'lan'  
            option type 'bridge'  
			option ifname 'eth0'  
			option proto 'static'  
			option ipaddr '192.168.25.10'  
			option netmask '255.255.255.0'  
			option ip6assign '60'  
			option gateway '192.168.25.1'  
			option dns '192.168.25.1' 
重启网络  
/etc/init.d/network restart
```

##### 6、配置OpenWRT

访问192.168.25.10即可进入路由器设置

##### 7、N1连接docker openwrt

经过以上设置后，N1并不能连接docker openwrt，ping不通，需要进行如下操作：
nano /etc/network/interfaces #编辑网络配置文件 先把里面的iface eth0 inet dhcp注释掉 然后在最后添加如下代码：
```bash 
auto eth0 iface eth0 inet  
manual auto macvlan iface  
macvlan inet dhcp  
#iface macvlan inet static  
# address 192.168.25.205  
# netmask 255.255.255.0  
# gateway 192.168.25.1  
# dns-nameservers 192.168.25.1  
pre-up ip link add macvlan link eth0 type macvlan mode bridge  
post-down ip link del macvlan link eth0 type macvlan mode bridge  
```
上面注释掉的部分是其他教程说的设置固定ip的方法，但是我测试直接使用DHCP也是可以的 需要固定IP的话，可以参考 保存配置文件，重启N1，就可以ping通docker openwrt了
