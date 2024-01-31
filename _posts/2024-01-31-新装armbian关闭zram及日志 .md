---
layout:     post
title:      新装armbian关闭zram及日志 
subtitle:   新装armbian关闭zram及日志 
date:       2024-01-31
author:     BY
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - N1盒子
---

一、首先将var/log等目录导入内存盘

   参考此前文章 的第三条 树莓派安装后常用设置及优化

二、删除zram的swap

  1.查看现有的swap,使用命令
  
    cat /proc/swaps或者swapon -s
	
  2.然后禁用当前swapswapoff /dev/zram1

  3.禁用zram服务,修改文件/etc/default/armbian-zram-config，将第一行的启用ENABLED=true改为ENABLED=false
  
  4.禁用zram的/var/log,修改/etc/default/armbian-ramlog，将第一行的启用ENABLED=true改为ENABLED=false
  
  5.禁用定时截断任务/etc/cron.d/armbian-truncate-logs,定时任务前加#注释
  
  6.禁用另一个任务，修改文件/etc/cron.daily/armbian-ram-logging如下，同样是加井号注释
        /bin/sh
		
       # /usr/lib/armbian/armbian-ramlog write >/dev/null 2>&1
	   
  
重启即可，/var/log.hdd/为空了


