---
title: 基本的shell命令使用
layout: post
description: shell的一些简单使用
keywords: linux, shell
categories: programming
---

####  tee - replicate the standard output

```
pwd | tee pwd.txt | ls | cat -n
```
>1  test.txt
>2  pwd.txt
 
pwd.txt的内容就是执行pwd的结果：
>/home/caowei/code

#### sleep.sh
``` bash
#!/bin/bash
#Filename: sleep.sh
echo -n Count:
 tput sc # 存储光标位置
 count=0
 while true; do
     if [ $count -lt 10 ]; then
         (( count++ ))
         sleep 1
         tput rc # 恢复光标位置
         tput ed # 清除从当前光标位置到行尾之间的所有内容
         echo -n $count
     else
         exit 0
         echo ""
     fi
 done
```

#### FTP

``` bash
# "-n" - Restrains ftp from attempting "auto-login" upon initial connection.
# "-i" - Turns off interactive prompting during multiple file transfers.
# "-v" - Verbose option forces ftp to show all responses from the remote server,
#        as well as report on data transfer statistics.

HOST='domain.com'
USER='foo'
PASSWD='password'
ftpresults=`ftp -niv << EOF
open $HOST
user $USER $PASSWD
cd /done
$getstr
bye
EOF`

echo "FTP Results: $ftpresults"
```

#### sqlplus in bash

``` bash
sqlresult=`$ORACLE_HOME/bin/sqlplus -s << EOF
    $UserName/$Password@$ServerName
    set serveroutput off;
    set feedback off;
    set heading off;
    set termout off;
    set verify off;
    $sql;
EOF`
```

#### find及其组合命令
- Finding the 10 largest size files from a given directory

`find . -type f -exec du -k {} \; | sort -nrk 1 | head`

 >{}是一个特殊的字符串，与-exec结合使用。对于每一个匹配的文件，{}都会被替换成相应的文件名。

- 清空当前目录下所有文件的内容，但不删除文件
`find . -type f -exec cp /dev/null {} \;`

- 统计当前目录中所有txt文件的行数
`find . -type f -name "*.txt" | xargs wc -l`

- 查找当前目录及其子目录下包括`About`字符的文件
`find . -type f | xargs grep -n About`
>xargs命令把从stdin接受的数据重新格式化，然后再将其作为参数提供给其他命令。

#### uniq
>By default, `uniq` prints the uniq lines in a sorted file. To return all duplicate lines, the input must first be sorted. for non-sorted data, an alternative command is `sort -u`

`uniq -u`
>--unique, print only unique lines

`uniq -d`
>--repeated, print only duplicate lines

`uniq -c`
>--count, print the number of times each line occurred along with line

#### bc
`echo $number%2 | bc`
>**可以用来判断奇偶数**

`echo "scale=5;e(1)" | bc -l`
>result: 2.71828
>**具体的函数可以参考手册 man bc**

`echo "scale=2;34/3" | bc`
>result: 11.33

#### send e-mail
##### 方法一
`cat test.txt`
> To: A@XXXXX.com
From: B@XXXXX.com
Cc: C@XXXXXX.com
Subject: Test E-mail
>
>This is a test e-mail
.

**then run command:**
`sendmail -t < test.txt`

##### 方法二
`(cat mail.txt; uuencode $1 $1) | mailx -s "$subject" $address`
>发送单个附件

##### 方法三 - 发送多个附件
>uuencode r1.tar.gz r1.tar.gz > /tmp/out.mail
 uuencode r2.tar.gz r3.tar.gz >> /tmp/out.mail
uuencode r3.tar.gz r3.tar.gz >> /tmp/out.mail
cat email.txt >> /tmp/out.mail
mailx -s “Reports” user@my.somewhere.com < /tmp/out.mail

#### 一些命令行
`cat file1 file2 | sort | uniq -d`
>两个文件的交集, 不过要确保file1和file2没有重复行，这样得出的才是真的交集

`sort -t"|" +3 -4 +2 -3 +5 -6n file `
>对多个字段进行排序
`-t"|"` 对文件以 | 作为分列符号
`+3 -4` 对第四个字段作为关键字排序
`+2 -3` 同上，对第三个关键字进行排序
`+5 -6n` 对第六列进行数据排序
最后的内容相当于第四列作为第一优先级排序，之后是第三列，最后是第六列。

`awk '!a[$0]++' filename`
>去除文件的重复行，在某些系统上可能要使用nawk

`sed -n '1p' filename`
>输出文件的第一行内容, 适用于处理小型文件
>如果要处理较大的文件(上百或者上千万行), 使用`sed -n '1p;1q' fielname`
>这条命令将只输出第一行，同时退出程序。

`set`
>`set` 命令除了设置环境变量之外，还会将其他在 `shell` 内的变量通通显示出来，比如使用 `set | more` 查看.

```
# one example for set
# Braces avoiding confusion:
# Output is "a0b0c0"
set -- a b c
echo "${1}0${2}0${3}0"
```

`$(<input)`
>remove all trailing newlines from the command inside them
>比如说你有一个一列数的文件file, 你要变为一行, 可以使用`echo $(<file)`, 也可以使用`cat file | xargs`
>当然这样出现的一行数是以默认的IFS(默认是空格)分隔的

#### Reference
[关于shell脚本编程的10个最佳实践](http://blog.jobbole.com/16604/)

#### 学习
[Filesystem Hierarchy Standard](http://www.pathname.com/fhs/)

[The Linux Programming Interface](http://man7.org/tlpi/)

[The AWK Programming Language](http://plan9.bell-labs.com/cm/cs/awkbook/)
