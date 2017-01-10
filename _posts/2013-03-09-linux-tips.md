---
title: Linux 常用技巧
layout: post
comments: true
language: chinese
category: [linux]
keywords: linux,bash,技巧,经典
description: 简单记录下在 Linux 下常用的一些技巧，以方便查询使用，例如 生成随机字符串，特殊字符文件的处理， sudo 和 su 两个命令的区别等等。
---

简单记录下在 Linux 下常用的一些技巧，以方便查询使用，例如 生成随机字符串，特殊字符文件的处理， sudo 和 su 两个命令的区别等等。

<!-- more -->


## 生成随机字符串

在 Linux 中 /dev/urandom 可以用来产生真随机的内容，然后直接读取字符串的内容，从而生成随机的字符串。

如下的命令中，C 表示生成的字符串的字符数；L 表示要生成多少行随机字符。

{% highlight text %}
----- 生成全字符随机的字串
$ cat /dev/urandom | strings -n C | head -n L

----- 生成数字加字母的随机字串
$ cat /dev/urandom | sed 's/[^a-zA-Z0-9]//g' | strings -n C | head -n L
{% endhighlight %}

## 特殊字符文件处理

在 Linux 中，文件名的长度最大可以达到 256 个字符，可以使用字符有：字母、数字、'.'(点)、'\_'(下划线)、'-'(连字符)、' '(空格)，其中开始字符不建议使用 '\_'、'-'、' ' 字符。'/'(反斜线) 用于标示目录，不能用作文件或者文件夹名称。

另外，在 shell 中，'?'(问号)、'*'(星号)、'&' 字符有特殊含义，同样不建议使用。

在 shell 中，将 \-\- 之后的内容当作文件。

{% highlight text %}
$ cd .>-a                             ← 创建一个文件，或者 >-a
$ vi -- -a                            ← 编辑，或者 echo "">-a
$ rm -- -a                            ← 删除，或者 rm ./-a
$ touch '><!*'                        ← 创建
$ touch '?* $&'                       ← 创建
{% endhighlight %}

对于这样的文件，可以执行如下操作。

{% highlight text %}
----- 将非乱码的文件移出到某个目录下
$ find . -name "[a-z|A-Z]*" | xargs -I {} mv {} /somepath

----- 也可以通过inode删除
$ ls -i
$ find -inum XXX | xargs -I {} rm {}
$ find -inum XXX -delete
{% endhighlight %}


## sudo VS. su

在切换时实际上是两种策略：1) su 切换到相应的用户，所以需要切换用户的密码；2) sudo 不知道 root 密码的时候执行一些 root 的命令，需要在 suders 中配置+自己用户密码。

{% highlight text %}
$ sudo su root                        ← 需要用户的密码+sudoers配置
$ su root                             ← 需要root用户密码
{% endhighlight %}

注意，之所以使用 sudo su root 这种方式，可能是 btmp 等类似的文件，只有 root 可以写入，否则会报 Permission Denied，此时可以通过 strace 查看报错的文件。


{% highlight text %}
{% endhighlight %}
