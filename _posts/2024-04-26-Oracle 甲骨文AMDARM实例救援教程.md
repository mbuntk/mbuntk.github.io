---
layout:     post
title:      Oracle 甲骨文AMD/ARM实例救援教程
subtitle:   Oracle 甲骨文AMD/ARM实例救援教程
date:       2024-04-26
author:     sundys
header-img: img/bg-linux.jpg
catalog: true
tags:
    - Linux
    - 甲骨文	
---

由于甲骨文不提供系统重置，对于爱折腾的龟友，如果小龟折腾坏了要么就是自己配置VCN救援，要么就只能删机重开。但甲骨文很多区域的ARM甚至是AMD都资源紧缺，可能删除后就很难开出来了。VCN救援对于不少龟友可能又有些难度。那有没有更简单的方法呢？下面我们就以AMD救援ARM为例详细说一下如何救砖。

该方法也适用于ARM救援AMD，ARM救援ARM，AMD救援AMD。

> **2024年1月更新**：
> 
> 更新官方原版Ubuntu 20.04系统 AMD / ARM恢复包，启用密码&秘钥双登录模式。
> 
> 实际执行恢复过程中，出现了一些概率，在恢复后root密码变为执行恢复命令的VPS的root密码。原因暂时不明。所以如果SSH登录时root密码出现错误，请使用你执行恢复命令的那台VPS的root密码进行尝试。或直接下载并在SSH工具中载入私钥，使用秘钥无密码登录。

1、首先将失联的ARM进行关机：甲骨文后台=>计算>>实例，选择ARM实例进入实例详细信息页面。点击**停止**，弹出框内勾选(Force stop the instance by immediately powering off)，然后**确定**，等待停止成功。

![](/img/or-bg01.png)

2、分离ARM引导卷：在ARM实例详细信息页面，下拉到下方列表\[左下角\]，选择**引导卷**，点击引导卷列表右边那三点图标，选择**分离**，然后**确定**。

![](/img/or-bg02.png)

3、新建一台免费AMD。在新建的AMD实例详细信息页面，下拉到下方列表\[左下角\]，选择**附加的块储存卷**，点击**附加块储存卷**，在弹出页面的选择卷下的方框里选择刚刚那个分离出来引导卷，挂载方式选择半虚拟化，然后**确定**。

![](/img/or-bg03.png)

![](/img/or-bg04.png)

4、SSH连接刚刚新建的AMD实例。使用 `lsblk` 或 `fdisk -l` 命令，你就可以查看到附加的ARM引导卷，一般是 `/dev/sdb` （具体盘符请自行查看）。

5、接下来就下载DD救援包。如果自己没有备份救援包的话，这里提供2个ARM救援包和1个AMD救援包供大家使用（大家也可以自行备份自己手上现有的AMD或ARM机型，制作救援包。方法见文末）：

**Ubuntu 20.04 ARM 官方原版完整救援包**（恢复数据约46G，耗时约1个多小时）

```
<span>wget </span><span>--</span><span>no</span><span>-</span><span>check</span><span>-</span><span>certificate https</span><span>:</span><span>//github.com/honorcnboy/BlogDatas/releases/download/OracleRescueKit/Ubuntu20.04.arm.img.gz</span>
```

用户名:`root`, 密码:`CNBoy.org`。或下载下方秘钥，载入SSH工具，无密码登录

```
<span>https</span><span>:</span><span>//github.com/honorcnboy/BlogDatas/releases/download/OracleRescueKit/backup</span>
```

**Ubuntu 20.04 AMD 官方原版完整救援包**（恢复数据约46G，耗时约1个多小时）

```
<span>wget </span><span>--</span><span>no</span><span>-</span><span>check</span><span>-</span><span>certificate https</span><span>:</span><span>//github.com/honorcnboy/BlogDatas/releases/download/OracleRescueKit/Ubuntu20.04.amd.img.gz</span>
```

用户名:`root`, 密码:`CNBoy.org`。或下载下方秘钥，载入SSH工具，无密码登录

