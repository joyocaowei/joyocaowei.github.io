---
title: 用于测试博客的显示效果 
layout: post
description: 用来调试博客的显示效果 
keywords: test 
categories: programming
---

### MathJax

You can render *LaTeX* mathematical expressions using **MathJax**, as on [math.stackexchange.com](http://math.stackexchange.com/):

用`\(`和`\)`括起来的是行内公式: \\( E=mc^2 \\)
这是另一个行内公式测试： $ E=mc^2 $
用`$$`和`$$`括起的是行公式

The *Gamma function* satisfying $$\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$$ is via the Euler integral

$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
$$

> **Tip:** Make sure you include **MathJax** into your publications to render mathematical expression properly. Your page/template should include something like this:

```js
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"> </script>
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
---      | ---
Computer | $1600
Phone    | $12
Pipe     | $1

You can specify column alignment with one or two colons:

| Item     | Value | Qty    |
| :----    | ----: | :----: |
| Computer | $1600 |  5     |
| Phone    | $12   |  12    |
| Pipe     | $1    |  234   |


#### 测试图片显示问题

存储在github上的图片

![pic1](/images/forkme.png)

在Imgur上的图片
 
![Imgur](http://i.imgur.com/JpVp10u.jpg "Box of blue eyed kittens")

顺便推荐一下Imgur这个网站，作为图床还是不错的，不过免费帐户有时间限制。还可以考虑七牛和[drp.io](http://drp.io/)。

在[康西图床](http://pic.conn.cc/)上的图片

![Tu.conn.cc](http://dn-tucdn.qbox.me/Vy.png/w.jpg "bash terminal")


#### Table of contents

Test insert a table of contents using the marker `[TOC]`:

[TOC]

添加此功能有点麻烦，暂时不需要，作为TODO list的一部分。

--------


