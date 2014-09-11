---
title: 用于测试博客的显示效果 
layout: post
description: 用来调试博客的显示效果 
keywords: test 
categories: programming
---

### MathJax

You can render *LaTeX* mathematical expressions using **MathJax**, as on [math.stackexchange.com](http://math.stackexchange.com/):

The *Gamma function* satisfying $$\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$$ is via the Euler integral

$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
$$

> **Tip:** Make sure you include **MathJax** into your publications to render mathematical expression properly. Your page/template should include something like this:

```js
<script type="text/javascript" src="https://stackedit.io/libs/MathJax/MathJax.js?config=TeX-AMS_HTML"></script>
```

$$
f(x,y,z) = 3y^2z \left( 3+\frac{7x+5}{1+y^2} \right)
$$



### 测试代码高亮问题

```C
//insertion_sort.c  
//插入排序使用了增量的方法：在排序子数组A[0..i-1]后，将单个元素A[i]插入子数组的适当位置，产生排序好的子数组A[0..i]
  
#include<stdio.h>  
//定义一个带参数的宏，将数组长度存储在变量len中  
#define GET_ARRAY_LEN(array,len){len=(sizeof(array)/sizeof(array[0]));}  
int main()  
{  
    int len;  
    int array[] = { 31, 41, 59, 26, 41, 58 };  
    GET_ARRAY_LEN(array, len);  
    for (int i = 1; i < len; i++)  
    {  
        int temp = array[i];  
        //insert arrray[i] into sorted sequence array[0..i-1]  
        int j = i - 1;  
        while (j >= 0 && array[j] > temp)  
        {  
            array[j + 1] = array[j];  
            j = j - 1;  
        }  
        array[j + 1] = temp;  
    }  
    for (int k = 0; k < len; k++)  
    {  
        printf("%d\n", array[k]);  
    }  
    return 0;  
}  
```

### 测试表格显示问题

**Markdown Extra** has a special syntax for tables:

Item     | Value
-------- | ---
Computer | $1600
Phone    | $12
Pipe     | $1

You can specify column alignment with one or two colons:

| Item     | Value | Qty   |
| :------- | ----: | :---: |
| Computer | $1600 |  5    |
| Phone    | $12   |  12   |
| Pipe     | $1    |  234  |


#### 测试图片显示问题

![pic1](/images/forkme.png)


#### Table of contents

You can insert a table of contents using the marker `[TOC]`:

[TOC]

也许这个是没有效果的？

--------


