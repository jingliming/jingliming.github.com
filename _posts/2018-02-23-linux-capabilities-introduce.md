---
title: Linux 系统用户
layout: post
comments: true
language: chinese
category: [linux]
keywords: linux,system user
description: 在 Linux 系统中，很多操作是需要 root 用户权限才能操作的，常见的包括 chown、使用 Raw Sokcet (ping) 等，如果使用 sudo 就会导致配置管理和权限控制比较麻烦。这里简单介绍一下 Linux 中的解决方案。
---

在 Linux 系统中，很多操作是需要 root 用户权限才能操作的，常见的包括 chown、使用 Raw Sokcet (ping) 等，如果使用 sudo 就会导致配置管理和权限控制比较麻烦。

其中一个解决方案是，类似于 passwd ，对一个 owner 为 root 的可执行文件可以增加粘滞位 (Set User ID on execution, SUID)，也就是 `chmod +s` 。

这样在运行的时候使用的就是 root 权限，带来的问题就是会导致其运行的权限过高，在某种程度上增大了安全攻击面，有些平台上 ping 也是采用上述的粘滞位机制。

为了只给这个程序开所需要的权限，Linux 提供了一套 capabilities 机制，这里简单介绍。

<!-- more -->

## 问题简述

通常使用很多的程序会使用一个非特权的系统帐号，假设有监控系统采集服务器的指标，例如通过如下用户添加 `monitor:monitor` 用户以及用户组。

{% highlight bash %}
#! /bin/sh

# NOTE: Command 'id -g monitor' will failed on some platform.
if ! (grep -E '^monitor\>:' /etc/group >/dev/null 2>&1) ; then
        echo "Group 'monitor' does not exist, try to create one"
        /usr/sbin/groupadd -g 68 -o -r monitor >/dev/null 2>&1 || :
else
        echo "Group 'monitor' already exists"
        exit 1
#        gid=`id -g monitor`
#        if [ "x${gid}" != "x68" ]; then
#                echo "Change group id(${gid}) to 68"
#                /usr/sbin/groupmod -g 68 monitor
#        fi
fi

if ! (grep -E '^monitor\>:' /etc/passwd >/dev/null 2>&1) ; then
        echo "User 'monitor' does not exist, try to create one"
        /usr/sbin/useradd -M -g monitor -o -r -d /your/home/path -s /bin/false \
                -c "Monitor Agent" -u 68 monitor >/dev/null 2>&1 || :
else
        echo "User 'monitor' already exists"
        exit 2
#        uid=`id -u monitor`
#        if [ "x${uid}" != "x68" ]; then
#                echo "Change user id(${uid}) to 68"
#                /usr/sbin/usermod -u 68 monitor
#        fi
fi
exit 0
{% endhighlight %}

如上添加的是系统用户，很多目录是没有权限，可以通过如下方式查看是否有权限。

{% highlight text %}
----- 查看是否有访问权限
# su - monitor -s /bin/bash -c "ls -alh /var/lib/mysql/mysql.sock"

----- 查看是否有写权限
# su - monitor -s /bin/bash -c "touch /var/lib/mysql/mysql.sock"
{% endhighlight %}

访问其它用户的文件时需要保证：A) 保证文件对其它用户是可读写的；B) 整个路径目录对其它用户是可执行的。

如果在通过上述的方式检查时发现没有权限，那么需要通过如下命令修改 monitor 用户的属组，同时要保证 相关的文件以及目录有该属组的访问权限 。

如下示例仍以 MySQL 用户为例。

{% highlight text %}
----- 设置monitor用户的默认属组
# usermod -g mysql monitor

----- 检查是否修改成功，注意后面的属组
$ id monitor
uid=68(monitor) gid=995(mysql) groups=995(mysql)
{% endhighlight %}

目前这种方案只能满足一个组件的权限设置，因为主要属组只能设置一个。

## Capabilities 简介

该机制是在 Linux 2.2 之后引入的，它将 root 用户的权限细分为不同的领域，可以分别启用或禁用；从而，在实际进行特权操作时，如果 euid 不是 root，便会检查是否具有该特权操作所对应的 capabilities，并以此为依据，决定是否可以执行特权操作。

注意，capabilities 是细分到线程的，每个线程都有自己对应的 capabilities 。

文件系统支持文件附加属性，使得可执行文件具有一定的 capabilities，从而在运行时确定其 capabilities，而对于文件 capabilities 的支持，直到内核 2.6.24 之后才完成。

