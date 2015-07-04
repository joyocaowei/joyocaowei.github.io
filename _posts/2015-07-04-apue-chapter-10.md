---
layout: post
title: APUE_3E -Signal
description: APUE Chapter 10 notes
keywords: linux
categories: programming
---

信号是软件中断, 信号提供了异种处理异步事件的方法. 可以使用`kill -l`查看系统的信号.
在头文件`<signal.h>`中, 信号被定义为正整数常量. Linux 3.2.0将信号定义在`<bits/signum.h>`中.

当造成信号的**事件**发生时,为进程产生一个信号(或向一个进程发送一个信号). **事件**可以是硬件异常(如除以0), 软件异常(如alarm定时器超时), 终端产生的信号或调用kill函数. 当一个信号产生时, 内核通常在进程表中以某种形式设置一个标志.

### Signal Concepts

**信号是异步事件的经典实例**. 在某个信号出现时, 可以告诉内核按下列三种方式之一进行处理:
(1). 忽略此信号. 但是SIGKILL和SIGSTOP不能被忽略(它们向内核和超级用户提供了使进程终止或停止的可靠方法).
(2). 捕捉信号. 为了做到这一点, 要通知内核在某种信号发生时, 调用一个用户函数.
(3). 执行系统默认动作. 大多数信号的默认动作是终止该进程. "终止+core"表示在进程当前工作目录的core文件中复制了该进程的内存映像.

