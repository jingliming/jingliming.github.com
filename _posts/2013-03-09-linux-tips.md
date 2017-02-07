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

## 文本替换

命令行批量替换多文件中的字符串，常用有三种方法：Mahuinan 法、Sumly 法和 30T 法。

{% highlight text %}
----- Mahuinan法，用sed命令批量替换多个文件中的字符串
$ sed -i "s/orig/sub/g" "`grep orig -rl directory`"
解读：
    sed -i 选项表示原地替换，若只显示结果则用-e替换；
    g 表示全局，否则只替换第一个匹配字符串；
    grep -r 表示对目录递归调用；-l(L小写)列出匹配的文件。

----- Sumly法
$ perl -pi -e "s/China/Sumly/g" /www/*.htm /www/*.txt
解读：
    将www文件夹下所有的htm和txt文件中的"China"都替换为"Sumly"

----- 30T法
$ perl -pi -e 's/baidu/30T/g' `find /www -type f`
解读：
　　将www文件夹下所有文件，不分扩展名，所有的"baidu"都替换为"30T"
{% endhighlight %}

注意，在 Mahuinan 中，为了防止文件名中存在空格，所以使用引号包裹。

<!--
splint替换，首先通过 $ sed -n "/\/\*@[a-z]\{4,20\}@\*\//p" null1.c进行测试，然后对grep进行测试
$ grep "\/\*@[a-z]\{1,\}@\*\/" -lr .  为防止文件名之间由空格，将其由双引号包裹。 最后的实际命令为：
$ sed -i "s/\/\*@[a-z]\{1,\}@\*\/ //g"  "`grep "\/\*@[a-z]\{1,\}@\*\/" -lr .` "
-->


## install 和 cp 命令

两者都可以将文件/目录拷贝到指定的目录，不过 install 允许你控制目标文件的属性，常用于程序的 Makefile 文件。

{% highlight text %}
install [OPTION]... SOURCE DIRECTORY
常用参数：
    -m, --mode=0700
        设定权限；
    -v, --verbose
        打印处理的每个文件/目录名称；
    -d, --directory
        所有参数都作为目录处理，而且会创建指定目录；
    -g, --group=mysql
        设定所属组，而不是进程目前的所属组；
    -o, --owner=mysql
        设定所有者，只适用于超级用户；
    -s, --strip
        使用strip命令删除文件中的符号表 (symbol table)；
    -S, --suffix=exe
        指定安装文件的后缀；
    -p, --preserve-timestamps
        以源文件的访问/修改时间作为相应的目的地文件的时间属性；
    -t, --target-directory
        如果最后一个文件是一个目录并且没有使用-T,--no-target-directory选项，
        则将每个源文件拷贝到指定的目录，源和目标文件名相同；
{% endhighlight %}

<!--
--backup[=CONTROL]：为每个已存在的目的地文件进行备份。 -b：类似 --backup，但不接受任何参数。
-->

使用 -d 选项时，如果指定安装位置为 /usr/local/foo/bar，一般 /usr/loacal 已存在，install 会直接创建 foo/bar 目录，并把程序安装到指定位置。

如果指定了两个文件名，则将第一个文件拷贝到第二个,

{% highlight text %}
----- 安装目录，等价于mkdir /tmp/bin
$ install --verbose -d -m 0755 /tmp/bin

----- 安装文件，等价于cp a/e c
$ install -v -m 0755 a/e c

----- 安装文件，等价于mkdir -p a/b && cp x a/b/c
$ install -v -m 0755 -D x a/b/c
{% endhighlight %}

## 杂项

记录些简单的技巧。

### 写入多行

可以使用 echo 添加到文件，不过这样会比较麻烦，可以使用如下方式；不过 $ 需要做转义。

{% highlight text %}
$ cat << EOF >> /tmp/foobar.conf
net.core.rmem_default = 262144
net.core.rmem_max = 262144
net.core.wmem_default = 262144
net.core.wmem_max = 262144
export PATH=\$PATH:\$HOME/bin
EOF
{% endhighlight %}


{% highlight text %}
{% endhighlight %}
