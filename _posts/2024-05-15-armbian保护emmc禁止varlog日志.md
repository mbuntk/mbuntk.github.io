---
layout:     post
title:      armbian保护emmc禁止varlog日志
subtitle:   armbian保护emmc禁止varlog日志,docker容器限制日志大小 
date:       2024-05-15
author:     sundys
header-img: img/docker.png
catalog: true
tags:
    - Docker
---

因为emmc存储是一种flash存储技术，其写入寿命非常有限，所以系统运行中应尽量避免数据写入。

如果我们没有装什么特殊程序的话，通常来说数据的主要写入就是/var/log目录的日志了，一天几十MB还是有的。

armbian其实已经考虑了这个问题，因为armbian就是给arm架构订制的debian发行版嘛，所以它默认是创建了一个内存盘（zram文件系统）挂载到了/var/log目录：
```
root@aml:/var/log# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            469M     0  469M   0% /dev
tmpfs           184M   22M  163M  12% /run
/dev/mmcblk1p2  6.4G  2.1G  4.3G  33% /
tmpfs           920M     0  920M   0% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           920M     0  920M   0% /sys/fs/cgroup
tmpfs           920M  8.0K  920M   1% /tmp
/dev/mmcblk1p1  122M   58M   64M  48% /boot
/dev/zram0       49M   15M   31M  32% /var/log
tmpfs           184M     0  184M   0% /run/user/0
```
所以频繁的日志写入并不会直接伤害到emmc。

但是这块zram盘只有49MB，基本上1~2天就会写满，所以armbian是如何处理的呢？

经过我的研究，发现系统做了1个systemd启动任务+2个cron任务用来解决这个问题，下面简单说一下原理。

## 详细分析

当然是定期删除日志了，难不成还有魔法嘛。
```
root@aml:/var/log# cat /etc/cron.d/armbian-truncate-logs
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
 
*/15 * * * * root /usr/lib/armbian/armbian-truncate-logs
```
每15分钟就会执行一次truncate日志，这个脚本内容如下：

```
treshold=75 # %
 
[ -f /etc/default/armbian-ramlog ] && . /etc/default/armbian-ramlog
 
[ "$ENABLED" != true ] && exit 0
 
logusage=$(df /var/log/ --output=pcent | tail -1 |cut -d "%" -f 1)
if [ $logusage -ge $treshold ]; then
		# write to SD
                /usr/lib/armbian/armbian-ramlog write >/dev/null 2>&1
                # rotate logs on "disk"
                chown root.root -R /var/log.hdd
                /usr/sbin/logrotate --force /etc/logrotate.conf
		# truncate
	        /usr/bin/find /var/log -name '*.log' -or -name '*.xz' -or -name 'lastlog' -or -name 'messages' -or -name 'debug' -or -name 'syslog' | xargs truncate --size 0
		/usr/bin/find /var/log -name 'btmp' -or -name 'wtmp' -or -name 'faillog' | xargs truncate --size 0
		# remove
		/usr/bin/find /var/log -name '*.[0-9]' -or -name '*.gz' | xargs rm >/dev/null 2>&1
 
fi
```
其实就是看一下/var/log的zram盘是否利用率超过75%，一旦超过就扫描/var/log下面各种日志文件进行截断。

另外，我们还看到它调用：

> /usr/lib/armbian/armbian-ramlog write >/dev/null 2>&1

这个脚本的write命令会把/var/log内存盘的数据rsync到/var/log.hdd/目录，而/var/log.hdd目录是emmc上的一部分。因此，我们明白在truncate日志之前会先把当前最新日志持久化到emmc上，然后再把zram内存里的日志截断掉。

另外，这个脚本还调用了logrotate程序进行日志滚动，我详细看了一下logrotate配置文件，发现它归档的是/var/log.hdd里面的日志文件，其根本目的是为了配合zram -> emmc做rsync的时候可以结合rsync –delete选项删除掉归档的老日志文件，起到控制emmc容量的目的。**（大家不理解可以不关心这一段逻辑）**