![七牛云](http://joyo-pic-1.qiniudn.com/APUE_10_1.png)

### 函数signal

``` c
#include <signal.h>
/*Returns: previous disposition of signal if OK, SIG_ERR on error*/
void (*signal(int signo, void (*func)(int)))(int);
```

>*signo*参数是上图中的信号名. *func*的值是常量SIG_IGN, 常量SIG_DFL或当接到此信号后要调用的函数的地址.
如果指定SIG_INT, 则向内核表示忽略该信号(记住SIGKILL和SIGSTOP不能忽略). 如果指定SIG_DEF, 则表示接到此信号后的动作是系统默认动作.
**当指定函数地址时, 则在信号发生时, 调用该函数, 我们称这种处理为捕捉该信号(signal handler)**.

可以将signal函数原型写成如下格式(之后的例子都用如下定义代替):
``` c
typedef void Sigfunc(int);
Sigfunc *signal(int, Sigfunc *);
```

下面是一个简单的信号处理程序, 它捕捉两个用户定义的信号并打印信号编号.

``` c
/* example: sigusr.c */

#include "apue.h"

static void sig_usr(int);   /* one handler for both signals */

int main(void){
    if (signal(SIGUSR1, sig_usr) == SIG_ERR)
        err_sys("can't catch SIGUSR1");
    if (signal(SIGUSR2, sig_usr) == SIG_ERR)
        err_sys("can't catch SIGUSR2");
    for ( ; ; )
        /*pause函数使调用进程挂起直到捕捉到一个信号*/
        pause();
}

static void sig_usr(int signo)      /* argument is signal number */
{
    if (signo == SIGUSR1)
        printf("received SIGUSR1\n");
    else if (signo == SIGUSR2)
        printf("received SIGUSR2\n");
    else
        err_dump("received signal %d\n", signo);
}
```

>`$ ./sigusr &`
[1] 12878
`$ kill -USR1 12878`
received SIGUSR1
`$ kill -USR2 12878`
received SIGUSR2
`$ kill 12878`
[1]+  Terminated              ./sigusr
**kill(1)命令和kill(2)函数只是将一个信号发送给一个进程或进程组. 该信号是否终止则取决于该信号的类型, 以及进程是否安排了捕捉该信号**

### 可重入函数(Reentrant Functions)

The Single UNIX Specification specifies the functions that are guaranteed to be safe
to call from within a signal handler. These functions are reentrant and are called
async-signal safe by the Single UNIX Specification. Besides being reentrant, they block
any signals during operation if delivery of a signal might cause inconsistencies.

![七牛云](http://joyo-pic-1.qiniudn.com/APUE_10_4.png)

### SIGCHLD

在一个进程终止或停止时, SIGCHLD信号被送给其父进程. 如果父进程希望被告知其子进程的这种状态改变,则应捕捉此信号. 信号捕捉函数中通常要调用一种wait函数以取得子进程ID和其终止状态.

### 函数kill, raise, alarm, pause

``` c
#include <signal.h>
/* Both return: 0 if OK, −1 on error */
int kill(pid_t pid, int signo);
int raise(int signo);
```

``` c
#include <unistd.h>
/* Returns: 0 or number of seconds until previously set alarm */
unsigned int alarm(unsigned int seconds);

/* Returns: −1 with errno set to EINTR */
int pause(void);
```

函数kill将信号发送给进程或进程组(p267). 如果signo参数为0, 则kill仍执行正常的错误检查, 但不发送信号.
函数raise则允许进程向自身发送信号.
`raise(signo) <==> kill(getpid(), signo)`

函数alarm可以设置一个定时器, 在将来的某个时刻超时, 产生SIGALRM信号. 如果忽略或者不捕捉该信号, 默认动作是终止调用该alarm函数的进程.
>每个进程只能有一个闹钟时间. 如果在调用alarm时, 之前已为该进程注册的闹钟时间还没有超时, 则该闹钟时间的余留值作为alarm函数调用的值返回. 以前注册的闹钟时间被新值代替.
如果有以前注册的尚未超时的闹钟时间, 而本次调用的seconds为0, 则取消以前的闹钟时间, 其余留值仍作为alarm函数的返回值.
如果我们想捕捉SIGALRM信号, 则必须在调用alarm之前安装该信号的处理程序.

pause函数使调用进程挂起直到捕捉到一个信号. 只有执行了一个信号处理程序并从其返回时, pause才返回.

### 信号集

``` c
#include <signal.h>
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signo);
int sigdelset(sigset_t *set, int signo);
    /* All four return: 0 if OK, −1 on error */
int sigismember(const sigset_t *set, int signo); 
    /* Returns: 1 if true, 0 if false, −1 on error */
```

### 函数sigprocmask, sigpending

调用sigprocmask函数可以检测或更改, 或者同时检测和更改进程的信号屏蔽字. 注意sigprocmask是仅为单线程进程定义的. 处理多线程进程信号的屏蔽使用pthread_sigmask函数.

``` c
#include <signal.h>
int sigprocmask(int how, const sigset_t *restrict set, sigset_t *restrict oset);
/* Returns: 0 if OK, −1 on error */
```

首先, 若*oset*是非空指针, 那么进程的当前信号屏蔽字通过*oset*返回.
其次, 若*set*是一个非空指针, 则参数*how*指示如何修改当前屏蔽字.
|*how*|description|
|:-----:|:-----:|
|SIG_BLOCK|The new signal mask for the process is the union of its current signal mask and the signal set pointed to by set. That is, set contains the additional signals that we want to block.|
|SIG_UNBLOCK|The new signal mask for the process is the intersection of its current signal mask and the complement of the signal set pointed to by set. That is, set contains the signals that we want to unblock.|
|SIG_SETMASK|The new signal mask for the process is replaced by the value of the signal set pointed to by set.|

最后, 如果*set*是个空指针, 则不改变该进程的信号屏蔽字, *how*的值也无意义.
>每个进程都有一个信号屏蔽字(signal mask), 它规定了当前要阻塞递送到该进程的信号集.

**在调用sigprocmask后如果有任何未决的, 不再阻塞的信号, 则在sigprocmask返回前, 至少将其中之一递交给该进程.**
>**未决**: 在信号产生(generation)和递送(delivery)之间的时间间隔内, 称信号是未决的(pending).


``` c
#include <signal.h>
int sigpending(sigset_t *set);
/* Returns: 0 if OK, −1 on error */
```

**sigpending**函数返回一信号集, 对于调用进程而言, 其中的各信号是阻塞不能递送的, 因而也一定是当前未决的. 该信号集通过*set*参数返回.

``` c
/* APUE_3E p277 - critical.c */
#include "apue.h"

static void sig_quit(int);

int main(void)
{
    sigset_t    newmask, oldmask, pendmask;

    if (signal(SIGQUIT, sig_quit) == SIG_ERR)
        err_sys("can't catch SIGQUIT");

    /*
     * Block SIGQUIT and save current signal mask.
     */
    sigemptyset(&newmask);
    sigaddset(&newmask, SIGQUIT);
    if (sigprocmask(SIG_BLOCK, &newmask, &oldmask) < 0)
        err_sys("SIG_BLOCK error");

    sleep(5);   /* SIGQUIT here will remain pending */

    if (sigpending(&pendmask) < 0)
        err_sys("sigpending error");
    if (sigismember(&pendmask, SIGQUIT))
        printf("\nSIGQUIT pending\n");

    /*
     * Restore signal mask which unblocks SIGQUIT.
     */
    if (sigprocmask(SIG_SETMASK, &oldmask, NULL) < 0)
        err_sys("SIG_SETMASK error");
    printf("SIGQUIT unblocked\n");

    sleep(5);   /* SIGQUIT here will terminate with core file */
    exit(0);
}

static void sig_quit(int signo)
{
    printf("caught SIGQUIT\n");
    if (signal(SIGQUIT, SIG_DFL) == SIG_ERR)
        err_sys("can't reset SIGQUIT");
}
```

>`$ ./critical`
^\              // *generate signal once (before 5 seconds are up)*
SIGQUIT pending  // *after return from sleep*
caught SIGQUIT  // *in signal handler*
SIGQUIT unblocked // *after return from sigprocmask*
^\Quit  // *generate signal again*


### 函数sigaction

```c
#include <signal.h>
int sigaction(int signo, const struct sigaction *restrict act, struct sigaction *restrict oact);
/* Returns: 0 if OK, −1 on error */
```

sigaction函数的功能是检查或修改(或检查并修改)于指定信号相关联的处理动作. 此函数取代了unix早起版本使用的signal函数. 其中参数*signo*是要检测或者修改其具体动作的信号编号. 若*act*指针非空, 则要修改其动作. 如果*oact*指针非空, 则系统由oact指针返回该信号的上一个动作. 其中结构体(*act, oact*)如下所示:

``` c
struct sigaction {
/* addr of signal handler, or SIG_IGN, or SIG_DFL */
void (*sa_handler)(int);

/* additional signals to block */
sigset_t sa_mask;

/* signal options */
int sa_flags;

/* alternate handler */
void (*sa_sigaction)(int, siginfo_t *, void *);
};
```

### 函数sigsetjmp和siglongjmp

``` c
#include <setjmp.h>
/* Returns: 0 if called directly, nonzero if returning from a call to siglongjmp */
int sigsetjmp(sigjmp_buf env, int savemask);

void siglongjmp(sigjmp_buf env, int val);
```

当捕捉到一个信号时, 进入信号捕捉函数, 此时当前信号被自动的加入进程的信号屏蔽字中. 这两个函数用于信号处理程序中的非局部转移. 如果*savemask*不为0, 则sigsetjmp在*env*中保存当前进程的信号屏蔽字. 调用siglongjmp时, 如果sigsetjmp已经调用了非0的*savemask*(env保存当前进程的信号屏蔽字), 那么siglongjmp从env中恢复保存的信号屏蔽字.

``` c
#include "apue.h"
#include <setjmp.h>
#include <time.h>

static void                     sig_usr1(int);
static void                     sig_alrm(int);
static sigjmp_buf               jmpbuf;
static volatile sig_atomic_t    canjump;

int main(void){
    if (signal(SIGUSR1, sig_usr1) == SIG_ERR)
        err_sys("signal(SIGUSR1) error");
    if (signal(SIGALRM, sig_alrm) == SIG_ERR)
        err_sys("signal(SIGALRM) error");

    pr_mask("starting main: ");     /* {Prog prmask} */

    if (sigsetjmp(jmpbuf, 1)) {

        pr_mask("ending main: ");

        exit(0);
    }
    canjump = 1;    /* now sigsetjmp() is OK */

    for ( ; ; )
        pause();
}

static void sig_usr1(int signo){
    time_t  starttime;

    if (canjump == 0)
        return;     /* unexpected signal, ignore */

    pr_mask("starting sig_usr1: ");

    alarm(3);               /* SIGALRM in 3 seconds */
    starttime = time(NULL);
    for ( ; ; )             /* busy wait for 5 seconds */
        if (time(NULL) > starttime + 5)
            break;

    pr_mask("finishing sig_usr1: ");

    canjump = 0;
    siglongjmp(jmpbuf, 1);  /* jump back to main, don't return */
}

static void sig_alrm(int signo){
    pr_mask("in sig_alrm: ");
}
```

``` bash
caowei@Ubuntu:~/codes/apue.3e/signals$ ./mask &
[1] 4251
caowei@Ubuntu:~/codes/apue.3e/signals$ starting main: 
kill -10 4251 (也可使用 kill -USR1 4251)
starting sig_usr1:  SIGUSR1
caowei@Ubuntu:~/codes/apue.3e/signals$ in sig_alrm:  SIGUSR1 SIGALRM
finishing sig_usr1:  SIGUSR1
ending main: 

[1]+  Done                    ./mask
```

### 函数sigsuspend | 函数abort | 函数system

``` c
#include <signal.h>
int sigsuspend(const sigset_t *sigmask);
    /* Returns: −1 with errno set to EINTR(表示被中断的系统调用) */
```

>进程的信号屏蔽字设置为*sigmask*指向的值. 在捕捉到一个信号或发生一个会终止该进程的信号之前, 该进程被挂起. 如果捕捉到一个信号(除了sigmask指向的屏蔽字)并且从该处理程序返回, 则sigsuspend返回, **并且该进程的信号屏蔽字设置为调用sigsuspend之前的值**.
应用的例子(p287-290): 1. 保护关键代码区, 使其不被特定信号中断的方法. 2. 等待一个信号处理程序设置一个全局变量. 3. 可以用信号实现父进程于子进程之间的同步.

``` c
#include <stdlib.h>
void abort(void); /*此函数不返回值*/
```

abort函数的功能是使程序异常终止. 此函数将SIGABRT信号发送给调用进程(进程不应忽略该信号).
让进程捕捉SIGABRT的意图是: 在进程终止之前由其执行所需的清理操作.

``` c
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void abort(void)            /* POSIX-style abort() function */
{
    sigset_t            mask;
    struct sigaction    action;

    /* Caller can't ignore SIGABRT, if so reset to default */
    sigaction(SIGABRT, NULL, &action);
    if (action.sa_handler == SIG_IGN) {
        action.sa_handler = SIG_DFL;
        sigaction(SIGABRT, &action, NULL);
    }
    if (action.sa_handler == SIG_DFL)
        fflush(NULL);           /* flush all open stdio streams */

    /* Caller can't block SIGABRT; make sure it's unblocked */
    sigfillset(&mask);
    sigdelset(&mask, SIGABRT);  /* mask has only SIGABRT turned off */
    sigprocmask(SIG_SETMASK, &mask, NULL);
    kill(getpid(), SIGABRT);    /* send the signal */

    /* If we're here, process caught SIGABRT and returned */
    fflush(NULL);               /* flush all open stdio streams */
    action.sa_handler = SIG_DFL;
    sigaction(SIGABRT, &action, NULL);  /* reset to default */
    sigprocmask(SIG_SETMASK, &mask, NULL);  /* just in case ... */
    kill(getpid(), SIGABRT);                /* and one more time */
    exit(1);    /* this should never be executed ... */
}
```

函数system在程序中执行一个命令字符串很方便. POSIX.1要求system忽略SIGINT和SIGQUIT, 阻塞SIGCHLD.

``` c
#include <stdlib.h>
int system(const char *cmdstring);
```

``` c
/* system函数的POSIX.1的实现 */
#include    <sys/wait.h>
#include    <errno.h>
#include    <signal.h>
#include    <unistd.h>

int system(const char *cmdstring)   /* with appropriate signal handling */
{
    pid_t               pid;
    int                 status;
    struct sigaction    ignore, saveintr, savequit;
    sigset_t            chldmask, savemask;

    if (cmdstring == NULL)
        return(1);      /* always a command processor with UNIX */

    ignore.sa_handler = SIG_IGN;    /* ignore SIGINT and SIGQUIT */
    sigemptyset(&ignore.sa_mask);
    ignore.sa_flags = 0;
    if (sigaction(SIGINT, &ignore, &saveintr) < 0)
        return(-1);
    if (sigaction(SIGQUIT, &ignore, &savequit) < 0)
        return(-1);
    sigemptyset(&chldmask);         /* now block SIGCHLD */
    sigaddset(&chldmask, SIGCHLD);
    if (sigprocmask(SIG_BLOCK, &chldmask, &savemask) < 0)
        return(-1);

    if ((pid = fork()) < 0) {
        status = -1;    /* probably out of processes */
    } else if (pid == 0) {          /* child */
        /* restore previous signal actions & reset signal mask */
        sigaction(SIGINT, &saveintr, NULL);
        sigaction(SIGQUIT, &savequit, NULL);
        sigprocmask(SIG_SETMASK, &savemask, NULL);

        execl("/bin/sh", "sh", "-c", cmdstring, (char *)0);
        _exit(127);     /* exec error */
    } else {                        /* parent */
        while (waitpid(pid, &status, 0) < 0)
            if (errno != EINTR) {
                status = -1; /* error other than EINTR from waitpid() */
                break;
            }
    }

    /* restore previous signal actions & reset signal mask */
    if (sigaction(SIGINT, &saveintr, NULL) < 0)
        return(-1);
    if (sigaction(SIGQUIT, &savequit, NULL) < 0)
        return(-1);
    if (sigprocmask(SIG_SETMASK, &savemask, NULL) < 0)
        return(-1);

    return(status);
}
```
