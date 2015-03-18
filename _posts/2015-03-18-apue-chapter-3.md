---
layout: post
title: APUE_3E笔记 - File I/O
description: APUE第三章学习笔记
keywords: Linux
categories: programming
---

这章主要描述的一方面是unbuffered I/O, 主要是这些函数：open, openat, create, close, lseek, read, write, 。另一方面, 讨论多个进程间如何共享文件, 以及所涉及的在内核中的数据结构, 并进一步说明dup, fcntl, sync, fsync和ioctl函数。

#### 文件描述符(File Descriptors)
对于内核而言, 所有打开的文件都通过文件描述符引用. 当我们打开一个现有的文件或者创建一个新文件时, 内核向进程返回一个文件描述符.
按照惯例, unix系统将文件描述符0, 1, 2分别关联进程的标准输入, 标准输出, 标准错误.

#### open
使用open函数可以打开或者创建一个文件.
``` c
#include <fcntl.h>
#include <sys/stat.h>
#include <stdio.h>
#include <errno.h>

int main(void) {

    char filename[] = "test.txt";
    /*可以用于测试一个文件是否存在,若不存在,则创建此文件; 如果文件存在,则出错;这使得测试文件存在和创建文件两者成为一个原子操作(atomic operation)*/
    int flag = O_RDWR | O_CREAT | O_EXCL;    

    /*file permission 755*/
    /*mode定义在sys/stat.h中, 关于各个常量的解释可以查看man 2 open*/
    mode_t mode = S_IRUSR | S_IWUSR | S_IXUSR | S_IRGRP | S_IXGRP| S_IROTH | S_IXOTH;
    int fd = open(filename, flag, mode);
    
    if (fd == -1) {
        /*EEXIST常量可以查看man 3 errno*/
        if (errno == EEXIST) {
            printf("open fail cause file already exists.\n");
        } else {
            printf("open/create file fail\n");
            printf("errno = %d\n", errno);
        }
        return 1;
    }
    printf("fd is %d.\n", fd);
    return 0;
}
```
编译之后,第一次执行会创建test.txt文件, 第二次运行会报错: open fail cause file already exists.

由open函数返回的文件描述符一定是最小的未被使用的描述符数值. 上例中是3.

#### close
关闭一个文件还会释放该进程加在该文件上的所有记录锁. 当一个进程终止时,内核自动关闭它所有打开的文件. 很多程序都利用了这一点而不显式的用close关闭打开的文件, 比如使用exit(0).
```
#include <stdlib.h>
void exit(int status);
```

#### lseek
每个打开文件都有一个与其相关的当前偏移量(current file offset). 通常读和写操作都从当前的文件偏移量处开始, 并使偏移量增加所读写的字节数. 按系统默认情况,当打开一个文件时,除非指定O_APPEND选项,否则该偏移量被设置为0.

``` c
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
```
>若whence是SEEK_SET, 则将该文件的偏移量设置为距离文件开始处offset个字节.
>若whence是SEEK_CUR, 则将该文件的偏移量设置为当前位置加offset(可正可负).
>若whence是SEEK_END, 则将该文件的偏移量设置为文件长度加offset(可正可负).

``` c
#include <unistd.h>
#include <errno.h>
#include <stdio.h>

int main(void){
    /*如果文件描述符只想的是一个管道, FIFO或网络套接字, 则lseek返回-1, 并将errno设置为ESPIPE - Invalid seek (POSIX.1)*/
    if (lseek(STDIN_FILENO, 0, SEEK_CUR) == -1)
        printf("errno = %d\n", errno);
        printf("cannot seek\n");
    else
        printf("seek ok\n");
    exit(0);
}
```
> $ ./seek2 < setfl.c  
seek OK
$ cat Makefile | ./seek2  
errno = 29
cannot seek

lseek仅将当前的文件偏移量记录在内核中, 它并不引起任何I/O操作. 然后该偏移量用于下一个读或者写. 文件偏移量可以大于文件的当前操作, 当对该文件的下一次写将会加长该文件, 并在文件中形成一个空洞. 位于文件中但没有写过字节都被读为0. 文件中的空洞并不要求在磁盘上占用存储区.

