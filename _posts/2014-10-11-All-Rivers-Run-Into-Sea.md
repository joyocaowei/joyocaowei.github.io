---
title: 海纳百川
layout: post
description: All rivers run into sea.[遇到的一些小问题以及参考解法（主要是工作或者学习上的小问题）]
keywords: question
categories: programming
---

##### Linux下删除名字乱码的文件或者文件夹

有时会遇到类似于`20P)Ò`这样的文件夹或者文件，可以查找文件的inode，然后使用find命令删除

使用`ls -i`可以查看文件或者文件夹的inode
>  3146 20P)Ò

然后使用命令`find . -inum 3146 -exec rm -r {} \;`将其删除
> **注意：** 如果是文件的话，命令中的`-r`参数就不需要了

**More**
[Linux下删除名字乱码的文件](http://2922055.blog.51cto.com/2912055/999606)

##### SunOS上获取5天前的日期

``` bash
# 获得五天以前的日期, 假设今天是20141030, 那么将得到20141025
perl -MPOSIX -le 'print strftime "%Y-%m-%d", localtime(time - 3600*24*5)'

# 你也可以使用TimeZone的写法, EST是时区, 120=24*5
# 这是在SunOS系统下的用法
5DaysBefore=`TZ="EST+120" date +'%Y-%m-%d'`

# 这是在GNU date下的写法
date +%Y-%m-%d -d '5 days ago'
```

*未完待续…… 最后更新 2014/10/30*
