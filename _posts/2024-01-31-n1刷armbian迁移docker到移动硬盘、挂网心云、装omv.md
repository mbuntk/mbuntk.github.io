---
layout:     post
title:      n1刷armbian迁移docker
subtitle:   n1刷armbian迁移docker到移动硬盘、挂网心云、装omv
date:       2024-01-31
author:     BY
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - N1盒子
	- Armbian
	- Docker
---

1.刷armbian、安装docker,这个有很多教程了，不再写了。

2.移动硬盘分区、挂载:

(1) 首先查看新硬盘基本信息

     fdisk -l

--会罗列出很多分区地址,自行确认自己的新硬盘识别地址在哪里，我的新硬盘被识别到了”/dev/sda”。

(2) 对新硬盘进行分区

fdisk /dev/sda

--[1] 这里的 /dev/sda 是步骤（1）中 查询出来的硬盘识别文件地址。如果你在步骤（1）中要格式化的硬盘存在于其他地址请相应改变。

--[2] 在提示信息引导下，我选择（n） “add a new partition” 将硬盘划分为一个新分区。

(p) primary ----主分区（看个人选择）

(e) extended----扩展分区（看个人选择）

--[3] 若整个硬盘只作为一个分区，下面三步默认回车即可；若只拿一部分空间出来当分区详细如下：

   [3-1] 第一步是分区盘号，默认回车自动分配盘号，可自己定义一下盘号例如输入4，则盘号为sda4。
   
   [3-2] star-是从2048字节开始，开始大小建议默认2048（默认回车即可）
   
  [3-3] end-输入结束字节，开始字节到结束字节为新建分区盘的大小，输入后回车即可，直接回车则默认输入最大字节。
  
--[4] 最后再输出（p）确认下自己创建的分区表信息是否正确。确认无误后（w）保存。

--[5] 如果成功，系统会提示“The partition table has been altered” 分区表已更改完毕 。

(3) 查看新硬盘识别到了哪里

    重新输入(1) 内容 ，我本地的新硬盘分了两个区被识别到了 “/dev/sda1”、“/dev/sda2” 。
	
(4) 新硬盘格式化

        mkfs -t ext4 /dev/sda1

2.迁移docker到硬盘

（1）挂载硬盘

    mkdir /mnt/USB              # 创建目录供挂载使用

    mount -v /dev/sda1 /mnt/USB  # 挂载 U 盘

    df -h                       # 查看挂载状态

（2）停止docker

       service docker stop


（3）迁移docker数据

# 创建目录

mkdir /mnt/USB/docker -p

# 拷贝数据
# -rpvb 递归/保留属性/覆盖/详细

cp /var/lib/docker/* /mnt/USB/docker -rpvb

mv /var/lib/docker /var/lib/docker.bak

# 软连接：实际 + 目标

ln -s /mnt/USB/docker /var/lib

# 恢复步骤，删除软连接（警告！尾部没有左斜杠 /）

     #rm -rf /var/lib/docker

# 生效/启动

      systemctl daemon-reload
	  service docker restart

# 验证

     docker info
	 
#docker info | grep 'docker Root Dir'

-----------------

显示=成功

Docker Root Dir: /mnt/USB/docker

# 重启自动挂载 U 盘，在 rc.local

sed -i '/exit 0/i\mount -v /dev/sda /mnt/USB' /etc/rc.local


（4）docker延迟启动

防止硬盘挂载慢，dockers启动后挂载失败

# 移除docker自启服务

systemclt disable docker

编辑/etc/rc.local文件，文件末尾exit0之前追加如下内容并保存：

sleep 60

systemctl start docker

以上就是N1盒子用U盘扩容的全部教程。

