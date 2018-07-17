---
title: Linux 资源限制
layout: post
comments: true
language: chinese
category: [misc, linux]
keywords: Linux,资源限制,fork炸弹,ulimit
description: 如何通过最简单的设置来实现最有效的性能调优，在有限资源的条件下保证程序的运作，ulimit 是在处理这些问题时，经常使用的一种简单手段。ulimit 是一种 Linux 系统的内键功能，它具有一套参数集，用于为由它生成的 shell 进程及其子进程的资源使用设置限制。
---

如何通过最简单的设置来实现最有效的性能调优，在有限资源的条件下保证程序的运作，ulimit 是在处理这些问题时，经常使用的一种简单手段。

ulimit 是一种 Linux 系统的内键功能，它具有一套参数集，用于为由它生成的 shell 进程及其子进程的资源使用设置限制。

<!-- more -->

## Fork 炸弹

简单来说，这是一个恶意程序，它的内部是一个不断在 fork 进程的无限递归循环，不需要有特别的权限即可对系统造成破坏。

Jaromil 在 2002 年通过 Bash 设计了最为精简的一个 fork 炸弹的实现，也就是 `:() { :|:& };:` 或者 `.() { .|.& };.` 。

这行命令如果这样写成如下的多行 Bash Script 就不难理解了：

{% highlight text %}
:()
{
:|: &
}
;
:
{% endhighlight %}

* 第 1 行说明下面要定义一个函数，函数名为小数点或者冒号，没有可选参数。
* 第 2 行表示函数体开始。
* 第 3 行是函数体真正要做的事情，首先它递归调用本函数，然后利用管道调用一个新进程（它要做的事情也是递归调用本函数），并将其放到后台执行。
* 第 4 行表示函数体结束。
* 第 5 行并不会执行什么操作，在命令行中用来分隔两个命令用。从总体来看，它表明这段程序包含两个部分，首先定义了一个函数，然后调用这个函数。
* 第 6 行表示调用本函数。

冒号或者逗号其实是函数名，这个脚本就是在不断的执行该函数，然后不断 fork 出新的进程。

<!--
对于函数名，大家可能会有所疑惑，小数点也能做函数名使用吗？毕竟小数点是 shell 的一个内嵌命令，用来在当前 shell 环境中读取指定 文件，并运行其中的命令。实际上的确可以，这取决于bash对命令的解释顺序。

默认情况下，bash处于非POSIX模式，此时对命令的解释顺序如下：

* 关键字，例如 if、for 等。
* 别名。别名不能与关键字相同，但是可以为关键字定义别名，例如 end=fi。
* 特 殊内嵌命令，例如 break、continue 等。POSIX 定义的特殊内嵌命令包括：.（小数点）、:（冒号）、break、continue、 eval、exec、exit、export、readonly、 return、set、shift、times、trap 和 unset。 bash 又增加了一个特殊的内嵌命令 source。
* 函数。如果处于非 POSIX 模式，bash 会优先匹配函数，然后再匹配内嵌命令。
* 非特殊内嵌命令，例如 cd、test 等。
* 脚本和可执行程序。在 PATH 环境变量指定的目录中进行搜索，返回第一个匹配项。
由 于默认情况下，bash 处于非 POSIX 模式，因此fork炸弹中的小数点会优先当成一个函数进行匹配。(注：使用小数点代替其中的冒号，也能起到完全相同的效果。)

要使用POSIX模式来运行bash脚本，可以使用以下三种方法：

* 使用 –posix 选项启动 bash。
* 在运行bash之后，执行 set -o posix 命令。
* 使用 /bin/sh 。
-->

那么，有没有办法扼制这种情况的发生呢？答案是肯定的，只需设置进程的limit数即可。

在上面的例子中，我们将用户可以创建的最大进程数限制为 128，执行fork炸弹会迅速 fork 出大量进程，此后会由于资源不足而无法继续执行。使用工具ulimit即可设置各种限制数，具体的请参考该工具的man或help。

## Linux 资源限制

一般最常使用的是文件句柄和进程数，这里简单通过这两个介绍。

