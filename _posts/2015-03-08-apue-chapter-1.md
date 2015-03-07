---
layout: post
title: APUE读书笔记 - UNIX System Overview
description: APU第一章的读书笔记
keywords: linux
categories: programming
---

#### 环境设置

操作环境：Ubuntu14.04
我将源代码解压到目录：`~/codes`
在使用源码编译(make)的时候，出现`/usr/lib/ld: cannot find -lbsd`错误。
执行`sudo apt-get install libbsd-dev`解决上面的错误，再次使用make编译。

首先测试第一个程序**myls.c**

``` c
#include "apue.h"
#include <dirent.h>

int main(int argc, char *argv[]){
    DIR *dp;
    struct dirent *dirp;

    if(argc != 2)
        err_quit("usage: ls directory_name");

    if((dp = opendir(argv[1])) == NULL)
        err_sys("can't open %s", argv[1]);
    while((dirp = readdir(dp)) != NULL)
        printf("%s\n", dirp->d_name);

    closedir(dp);
    exit(0);
}
```

`gcc -o myls myls.c -I ~/codes/apue.3e/include/ -L ~/codes/apue.3e/lib/ -lapue`
> **-I**: 告诉gcc include路径在哪里
> **-L**: 告诉gcc library库的路径
> **-lapue**: 说要在lib中找include下apue文件对应的libapue.a或者libapue.so，就是找到编译过apue.h中的函数。

但是这样做命令行有点长, 可以把[gcc环境变量](http://gcc.gnu.org/onlinedocs/gcc/Environment-Variables.html)导入进来：
>export C_INCLUDE_PATH=\$HOME/codes/apue.3e/include/
>export LIBRARY_PATH=\$HOME/codes/apue.3e/lib/
之后就可以使用`gcc -o myls myls.c -lapue`来编译了。
其实也可以直接写入文件`.bashrc`或者写入一个名为**apue.ini**的配置文件，使用sourc命令`. apue.ini`把变量加载到shell中也可。

#### 进程和线程
程序的执行实例被称为进程(*process*), unix系统确保每个进程都有一个唯一的数字标示符，称为进程ID(*process ID*);

线程是运行在一个单一进程上下文中的逻辑刘，有内核进行调度。一个进程内的所有线程共享同意地址空间、文件描述符、栈以及进程相关的属性。与进程一样，线程也有一个唯一的线程ID(*thread ID*)

##### 简单的unix系统进程控制功能

``` c
#include "apue.h"
#include <sys/wait.h>

int main(void){

    char buf[MAXLINE]; /* form apue.h */
    pid_t pid;
    int status; 

    printf("%% "); /* print promt (printf requires %% to print %)  */
    while (fgets(buf, MAXLINE, stdin) != NULL){
        if (buf[strlen(buf) - 1] == '\n')
            buf[strlen(buf) - 1] = 0; /* replace newline with null */
        if ((pid = fork()) < 0){
            err_sys("fork error");
        }else if (pid == 0) { /* child */
            execlp(buf, buf, (char *)0);
            err_ret("couldn't execute: %s", buf);
            exit(127);
        }

        /* parent */
        if ((pid = waitpid(pid, &status, 0) < 0)){
                err_sys("waitpid error");
        }
        printf("%% ");
   }
   exit(0);
}
```

#### 信号，时间值
信号(signal)用于通知进程发生了某种情况。进程的三种处理信号的方式：
>忽略信号。
>按系统默认方式处理。很多情况下是终止该进程。
>提供一个函数，信号发生时调用该信号(捕捉信号)。

要获取任一进程的时钟时间、用户时间和系统时间只需要使用time(1)命令。
>`time ls -l > /dev/null`
>real	0m0.007s
>user	0m0.000s
>sys	0m0.004s

#### Reference

[An Introduction to GCC - 3.1.2小节](http://www.network-theory.co.uk/docs/gccintro/index.html)
摘抄*APUE Chapter 1 - UNIX System Overview*