**所以呢，这个cron会导致每15分钟就会向emmc同步一次数据，并且缩小zram盘占用容量，这无疑是对emmc的频繁伤害。**

另外还有一个天级cron是进行一次write同步，也是调用的如下同步命令：

> /usr/lib/armbian/armbian-ramlog write >/dev/null 2>&1

因此，最简单的就是让这个write操作失灵，不向emmc同步日志数据不就好了嘛。

## 解决方法

打开/usr/lib/armbian/armbian-ramlog脚本，它实际执行的是这个shell方法：
```
syncToDisk () {
	isSafe
 
	echo -e "\n\n$(date): Syncing logs from $LOG_TYPE to storage\n" | $LOG_OUTPUT
 
	if [ "$USE_RSYNC" = true ]; then
		${NoCache} rsync -aXWv --delete --exclude armbian-ramlog.log --links $RAM_LOG $HDD_LOG 2>&1 | $LOG_OUTPUT
	else
		${NoCache} cp -rfup $RAM_LOG -T $HDD_LOG 2>&1 | $LOG_OUTPUT
	fi
 
	sync
}
```
只需要在函数头部返回即可避免rsync：
```
syncToDisk () {
	# no sync to protect emmc
	return 0
	isSafe
 
	echo -e "\n\n$(date): Syncing logs from $LOG_TYPE to storage\n" | $LOG_OUTPUT
 
	if [ "$USE_RSYNC" = true ]; then
		${NoCache} rsync -aXWv --delete --exclude armbian-ramlog.log --links $RAM_LOG $HDD_LOG 2>&1 | $LOG_OUTPUT
	else
		${NoCache} cp -rfup $RAM_LOG -T $HDD_LOG 2>&1 | $LOG_OUTPUT
	fi
 
	sync
}  
```

可以再观察一下/var/log与/var/log.hdd，会发现/var/log.hdd已经不再有后续数据更新，而/var/log仍旧会自动在75使用率的时候进行日志截断。

> 最后补充，armbian做了一个systemd服务：/lib/systemd/system/armbian-ramlog.service，它开机会创建zram盘，然后从emmc的/var/log.hdd中load数据到zram的/var/log路径下，完成开机初始化。

### docker限制日志文件大小

一、docker查看日志文件的方法, 除了

```  
docker logs -f 容器ID/容器名
```
这个方法以外。

在linux上，一般docker的日志文件存储在/var/lib/docker/containers/container\_id/ 目录下的 各个容器ID对应的目录下的\*-json.log 文件中

方法1：可以直接进入该目录下，查找日志文件

方法2：可以写一个脚本文件，执行即可

　　1》创建.sh文件【在你自己可以找到的目录下】
```  
vi docker_log_size.sh  
```  
文件内容

```
#!/bin/sh 
echo "======== docker containers logs file size ========"  

logs=$(find /var/lib/docker/containers/ -name *-json.log)  

for log in $logs  
        do  
             ls -lh $log   
        done
```

　　2》为该文件设置权限

```
chmod +x docker_log_size.sh
```

　　3》执行该文件
```
./docker_log_size.sh
```

二.设置Docker容器日志文件大小限制

1.新建/etc/docker/daemon.json，若有就不用新建了。添加log-dirver和log-opts参数，样例如下：

```
# vim /etc/docker/daemon.json

{
  "log-driver":"json-file",
  "log-opts": {"max-size":"1m", "max-file":"1"}
}
```

max-size=500m，意味着一个容器日志大小上限是500M，   
max-file=3，意味着一个容器有三个日志，分别是id+.json、id+1.json、id+2.json。

2.然后重启docker的守护线程

命令如下： 
```
systemctl daemon-reload  
```
```
systemctl restart docker
```
【需要注意的是：设置的日志大小规则，只对新建的容器有效】

