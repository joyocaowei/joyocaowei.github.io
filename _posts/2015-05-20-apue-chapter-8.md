fork可以创建新进程, exec可以初始执行新的程序. exit函数和wait函数处理终止和等待终止. 这些是基本的进程控制原语.

### 进程标识

``` c
#include <sys/types.h>
#include <unistd.h>

/*returns  the process ID of the calling process.  (This is often used by routines that generate unique temporary filenames.)*/
pid_t getpid(void);
/*returns the process ID of the parent of the calling process.*/
pid_t getppid(void); 
```

在shell中使用`echo $$`可获得当前进程的pid值
>`echo $$`
2961
`ps -p 2961`
  PID TTY          TIME CMD
 2961 pts/14   00:00:00 bash

进程ID为0的进程通常是调度进程, 常常被称为交换进程(swapper). 该进程是内核的一部分, 它并不执行任何磁盘上的程序, 也被称为系统进程. 
进程ID为1通常是init进程, 在自举过程结束时由内核调用. init进程不会终止, 它是一个普通的用户进程, 但是以超级用户权限运行.

### 函数fork

``` c
#include <unistd.h>
pid_t fork(void);
```

forking是Unix编程中最强大的概念之一. 一个现有进程可以调用fork创建一个新进程. 这个新进程被称为子进程(child process). fork函数被调用一次, 但返回两次. 两次返回的区别是子进程的返回值是0, 而父进程的返回值则是子进程的进程ID. 子进程和父进程继续执行fork之后的指令. 子进程是父进程的副本. 例如, 子进程获得父进程数据空间, 堆和栈的副本. 父进程和子进程并不共享这些存储的内容. 它们共享正文段(CPU执行的机器指令部分, 常常是只读的).

由于在fork之后常常跟随着exec, 所以现在的很多实现并不执行一个父进程数据段, 栈和堆的完全副本. 作为替代, 使用了写时复制(Copy-On-Write)技术.

#### 文件共享

fork的一个特性是父进程所有的打开文件描述符都被复制到子进程中, 并且父进程和子进程共享同一个文件偏移量.

