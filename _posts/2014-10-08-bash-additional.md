---
title: bash终端快捷键和个性化提示
layout: post
description: 一些经常使用的终端快捷键和.bashrc PS1 generator
keywords: bash
categories: configuration
---

#### bash终端上常用的快捷键

`Ctrl+a` 移动到行首

`Ctrl+e` 移动到行尾

`Ctrl+r` 向后搜索历史命令（使用`Ctrl+g`退出搜索）

`Ctrl+h` 退格删除一个字符，相当于`Backspace`键

`Ctrl+u` 从当前光标删除到行首

`Ctrl+k` 从当前光标删除到行尾

`Ctrl+l` 清屏

`Ctrl+f` 光标前移/一个字符

`Ctrl+b` 光标后移一个字符

`Ctrl+d` 删除一个字符

`Alt+f`  向前移动一个单词

`Alt+b`  想后移动一个单词

`Ctrl+p` 上一条命令

`Ctrl+n` 下一条命令

`Ctrl+y` 粘贴刚删除的内容

#### 个性化提示符

这是自己目前在用的，将下面的代码加入到文件`~/.bashrc`

```
PS1="\[\e[00;33m\]\s-\v\[\e[0m\]\[\e[00;37m\]@\h:\[\e[0m\]\[\e[00;36m\][\w]\[\e[0m\]\[\e[00;32m\]$\[\e[0m\]\[\e[00;37m\] \[\e[0m\]"
```

如果你想自定义提示符，可以使用**[PS1 generator](http://bashrcgenerator.com/)**自己定制。


#### More

[终端快捷键列表](http://tuhaihe.com/2013/06/20/bash-shortcuts.html)

[PS1 generator](http://bashrcgenerator.com/)

[Bash Manual - command line editing](https://www.gnu.org/software/bash/manual/bashref.html#Command-Line-Editing)