```
<span>https</span><span>:</span><span>//github.com/honorcnboy/BlogDatas/releases/download/OracleRescueKit/backup</span>
```

**debian10 ARM 网络精简救援包**（恢复数据约3G，耗时约10多分钟）

```
<span>wget </span><span>--</span><span>no</span><span>-</span><span>check</span><span>-</span><span>certificate https</span><span>:</span><span>//github.com/honorcnboy/BlogDatas/releases/download/OracleRescueKit/dabian10.arm.img.gz</span>
```

用户名：`root`, 密码：`10086.fit`

6、恢复镜像到 `/dev/sdb` 分区（如果你的引导卷加载路径不同，请自行修改路径）

```
<span>说明：为了防止你在</span><span>SSH</span><span>连接在恢复数据中途中断导致失败，建议使用</span><span> srceen </span><span>后台窗口运行以下命令</span><span>

</span><code><span>gzip </span><span>-</span><span>dc </span><span>'救援包完整路径'</span><span> </span><span>|</span><span> dd of</span><span>=</span><span>'引导卷加载路径'</span></code><span>

</span><span>例：</span><span>
</span><span>使用</span><span> </span><span>Ubuntu</span><span> </span><span>20.04</span><span> ARM </span><span>官方原版完整救援包，命令如下：</span><span>
</span><code><span>gzip </span><span>-</span><span>dc </span><span>/</span><span>root</span><span>/</span><span>Ubuntu20</span><span>.</span><span>04.arm</span><span>.</span><span>img</span><span>.</span><span>gz </span><span>|</span><span> dd of</span><span>=</span><span>/dev/</span><span>sdb</span></code><span>

</span><span>使用</span><span> </span><span>Ubuntu</span><span> </span><span>20.04</span><span> AMD </span><span>官方原版完整救援包，命令如下：</span><span>
</span><code><span>gzip </span><span>-</span><span>dc </span><span>/</span><span>root</span><span>/</span><span>Ubuntu20</span><span>.</span><span>04.amd</span><span>.</span><span>img</span><span>.</span><span>gz </span><span>|</span><span> dd of</span><span>=</span><span>/dev/</span><span>sdb</span></code><span>

</span><span>使用</span><span> debian10 ARM </span><span>网络精简救援包，命令如下：</span><span>
</span><code><span>gzip </span><span>-</span><span>dc </span><span>/</span><span>root</span><span>/</span><span>dabian10</span><span>.</span><span>arm</span><span>.</span><span>img</span><span>.</span><span>gz </span><span>|</span><span> dd of</span><span>=</span><span>/dev/</span><span>sdb</span></code>
```

7、恢复过程中你可以新开一个SSH窗口，然后运行以下命令后不要关闭，切换回恢复命令的窗口查看进度

```
<span>watch </span><span>-</span><span>n </span><span>5</span><span> pkill </span><span>-</span><span>USR1 </span><span>^</span><span>dd$</span>
```

![](/img/or-bg05.png)

8、等待镜像恢复完成后，到甲骨文后台，新建的AMD中卸载掉刚刚附加的块储存卷，并至ARM实例中挂载回这个引导卷。然后启动ARM实例。SSH连接ARM，你就会发现你的ARM又复活了！

接下来修改为自己的密码就可以开始使用了。

另外，现在不只是ARM，有些地区的AMD资源也出现了紧缺的情况，所以一旦玩儿坏删机就很难开出来了。博主建议大家都自行备份制作一份 AMD / ARM 救援包。

备份步骤前4步与上面一样，挂载好引导卷之后，SSH执行已下命令进行备份制作。待完成后，将 `own.img.gz` 下载下来自己备存即可。
```
<span>dd </span><span>if</span><span>=</span><span>/dev/</span><span>sdb </span><span>|</span><span> gzip </span><span>&gt;</span><span> </span><span>/root/</span><span>own</span><span>.</span><span>img</span><span>.</span><span>gz</span>
```
