---
layout:     post
title:      N1 通过 Docker 安装 Openwrt 为旁路由
subtitle:   N1 通过 Docker 安装 Openwrt 为旁路由
date:       2024-04-22
author:     sundys
header-img: img/ns-sj.png
catalog: true
tags:
    - N1盒子
    - Docker
	- Openwrt
---

## 说明

我的网络情况:

1.  主路由 IP 是 `192.168.4.1`;
2.  N1 Armbian IP 是 `192.168.4.2`;
3.  Openwrt IP 是 `192.168.4.11`.

## 1\. 开启网卡混杂模式

```
## 启用
ip link set eth0 promisc on
```

## 2\. docker 创建 macvlan 网络

```
docker network create -d macvlan --subnet=192.168.4.0/24 --gateway=192.168.4.1 -o parent=eth0 macnet
```

## 3\. 启动 openwrt 镜像

```
# for N1
docker run \
  -d \
  --name=unifreq-openwrt-aarch64 \
  --restart=unless-stopped \
  --network=macnet \
  --privileged \
  --ip=192.168.4.11 \
  unifreq/openwrt-aarch64:r21.11.11
```

## 4\. 修改容器 IP 信息并重启容器

```
# 见 Docker 镜像说明: https://hub.docker.com/r/unifreq/openwrt-aarch64
docker exec unifreq-openwrt-aarch64 sed -e "s/192.168.1.1/192.168.4.11/" -i /etc/config/network
docker restart unifreq-openwrt-aarch64
```

## 5\. http://192.168.4.11 访问 openwrt

默认账号密码: `root/password`

## 6\. 关闭 DHCP

路径: 网络 => 接口 => LAN => DHCP 服务器 => 基本设置 操作: 勾选忽略此接口 
![image](/img/ns-dk-01.png)

## 7\. 关闭 IPv6

路径: 网络 => 接口 => LAN => DHCP 服务器 => IPv6 设置 操作: 禁用 `路由通告服务`, `DHCPv6 服务`, `NDP 代理` 
![image](/img/ns-dk-02.png)

## 8\. 配置 网关 和 DNS

路径: 网络 => 接口 => LAN => 一般设置 => 基本设置 操作: 1. `IPv4 网关` 改为 `192.168.4.1`; 2. `IPv4 广播` 改为 `192.168.1.255`; 3. `使用自定义的 DNS 服务器` 改为 `114.114.114.114` 
![image](/img/ns-dk-03.png)

## 9\. 主路由系统是 `Padavan` 需要的配置

将 `IPv4 硬件加速` 改为 `Offload TCP/UDP for LAN`. 
![image](/img/ns-dk-04.png)

## 10\. 宿主机与Openwrt互通, 宿主机访问外网

目前宿主机无法访问 Openwrt 以及外网, 需要的配置: 在 /etc/rc.local 中增加初始化逻辑:

```
ip link add mynet link eth0 type macvlan mode bridge 
ip addr add 192.168.4.2 dev mynet
ip link set mynet up
ip route add 192.168.4.11 dev mynet
```

## 11\. 配置主路由

目前主路由下的设备可以在网络设置中将路由器的地址改为`192.168.4.11`即可实现通过旁路由上网. 为了避免各个设备分别配置, 达到全局使用旁路由的目的, 可以配置主路由: 将 DHCP 默认网关 和 DNS 都改为 `192.168.4.11` 
![image](/img/ns-dk-05.png)