其中设置时使用最多的是 `limits.conf` 和 `sysctl.conf` 区别在于，前者只针对用户，而后者是针对整个系统参数配置的。

默认的文件句柄限制是 1024，可以通过 `ulimit -n` 查看；系统总的限制保存在 `/proc/sys/fs/file-max` 文件中，可以通过 `/etc/sysctl.conf` 配置文件修改，当前整个系统的文件句柄使用数可以查看 `/proc/sys/fs/file-nr` 。


### ulimit

该命令是 bash 内键命令，可以通过 `type ulimit` 查看，它具有一套参数集，用来为由它生成的 shell 进程及其子进程的资源使用设置限制，针对的是 Per-Process 而非 Per-User 。

ulimit 用于 shell 启动进程所占用的资源，可以用来设置系统的限制，通过 `ulimit -a` 可以查看当前的资源限制，如果通过命令行设置，则只对当前的终端生效。

#### 永久生效

一般有两种方法：

* 命令写至profile或bashrc，也即登陆时自动修改限制，如`ulimit -S -c 0 > /dev/null 2>&1`；
* 在`/etc/security/limits.conf`配置文件中添加记录，并且在`/etc/pam.d/`中的seesion有使用到limit模块，注意需要重启生效。

一般配置文件中的格式如下：

{% highlight text %}
domain type item value
   domain 指定用户名或者通过@指定用户组，其中*表示所有用户
   type 也就是软设置或者硬设置，也就是hard或者soft，通过-标示同时设置两个值；
   item 指定想限制的资源，例如cpu nproc maxlogins等；
   value 相关指标对应的值，其中 unlimited 表示不限制。
{% endhighlight %}

#### Hard VS. Soft

其中的硬限制是实际的限制，而软限制，是 Warnning 限制，只会做出 Warning；在通过 ulimit 设置时分软硬设置，加 `-H` 就是硬，加 `-S` 就是软，默认是 `-S` 。

如果打开文件过多，会导致 `Too many open files` 报错，

### ulimit limits.conf pam_limits

在 Linux 中，每个进程都可以调用 `getrlimit()` 来查看自己的 limits，也可以调用 `setrlimit()` 来改变自身的 soft limits，如果要修改 hard limit，则需要确保进程有 `CAP_SYS_RESOURCE` 权限。

另外，进程 fork() 出来的子进程，会继承父进程的 limits 设定。

`ulimit` 是 shell 的内置命令，同样是调用上述的接口获取改变自身的 limits ，当在 shell 中执行应用程序时，相应的进程就会继承当前 shell 的 limits 设置。

而 shell 的初始 limits 是在启动时通过 `pam_limits` 设定的，这是一个 PAM 模块，用户登录会根据 `limits.conf` 定义的值进行配置。

可以开启 `pam_limits` 的 debug 来查看大致过程：

{% highlight text %}
$ cat /etc/security/limits.conf
$ grep pam_limits /etc/pam.d/password-auth-ac
session     required      pam_limits.so debug
$ tail /var/log/secure
{% endhighlight %}

也即是说：

1. 用户进行登录时触发 `pam_limits` 模块；
2. `pam_limits` 会读取 `limits.conf` 中的配置，并设定用户所登陆 shell 的 limits；
3. 用户登陆 shell 之后，可以通过 ulimit 命令查看或者修改当前 shell 的 limits;
4. 当用户在 shell 中执行程序时，该程序进程会继承 shell 的 limits 值，从而使子进程相关的配置生效。

#### 关于PAM

在通过第二种方式配置永久生效时，可以查看 `/etc/pam.d/login` 文件，确保有 `session required /lib/security/pam_limits.so` 配置项。

简单来说，`limits.conf` 文件实际是 Linux PAM <!-- Pluggable Authentication Modules --> 中 `pam_limits.so` 的配置文件。

例如，限制 admin 用户登录到 sshd 的服务不能超过 2 个。

{% highlight text %}
----- 在/etc/pam.d/sshd中添加
session required pam_limits.so
----- 在/etc/security/limits.conf中添加
admin - maxlogins 2
{% endhighlight %}

如果要查看应用程序知否支持 PAM ，最简答的是通过 ldd 查看动态库。