也就是说，完整的 capabilities 支持是在 2.6.24 版本以后支持的。

### 设置Capabilities

事实上 Linux 本身对权限的检查就是基于 capabilities 的，root 用户有全部的 capabilities，所以啥都能干；常见的操作如下：

如果想给一个可执行文件加上某个 capability，可以用 setcap 命令，如

{% highlight text %}
----- 对于ping增加Raw Socket权限
$ setcap cap_net_raw=+ep ping
参数：
   cap_net_raw 对应设置capability的名字；
   mode 也就是等号后面的内容，+ 表示启用，- 表示禁用；
      e 是否激活
   p 是否允许进程设置
   i 子进程是否继承

----- 同样以ping为例，可以通过如下方法查看其具有的Capability权限
$ getcap /bin/ping
{% endhighlight %}

这样普通用户执行这个 ping 命令，也可以正常使用 Raw Socket 这个权限了。

内核通过 `setxattr()` 系统调用执行，也就是为文件加上 `security.capability` 扩展属性。


### 线程相关

权限控制的最小粒度是线程，每个线程有三个与 Capabilities 相关的位图，分别为：

* Permitted，定义线程所能够拥有的特权的上限，如果需要的特权不在该集合中，是不能进行设置的；
* Inheritable，执行 fork() 运行其它进程时允许被继承的权限；
* Effective，当前允许执行的特权。

对应进程描述符 `struct task_struct` 中的 `cap_effective`、`cap_inheritable`、`cap_permitted`，可以通过 `/proc/PID/status` 来查看进程的能力。

### 获取和设置

系统调用 `man 2 capget` 和 `man 2 capset` 可被用于获取和设置线程自身的 capabilities；此外，也可以使用 libcap 中提供的接口 `man 3 cap_get_proc` 和 `man 3 cap_set_proc` 。




<!--
下表列出了一些常见的特权操作及其对应的 capability：

改变文件的所属者(chown()) CAP_CHOWN
向进程发送信号(kill(), signal()) CAP_KILL
改变进程的uid(setuid(), setreuid(), setresuid()等) CAP_SETUID
trace进程(ptrace()) CAP_SYS_PTRACE
设置系统时间(settimeofday(), stime()等) CAP_SYS_TIME





当然，Permitted集合默认是不能增加新的capabilities的，除非CAP_SETPCAP在Effective集合中。

如果要查看线程的capabilities，可以通过/proc/<PID>/task/<TID>/status文件，三种集合分别对应于CapPrm, CapInh和CapEff。但这种的显示结果是数值，不适合人类阅读。为此，可使用包libcap中的命令getpcaps <PID>获取该进程的主线程的capabilities。

类似的，如果要查看和设置文件的capabilities，可以使用命令getcap或者setcap。
-->

<!--
运行exec后capabilities的变化
上面介绍了线程和文件的capabilities，可能会觉得有些抽象难懂。下面将使用具体的计算公式，来说明执行exec后capabilities是如何确定的。

我们使用P代表执行exec前的capabilities，P’代表执行exec后的capabilities，F代表exec执行的文件的capabilities。那么：

P’(Permitted) = (P(Inheritable) & F(Inheritable)) | (F(Permitted) & cap_bset)

P’(Effective) = F(Effective) ? P’(Permitted) : 0

P’(Inheritable) = P(Inheritable)

其中的cap_bset是capability bounding set。通过与文件的Permitted集合计算交集，可进一步限制某些capabilities的获取，从而降低了风险。

而正如介绍文件的Effective bit时所说，文件可以将其Effective bit关闭。由此，在通过exec执行该文件后，实际的Effective集合为空集。随后，在需要进行特权操作时，可再将Permitted集合中的capabilities加入Effective集合中。





mkdir -p /tmp/user/group/test
chmod 700 -R /tmp/user
echo "just for test" > /tmp/user/group/test/read.txt
chmod 600 /tmp/user/group/test/read.txt
chown -R mysql:mysql /tmp/user
-->





## 参考

通过 `man 7 capabilities` 查看所有可用的 capabilities，而通过 `man 3 cap_from_text` 可以看到关于 capability mode 的表达式说明。

<!--
http://www.cnblogs.com/iamfy/archive/2012/09/20/2694977.html
-->

{% highlight text %}
{% endhighlight %}