``` c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/stat.h>

char buf1[] = "abcdefghij";
char buf2[] = "ABCDEFGHIJ";

int main(void){
	int	fd;
	mode_t FILE_MODE = (S_IRUSR | S_IWUSR | S_IXUSR | S_IRGRP | S_IXGRP| S_IROTH | S_IXOTH);
	/*if ((fd = creat("file.hole", FILE_MODE)) < 0)  "Same as below"*/
	if ((fd = open("file.hole", O_WRONLY | O_CREAT | O_TRUNC, FILE_MODE)) < 0)
		printf("creat error\n");

	if (write(fd, buf1, 10) != 10)
		printf("buf1 write error\n");
	/* offset now = 10 */

	if (lseek(fd, 16384, SEEK_SET) == -1)
		printf("lseek error\n");
	/* offset now = 16384 */

	if (write(fd, buf2, 10) != 10)
		printf("buf2 write error\n");
	/* offset now = 16394 */
	exit(0);
}
```
> $ **./hole2**  
$ **ls -l file.hole**
-rw-r--r-- 1 caowei caowei 16394  3月 13 23:40 file.hole
$ **od -c file.hole**  
0000000   a   b   c   d   e   f   g   h   i   j  \0  \0  \0  \0  \0  \0
0000020  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0
*
0040000   A   B   C   D   E   F   G   H   I   J
0040012
> \# 创建指定大小的file.nohole文件, man dd查看具体操作
$ **dd if=/dev/zero of=file.nohole bs=16394 count=1**  
记录了1+0 的读入
记录了1+0 的写出
16394字节(16 kB)已复制，0.00092656 秒，17.7 MB/秒
$ **ls -ls file.hole file.nohole**  
 8 -rw-r--r-- 1 caowei caowei 16394  3月 13 23:40 file.hole
20 -rw-rw-r-- 1 caowei caowei 16394  3月 13 23:42 file.nohole

#### read和write函数
``` c
#include <unistd.h>
/*返回值: 读到的字节数, 若已到文件尾, 返回0; 出错, 返回-1*/
/*读操作从文件的当前偏移量处开始,在成功返回之前,该偏移量将增加实际读到的字节数*/
ssize_t read(int fd, void *buf, size_t nbytes);

#include <unistd.h>
/*返回值: 成功, 返回已写的字节数; 出错, 返回-1*/
ssize_t write(int fd, const void *buf, size_t nbytes);
```

``` c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

#define	BUFFSIZE 4096
/*使用read和write函数复制一个文件*/
int main(void){
	int	n;
	char buf[BUFFSIZE];

	while ((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0)
		if (write(STDOUT_FILENO, buf, n) != n)
			printf("write error\n");

	if (n < 0)
		printf("read error\n");
	exit(0);
}
```

#### 文件共享
UNIX系统支持在不同的进程间共享打开文件, 内核使用3种数据结构表示打开文件.

![七牛云](http://joyo-pic-1.qiniudn.com/open_file_data_structure.png)

![七牛云](http://joyo-pic-1.qiniudn.com/open_same_file.png)

- 在完成每个write后, 在文件表项中的当前文件偏移量增加所写入的字节数. 如果着导致当前文件偏移量超出了当前文件长度, 则将i节点表项中的当前文件长度设置为当前文件偏移量(即文件被加长了).
- 如果使用了O_APPEND标志打开一个文件,则相应标志也被设置到文件表项的文件状态标志中. **每次对这种具有追加写标志的文件执行写操作时, 文件表项中的当前文件偏移量首先会被设置为i节点表项中的文件长度. 这就使得每次写入的数据都追加到当前文件的末尾.**(构成一个原子操作)  
 
 > 一般而言,原子操作(automic operation)指的是由多步组成的一个操作. 如果该操作原子地执行, 要么执行完所有的步骤, 要么一步也不执行, 不可能只执行所有步骤的一个子集.
- 若一个文件用lseek定位到文件的末尾, 则文件表项中的当前文件偏移量被设置为i节点表项中的当前文件长度.
- lseek函数只修改文件表项中的当前偏移量, 不进行任何I/O操作.

#### reference
[file i/o chp3](http://note.huangz.me/os/apue/chp3.html)
[创建指定大小的文件](http://www.1987.name/342.html)
APUE 3e - Filo I/O
