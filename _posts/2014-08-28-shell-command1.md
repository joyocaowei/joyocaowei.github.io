---
title: shell命令-part1
author: joyocaowei
layout: post
permalink:  /shell-command1/
tags:
- Linux
- shell
---

**工作或者学习 - shell命令记录（part1）**
--------


####  tee - replicate the standard output
```  
pwd | tee pwd.txt | ls | cat -n  
```  
 >1  test.txt  
 >2  pwd.txt  

 pwd.txt的内容就是执行pwd的结果： 
 >/home/caowei/code  
 
<!--more-->  
#### sleep.sh  
```  
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
```  
# "-n"  Restrains ftp from attempting "auto-login" upon initial connection.  
# "-i"  Turns off interactive prompting during multiple file transfers.  
# "-v"  Verbose option forces ftp to show all responses from the remote server, as well as report on data transfer statistics.  

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
#### find及其组合命令  
- Finding the 10 largest size files from a given directory  
```  
find . -type f -exec du -k {} \; | sort -nrk 1 | head  
```  
 >{}是一个特殊的字符串，与-exec结合使用。对于每一个匹配的文件，{}都会被替换成相应的文件名。 

- 清空当前目录下所有文件的内容，但不删除文件  
   `find . -type f -exec cp /dev/null {} \;`  
   
- 统计当前目录中所有txt文件的行数  
  `find . -type f -name "*.txt" | xargs wc -l`  
  
- 查找当前目录及其子目录下包括`About`字符的文件  
`find . -type f | xargs grep -n About`  
>xargs命令把从stdin接受的数据重新格式化，然后再将其作为参数提供给其他命令。 


#### bc  
   `echo $number%2 | bc` **可以用来判断奇偶数**  
