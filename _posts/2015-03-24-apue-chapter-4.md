---
layout: post
title: APUE_3E - Files and Directories
description: APUE第四章 - 文件和目录的简单摘要
keywords: linux
categories: programming
---

#### Overview

- 围绕stat函数, 详细介绍stat结构中的每一个成员, 可以使用`man 2 stat`查看详细信息. shell中的stat命令可以使用`man stat`查看
- 介绍unix文件系统结构 - 文件和目录在文件系统中是如何设计的?

#### 文件访问权限

- 当我们用名字打开任一类型的文件时, 对该名字中包含的每一个目录, 包括它可能隐含的当前工作目录都应该具有执行权限. 例如, 为了打开文件`/usr/inlcude/stdio.h`, 需要对目录`/`, `/usr`, `/usr/include`具有执行权限.
- 对一个文件的读权限决定了我们是否能够打开现有的文件进行读操作. 这与open函数的O_RDONLY和O_RDWR标志相关.
- 对一个文件的读权限决定了我们是否能够打开现有的文件进行读操作. 这与open函数的O_WRONLY和O_RDWR标志相关.
- 为了在open函数中对一个文件指定O_TRUNC标志, 必须对该目录具有写权限.
- **为了在一个目录中创建一个新文件, 必须对包含该文件的目录具有写权限和执行权限.**
- **为了删除一个现有文件, 必须对包含该文件的目录具有写权限和执行权限. 对该文件本身不需要有读写权限.**

进程每次打开, 创建或者删除一个文件时, 内核就如下顺序进行文件访问权限测试:

>1. If the effective user ID of the process is 0 (the superuser), access is allowed. This
gives the superuser free rein throughout the entire file system.
2. If the effective user ID of the process equals the owner ID of the file (i.e., the
process owns the file), access is allowed if the appropriate user access
permission bit is set. Otherwise, permission is denied. By appropriate access
permission bit, we mean that if the process is opening the file for reading, the
user-read bit must be on. If the process is opening the file for writing, the
user-write bit must be on. If the process is executing the file, the user-execute bit
must be on.
3. If the effective group ID of the process or one of the supplementary group IDs of
the process equals the group ID of the file, access is allowed if the appropriate
group access permission bit is set. Otherwise, permission is denied.
4. If the appropriate other access permission bit is set, access is allowed.
Otherwise, permission is denied.

#### 文件系统(important)

![七牛云](http://joyo-pic-1.qiniudn.com/Pic_4_13.png)
![七牛云](http://joyo-pic-1.qiniudn.com/Pic_4_14.png)
i节点几乎包含了文件有关的所有信息: 文件类型, 文件访问权限位, 文件长度和指向文件数据块的指针等. stat结构中的大多数信息都取自i节点. 只有文件名和i节点编号存放在目录项中.
![七牛云](http://joyo-pic-1.qiniudn.com/Pic_4_15.png)

#### Hard link & Symbolic link
硬链接直接指向文件的i节点, 符号链接是对一个文件的间接指针.
在shell中创建硬链接和符号链接非常方便
创建硬链接: `ln sourcefile destfile`
创建符号链接: `ln -s sourcefile destfile`

### reference
[The Standard C](http://www.iso-9899.info/wiki/The_Standard)
