---
layout: post
title: 安装Ubuntu 14.04 遇囧记事
description: 安装ubunt14时，老安装不上，原来是我的u盘有问题，人品差到极点。
keywords: linux
categories: life
---

08年买的电脑貌似已经被windows嫌弃了，为了让老电脑焕发出新的生命力，决定换成ubuntu系统。
安装的过程中真是波折不断，倒不是说安装很难，而是我的两个U盘想吃错了药似的，一点都不给面子。

#### 准备工作
 - 一个大于2G的U盘
 - Ubuntu14.04的镜像（根据cpu选择32位或者64位）
 - [UUI制作工具][1]

#### 安装出现的囧事
我的电脑竟然无法读取我的u盘内容，我原先一直以为是我自己制作过程发生了什么错误。然后我向同事借了一个U盘，同样用UUI制作，电脑可以读取（感谢同事给的U盘，用自己的两个U盘没有反馈），但是出现这样的错误（为什么我的U盘就不行呢？）：
>**Boot failure : No DEFAULT or UI configuration directive found!**

有信息就是好事，顺着这个信息很快找到了[解决方法][2]：
>我试了里面的几种方法，最后将U盘格式化为FAT后才成功安装。
>来自[Richad的答案](http://askubuntu.com/questions/329704/syslinux-no-default-or-ui-configuration-directive-found#412853)：
>**It seems for everyone it has a different solution, for me, the solution was to format my Flash drive with the FAT filesystem and make a boot drive with the Universal USB installer and voila, worked like a charm.**

#### 体验效果
先看我的机子配置（08年买的电脑，信息有限）：
![七牛云](http://joyo-pic-1.qiniudn.com/ubuntu14.04_system_info.png)

下面是电脑桌面：
![七牛云](http://joyo-pic-1.qiniudn.com/my_desktop.png)

感觉老电脑都比较适合安装linux系统，还是比较流畅的。值得一提的是3D效果，还是很让人惊艳的。

下面是compiz的配置图：
![七牛云](http://joyo-pic-1.qiniudn.com/compiz_setting.png)

[1]: http://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/
[2]: http://askubuntu.com/questions/329704/syslinux-no-default-or-ui-configuration-directive-found
