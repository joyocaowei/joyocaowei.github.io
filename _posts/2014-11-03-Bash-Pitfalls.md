---
layout: post
title: Bash Pitfalls
description: 关于bash的一些陷阱以及如何写更健壮的shell代码，主要整理自文章末尾的那些reference文章以及自己平常工作中遇到的坑
keywords: bash, linux
categories: programming
---

#### for i in $(ls *.mp3)
常见的错误代码如下:

``` bash
for i in $(ls *.mp3); do    # Wrong!
  some command $i           # Wrong!
done

for i in $(ls)              # Wrong!
for i in `ls`               # Wrong!
for i in $(find . -type f)  # Wrong!
for i in `find . -type f`   # Wrong!

for i in "$(ls *.mp3)"; do  # Wrong!
```

如果文件中有空格或者通配符的话，就会出现问题。一种较好的形式是：

``` bash
for i in $somedir/*.mp3; do
  # 第三行代码的作用是万一文件不存在不会出现"$somedir/*.mp3"
  [[ -f "$i" ]] || continue
  some command "$i"
done
```

#### cp $file $target
最好写成：`cp "$file" "$target"`
因为文件中有空格(space)或者通配符(wildcards)就有问题了。

#### [ $foo = "bar" ]
正确的写法是：
`[[ $foo = "bar" ]]` 或者 `[[ $foo == "bar" ]]` 或者 `[ "bar" = "$foo" ]`

#### 不要使用[[ $foo > 7 ]]进行比较
If you just want to do a numeric comparison (or any other shell arithmetic), it is much better to just use (( )) instead:

``` bash
# Bash / Ksh
((foo > 7))        # Right!
[[ "$foo" -gt 7 ]] # Works, but is pointless. Most will consider it wrong. Use ((...)) or let instead.
```

#### if [grep foo myfile]
其实`[]`是test命令，但是很多人认为if后面的命令就是这么写的。
真正的if语法是这样的：

``` bash
if COMMANDS; then
  COMMANDS
elif COMMANDS; then
  COMMANDS
else
  COMMANDS
fi
```

所以使用方式是：

```
# 你可以这么用，使用test语法[[ somecode ]]
if [[ -n "${my_var}" ]]; then
  do_something
fi

# 你也可以使用一般的命令
if grep foo myfile >/dev/null 2>&1; then
  some commands
fi
```

If the grep matches a line from myfile, then the exit code will be 0 (true), and the then part will be executed. Otherwise, if there is no matching line, the grep should return a non-zero exit code.

#### 关于read命令

```
#!/bin/bash
# IFS的默认值为空白符(换行符, 制表符或者空格)
# 如果是其它分隔符，要设置IFS以读取字段
OldIFS=$IFS
IFS=','
while read column1 column2 column3; do
  echo "$column1"
  echo "$column2"
  echo "$column3"
done < test
IFS=${OldIFS}
```

#### 更改文件的内容

```
# Don't do this
cat file | sed s/foo/bar/ > file  # Wrong!

# The following is completely portable
sed 's/foo/bar/g' file > tmpfile && mv tmpfile file # Good
# If changed string have space, you should quote it
sed "s/foo/bar other/g" file > tmpfile && mv tmpfile file

# Or you can do it with perl 5.x, file(s) means you can
#+ add one more files
perl -pi -e 's/foo/bar/g' file(s)  # Also good too
```

#### 小心使用`echo $msg`

``` bash
$ ls *.awk
histans1.awk  test.awk
$ msg="Please enter a file name of the form *.awk"
$ echo $msg
Please enter a file name of the form histans1.awk test.awk

$ echo "$msg"    # Right
Please enter a file name of the form *.awk
```

所以你在使用echo输出变量信息时要注意加上引号，另一种安全的写法是：
`printf "%s\n" "$msg"`

#### 少使用for arg in $*
尽量使用`for arg in "$@"`
参考[$* and $@ in bash](http://stackoverflow.com/questions/12314451/accessing-bash-command-line-args-vs?rq=1)

#### 不要这样使用`somecmd 2>&1 >logfile`
**Use: `somecmd >logfile 2>&1` instead**
>Redirections are evaluated left-to-right **before** the command is executed.
>所以你知道为什么`sed 's/foo/bar/' file > file`不对了吧?
>顺便说明一下linux下默认的文件描述符
>stdin(标准输入) - 0
>stdout(标准输出) - 1
>stderr(标准错误) - 2

#### Reference
[Bash Pitfalls](http://mywiki.wooledge.org/BashPitfalls)

[Shell Style Guide](https://google-styleguide.googlecode.com/svn/trunk/shell.xml)

[redirection tutorial](http://wiki.bash-hackers.org/howto/redirection_tutorial "里面还有FD和exec的描述")

[Bash Hackers Wiki Frontpage](http://wiki.bash-hackers.org/start)
