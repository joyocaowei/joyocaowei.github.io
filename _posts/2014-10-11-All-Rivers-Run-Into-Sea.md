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

##### SunOS上获取以前的日期

``` bash
# 获得五天以前的日期, 假设今天是20141030, 那么将得到20141025
perl -MPOSIX -le 'print strftime "%Y-%m-%d", localtime(time - 3600*24*5)'

# 你也可以使用TimeZone的写法, EST是时区, 120=24*5
5DaysBefore=`TZ="EST125EDT" date +'%Y-%m-%d'`

# 这是在GNU date下的写法(无法在SunOS下使用)
date +%Y-%m-%d -d '5 days ago'
```

>During a session, sometimes we have a need to change the system time for our session only. We have used it to simulate time based testing.

>Format `TZ=ESThhEDT`

>The EST set your time to Eastern Standard Time and EDT is Eastern Daylight Time.

>hh is the number of hours you wish to change.

>Example: Currently the system(`date`)
date is `Thu Apr  9 02:31:10 EDT 2015`

>And you wish to set it to yesterday at the same time. You would substitute a positive 29 for hh.

>Now the shell time is(`TZ=EST29EDT date`):
`Wed Apr  8 02:31:14 EDT 2015`

>Why 29 and not 24? The main UNIX clock is set from GMT not EST therefore you have to add 5 hours to your backward calculations to get the same exact time since EST is GMT - 5 hours. 

>Use negitive numbers to set the clock into the future. 

>Also if you need to set the minutes and seconds it is hh:mm:ss. These are all the number of hours, minutes and seconds from GMT that you wish to set. 
This is for Solaris 10.


*未完待续…… 最后更新 2015/04/09*
