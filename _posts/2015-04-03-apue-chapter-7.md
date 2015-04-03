---
layout: post
title: APUE_3E - Process Environment
description: APUE第七章的读书笔记
keywords: linux
categories: programming
---

#### exit

**The only way a program can be executed by the kernel is if one of the exec functions is called.**

![七牛云](http://joyo-pic-1.qiniudn.com/APUE_7_2.png)

进程自愿终止的唯一办法是显示或者隐式的(通过exit)调用_exit或者_Exit. 进程也可以非自愿地由一个信号终止.

#### 命令行参数

``` c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
	int	i;

	/* echo all command-line args */
	for (i = 0; argv[i] != NULL; i++)
		printf("argv[%d]: %s\n", i, argv[i]);
	exit(0);
}
```

>$ `./a.out a b c`
argv[0]: ./a.out
argv[1]: a
argv[2]: b
argv[3]: c

#### C程序的存储空间布局

- 正文段(Text segment): 已编译程序的机器代码
- 初始化数据段(data segment): 包含了程序中需明确赋初值的全局变量
- 未初始化数据段(bss segment): 未初始化的全局C变量
- 栈(Stack): where automatic variables are stored, along with information that is saved
each time a function is called.
- 堆(Heap): 通常在堆中进行动态分配存储

![七牛云](http://joyo-pic-1.qiniudn.com/APUE_7_6.png)

`size(1)`命令报告正文段, 数据段和bss段的长度(以字节为单位)
>`size a.out`
   text	   data	    bss	    dec	    hex	filename
   1235	    284	      4	   1523	    5f3	a.out
   第四和第五列分别以十进制和十六进制表示的三段总长度

下面的图来自CSAPP figure 9.26 - **The virtual memory of a Linux process**
![七牛云](http://joyo-pic-1.qiniudn.com/CSAPP_9_26.png)

#### 函数setjmp和longjmp

在c语言中,goto语句是不能跨越函数的, 而执行这种类型跳转功能的就是setjmp和longjmp. 因为它们是非局部的, 可以在栈上跳过若干调用帧,返回到当前函数调用路径上的某一个函数中. 这两个函数对于处理发生在深层嵌套函数调用中的出错情况是非常有用的.

``` c
#include <setjmp.h>
/* Returns: 0 if called directly, nonzero if returning from a call to longjmp */
int setjmp(jmp_buf env);
void longjmp(jmp_buf env, int val);
```
