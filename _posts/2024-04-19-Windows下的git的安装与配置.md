---
layout:     post
title:      Windows下的git的安装与配置 
subtitle:   Windows下的git的安装与配置 
date:       2024-04-19
author:     sundys
header-img: img/git.png
catalog: true
tags:
    - git配置
---

Windows下的git的安装与配置

下载git for Windows 到这里下载 https://git-scm.com/download/win git-for-window的安装包 安装的时候，一路下一步使用默认设置，就可以了。

安装成功后，在任意地方右键，应该会看到这样子：

![image](https://github.com/sundys/sundys.github.io/blob/master/img/git01.png)

3.打开https://github.com/ 网页注册一个属于自己github账号，教程例子：昵称test，邮箱test@qq.com

配置git 1.设置自己的昵称与email

选择任一文件夹鼠标右键git bash here：后输入:

git config --global user.name “yourname”

git config --global user.email “youremail@qq.com” 

有--global则表示全局配置

在某个项目根路径下面设置单独的Email与姓名：

git config user.name “yourname”

git config user.email “youremail@qq.com”

2.生成配置ssh密钥（在git bash命令行下）

查看是否已有密钥ls -al ~/.ssh存在则可忽略，不存在则可按下面的方式生成

![image](https://github.com/sundys/sundys.github.io/blob/master/img/git02.png)

首先确保系统盘c盘下的用户目录你正在使用的电脑用户目录内有.ssh文件夹，没有则可在此鼠标右键git bash here输入：mkdir .ssh

生成密钥：ssh-keygen -t rsa -C “test@qq.com” (邮箱请输入自己注册github时使用的邮箱)，回车，会提示你输入passphrase（没有输入则默认为没有密码，届时任何人都可在你创建的公开的仓库上push东西，所以最好需要自己简单易记的密码）。打开.ssh目录查看新生成的ssh key，会新增两个文件：id_rsa(私玥)和id_rsa_pub（公钥）。

添加ssh key：ssh-add ~/.ssh/id_rsa，如果报错：Could not open a connection to your authentication agent. 则尝试以下指令:

eval ssh-agent -s

ssh-add 打开.ssh文件夹下的id_rsa_pub文件复制，或直接使用命令clip < ~/.ssh/id_rsa_pub

打开github网页登录刚刚注册的github账号，登录后点击头像-settings

![image](https://github.com/sundys/sundys.github.io/blob/master/img/git03.png)

选择ssh and GPG后点击 keys New SSH key

![image](https://github.com/sundys/sundys.github.io/blob/master/img/git04.png)
![image](https://github.com/sundys/sundys.github.io/blob/master/img/git05.png)

title随便填，然后把刚刚复制的id_rsa_pub的内容复制到Key框下，点击Add SSH key即可

![image](https://github.com/sundys/sundys.github.io/blob/master/img/git06.png)

输入你的Github密码即可完成SSH Key的添加

3.测试是否配置ssh key成功，在git bash命令行下输入ssh -T git@github.com 如果刚刚在生成ssh密钥时有输入密码此处会要求输入密码，输入正确才会连接成功。另外在以后的项目push中也会要求输入该密码，所以请勿遗忘。

![image](https://github.com/sundys/sundys.github.io/blob/master/img/git07.png)

至此，基本的git配置已完成。