![七牛云](http://joyo-pic-1.qiniudn.com/APUE_8_2.png)

在fork之后处理文件描述符有以下两种常见的情况:

- 父进程等待子进程完成. 在这种情况下, 父进程无需对其描述符做任何处理. 当子进程终止后, 它曾进行过读, 写操作的任一共享文件描述符的文件偏移量已做了相应更新.
- 父进程和子进程各自执行不同的程序段. 在这种情况下, 在fork之后, 父进程和子进程各自关闭它们不需要使用的文件描述符, 这样不会干扰对方的文件描述符. 这种方法是网络服务经常使用的(父进程等待客户端的服务请求, 当这种请求到达时, 父进程调用fork, 使子进程处理此请求, 父进程等待下一个请求).

### 函数exit

不管系统如何终止, 最后都会执行内核中的同一段代码. 这段代码为相应进程关闭所有打开的描述符, 释放它所使用的存储器等. 在任意一种情况下, 该终止进程的父进程都能利用wait或者waitpid函数取得其终止状态.

如果父进程在子进程之前终止, 那么这些子进程的父进程都改变为init进程, 这种处理方法保证了每个进程都一个父进程.
init进程被编写成无论何时只要有一个子进程终止, init就会调用一个wait函数取得其终止状态. 这样防止了系统中塞满僵尸进程.

内核为每个终止子进程保存了一定量的信息, 所以当终止进程的父进程调用wait或者waitpid时, 可以得到这些信息. 这些信息至少包括进程ID, 该进程的终止状态以及该进程使用的CPU时间总量.
内核可以释放终止进程所使用的存储区, 关闭所有打开文件.在UNIX术语中, 一个已经终止的但是其父进程还未对其进行善后处理(获取终止进程的有关信息,释放它占用的资源)的进程被称为僵尸进程(zombie). ps(1)命令将僵尸进程的状态打印为Z.

### 函数wait和waitpid

``` c
#include <sys/wait.h>
pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
/*Both return: process ID if OK; 0, or −1 on error*/
```

调用wait或者waitpid的进程会发生什么

- 如果其所有子进程都还在运行, 则阻塞(父进程一直等到它的某个子进程退出之后才继续执行, 因为wait返回终止子进程的进程ID, 所以它总能了解是哪一个子进程终止了).
- 如果一个子进程已终止, 正等待父进程获取其终止状态, 则取得该子进程的终止状态立即返回.
- 如过它没有任何子进程, 则立即出错返回.

Exmple, 如果一个进程fork一个子进程, 但不要它等待子进程终止, 也不希望子进程处于僵尸状态直到父进程终止, 实现这一要求的诀窍是调用fork两次.

``` c
#include "apue.h"
#include <sys/wait.h>

int main(void){
    pid_t   pid;

    if ((pid = fork()) < 0) {
        err_sys("fork error");
    } else if (pid == 0) {      /* first child */
        if ((pid = fork()) < 0)
            err_sys("fork error");
        else if (pid > 0)
            exit(0);    /* parent from second fork == first child */

        /*
         * We're the second child; our parent becomes init as soon
         * as our real parent calls exit() in the statement above.
         * Here's where we'd continue executing, knowing that when
         * we're done, init will reap our status.
         */
        sleep(2);
        printf("second child, parent pid = %ld\n", (long)getppid());
        exit(0);
    }

    if (waitpid(pid, NULL, 0) != pid)   /* wait for first child */
        err_sys("waitpid error");

    /*
     * We're the parent (the original process); we continue executing,
     * knowing that we're not the parent of the second child.
     */
    exit(0);
}
```

>`$ ./fork2`
$ second child, parent pid = 1554(这里是user的init进程)
`ps -ef | grep init`
root         1     0  0 14:06 ?        00:00:01 /sbin/init
caowei    **1554**  1236  0 14:07 ?        00:00:00 init --user

#### 竞争条件(race condition)

当多个进程都企图对共享数据进行某种处理, 而最后的结果有取决于进程运行的顺序时, 我们认为发生了竞争条件.
如果一个进程希望等待一个子进程终止, 则它必须调用wait函数中的一个.
为了避免竞争条件和轮询(polling), 在多个进程之间需要有某种形式的信号发送和接受的方法. 在unix中可以使用信号机制, 各种形式的进程间通信(IPC)也可使用.

``` c
#include "apue.h"

static void charatatime(char *);

int main(void){
    pid_t   pid;
    TELL_WAIT();

    if ((pid = fork()) < 0) {
        err_sys("fork error");
    } else if (pid == 0) {
        WAIT_PARENT();      /* parent goes first */
        charatatime("output from child\n");
    } else {
        charatatime("output from parent\n");
        TELL_CHILD(pid);
    }
    exit(0);
}

static void charatatime(char *str){
    char    *ptr;
    int     c;

    setbuf(stdout, NULL);           /* set unbuffered */
    for (ptr = str; (c = *ptr++) != 0; )
        putc(c, stdout);
}
```

### 函数exec

当进程调用一种exec函数时,该进程执行的程序完全替换为新程序, 而新程序则从其main函数开始执行. 因为调用exec并不创建新进程, 所有前后的进程ID并未改变. exec只是用磁盘上的一个新程序替换了当前进程的正文段, 数据段, 堆段和栈段.

![七牛云](http://joyo-pic-1.qiniudn.com/APUE_8_15.png)

文件echoall1.c

``` c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]){
    int         i;
    char        **ptr;
    /*libc中定义的全局变量environ指向环境变量表*/
    extern char **environ;

    for (i = 0; i < argc; i++)      /* echo all command-line args */
        printf("argv[%d]: %s\n", i, argv[i]);

    for (ptr = environ; *ptr != 0; ptr++)   /* and all env strings */
        printf("%s\n", *ptr);

    exit(0);
}
```

文件exec1.c

``` c
#include "apue.h"
#include <sys/wait.h>

char    *env_init[] = { "USER=unknown", "PATH=/tmp", NULL };

int main(void){
    pid_t   pid;

    if ((pid = fork()) < 0) {
        err_sys("fork error");
    } else if (pid == 0) {  /* specify pathname, specify environment */
        if (execle("/home/caowei/codes/apue.3e/proc/echoall1", "echoall1", "myarg1",
                "MY ARG2", (char *)0, env_init) < 0)
            err_sys("execle error");
    }

    if (waitpid(pid, NULL, 0) < 0)
        err_sys("wait error");

    if ((pid = fork()) < 0) {
        err_sys("fork error");
    } else if (pid == 0) {  /* specify filename, inherit environment */
        if (execlp("echoall", "echoall", "only 1 arg", (char *)0) < 0)
            err_sys("execlp error");
    }

    exit(0);
}
```