### 查看进程的限制

进程自己可以通过 `getrlimit()` `prlimit()` 来获得当前 limits 配置，调用 `getrusage()` 获取自身的资源使用量，也可以通过 `/proc/PID/limits` 获取某个进程的 limits 设置。

要查看某个进程的资源使用量，通常可以通过 `/proc/PID/{stat,status}` 查看。具体某个值的含义，可以参考 proc 的手册。

<!---
https://feichashao.com/ulimit_demo/
https://easyengine.io/tutorials/linux/increase-open-files-limit/
-->


## 最大进程数

### 理论最大数

简单来说就是通过全局的段描述符统计，不过太复杂了，后面补充。

<!--
每个进程都要在全局段描述表GDT中占据两个表项
每个进程的局部段描述表LDT都作为一个独立的段而存在，在全局段描述表GDT中要有一个表项指向这个段的起始地址，并说明该段的长度以及其他一些 参数。除上之外，每个进程还有一个TSS结构(任务状态段)也是一样。所以，每个进程都要在全局段描述表GDT中占据两个表项。

GDT的容量有多大呢？
段寄存器中用作GDT表下标的位段宽度是13位，所以GDT中可以有213=8192个描述项。

除一些系统的开销(例如GDT中的第2项和第3项分别用于内核 的代码段和数据段，第4项和第5项永远用于当前进程的代码段和数据段，第1项永远是0，等等)以外，尚有8180个表项可供使用，所以理论上系统中最大的 进程数量是8180/2=4090。
所以系统中理论上最大的进程数是4090
-->

### 实际数目

Linux 中通过 Process Identification Value, PID 来标示进程，其类型为 pid_t 实际上就是 int 类型。

当前系统可创建的最大进程数可通过 `/proc/sys/kernel/pid_max` 方式查看，可以通过 `sysctl -w kernel.pid_max=65535` 命令进行修改。

## 最大线程数

通过 `/usr/include/bits/local_lim.h` 中的 `PTHREAD_THREADS_MAX` 宏定义最大线程数，一般来说，对于 LinuxThreads 一般是 1024；对于目前最常用的 ntpl 是没有硬性限制的，仅受限于系统资源。

这个系统的资源主要就是线程的 stack 所占用的内存，用 `ulimit -s` 可以查看默认的线程栈大小，一般情况下，这个值是 `8M=8192KB` 。

可以写一段简单的代码验证最多可以创建多少个线程

{% highlight text %}
include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void func()
{
}

int main(void)
{
    int i = 0;
    pthread_t thread;

    while ( 1 )
    {
        if (pthread_create(&thread, NULL, func, NULL) != 0)
        {
            return;
        }

        i++;
        printf("i = %d\n", i);
    }

    return EXIT_SUCCESS;
}
{% endhighlight %}

<!-- 试验显示，在我们的系统(Ubuntu-14.04-LTS-64bit)中linuxthreads 上最多可以创建 381 个线程，之后就会返回 EAGAIN -->

### 理论最大线程数

对于 32 位系统来说，最大可创建 381 个线程左右，因为 32 位 Linux 下的进程用户空间是 3G 的大小，也就是 3072M，用 3072M/8M=384，但是实际上代码段和数据段等还要占用一些空间，这个值应该向下取整到 383，再减去主线程，得到 382。

为了突破内存的限制，可以有两种方法：

* 用 ulimit -s 1024 减小默认栈大小；
* 调用pthread_create的时候用pthread_attr_getstacksize设置一个较小的栈大小。

## 最大文件

通过 /proc/sys/fs/file-max 可以查看整个系统可以使用的文件句柄数量，可以通过 `echo 1000000 > /proc/sys/fs/file-max` 临时修改，或者在 `/etc/sysctl.conf` 中设置 `fs.file-max = 1000000` 永久修改。

其中 /proc/sys/fs/nr_open 会标识单个进程可分配的最大文件数，实际的设置值为 `ulimit -n` ，默认是软资源限制，可以通过 `ulimit -Hn` 查看硬资源限制。


{% highlight text %}
{% endhighlight %}
