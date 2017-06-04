---
title: Linux 用户管理
layout: post
comments: true
language: chinese
usemath: true
category: [linux,misc]
keywords: linux,用户管理
description: Linux 是多用户系统，可以允许多个用户登陆，这里简单介绍与用户管理相关的内容。
---

Linux 是多用户系统，可以允许多个用户登陆，这里简单介绍与用户管理相关的内容。

<!-- more -->

## 用户管理

在 CentOS 中，useradd 和 adduser 是相同的；Ubuntu 上可能会有所区别，```/etc/login.defs``` 定义了部分在创建用户时的默认配置选项。

useradd 的操作的一般步骤为：

1. 帐号信息添加到 ```/etc/passwd```、```/etc/shadow```、```/etc/group```、```/etc/gshadow``` 文件中。
2. 创建 ```/home/USERNAME``` 目录。
3. 将 ```/etc/skel ``` 目录下的内容复制到 ```/home/USERNAME``` 目录下，很多是隐藏文件。
4. 在 ```/var/mail``` 目录下创建邮箱帐号。

其中第一步基本上所有的发行版本都会执行，而剩余的不同的发行版本会有不同的操作。最后还需要通过 ```passwd USERNAME``` 命令设置用户的密码，CentOS 在没有设置密码时无法登陆。

在通过 ```userdel USERNAME``` 删除用户时，会删除 ```/etc/passwd```、```/etc/group``` 中的内容，但是不会删除 ```/home/user``` 目录以及 ```/var/mail``` 目录下文件，可以使用 ```-r``` 删除这两项。

通过 ```id user``` 命令查看用户。


### 过期设置

通过如下方法设置过期条件。

{% highlight text %}
# useradd USER -e 01/28/12               # 创建用户时指定过期条件
# grep EXPIRE /etc/default/useradd       # 或者修改模板对应的默认参数
# useradd -D -e 01/19/12                 # 修改默认新建帐户过期时间
# useradd -D | grep EXPIR                # 查看
# chage -l USER                          # 查看用户的过期时间
# usermod -e 01/28/12 USER               # 修改用户属性
# chage -E 01/28/12 USER                 # 调整账户过期时间
{% endhighlight %}

## 审计

CentOS 系统上，用户登录历史存储在以下这些文件中：

* ```/var/run/utmp``` 记录当前打开的会话，会被 who 和 w 记录当前有谁登录以及他们正在做什么。
* ```/var/log/wtmp``` 存储系统连接历史记录，被 last 工具用来记录最后登录的用户的列表。
* ```/var/log/btmp``` 失败的登录尝试，被 lastb 工具用来记录最后失败的登录尝试的列表。

实际上，可以直接通过 ```utmpdump``` 将上述文件中保存的数据 dump 出来，另外，默认的登陆日志保存在 ```/var/log/secure``` 。

{% highlight text %}
----- 当前当登录的用户的信息
# who
huey     pts/1        2015-05-11 18:29 (192.168.1.105)
sugar    pts/2        2015-05-11 18:29 (192.168.1.105)

----- 登录的用户及其当前执行的任务
# w
18:30:51 up 3 min,  2 users,  load average: 0.10, 0.14, 0.06
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
huey     pts/1    192.168.1.105    18:29    3.00s  0.52s  0.00s w
sugar    pts/2    192.168.1.105    18:29    1:07   0.47s  0.47s -bash

-----  当前当登录的用户的用户名
# users
huey sugar

-----  当前与过去登录系统的用户的信息
# last
root     pts/3        192.168.1.105    Mon May 11 18:33 - 18:33  (00:00)
sugar    pts/2        192.168.1.105    Mon May 11 18:32   still logged in
sugar    pts/2        192.168.1.105    Mon May 11 18:29 - 18:32  (00:02)
huey     pts/1        192.168.1.105    Mon May 11 18:29   still logged in
reboot   system boot  3.5.0-43-generic Mon May 11 18:27 - 18:33  (00:05)
huey     pts/1        192.168.1.105    Sat May  9 10:57 - 17:31  (06:33)

----- 所有登录系统失败的用户的信息
# lastb
btmp begins Sat May  9 09:48:59 2015

----- 用户最后一次登录的信息
# lastlog
root             pts/3    192.168.1.105    一  5月 11 18:36:43 +0800 2015
huey             pts/1    192.168.1.105    一  5月 11 18:29:40 +0800 2015
mysql                                      **从未登录过**
sshd                                       **从未登录过**

----- 用户连接时间的统计数据
-----   1. 每天的总的连接时间
# ac -d
May  9  total        6.55
Today   total        0.54
----- 2. 每个用户的总的连接时间
# ac -p
    huey                                 6.78
    sugar                                0.23
    root                                 0.12
    total        7.13
{% endhighlight %}

<!--
## 安全加固


centos限制登录失败次数并锁定设置

vim /etc/pam.d/login

在#%PAM-1.0下面添加：
auth required pam_tally2.so deny=5 unlock_time=180 #登录失败5次锁定180秒，不包含root
auth required pam_tally2.so deny=5 unlock_time=180 even_deny_root root_unlock_time=180 #包含root

## 每次修改密码禁止使用前N次用过的密码

出于安全考虑，要求修改linux密码策略，每次修改密码时设置的新密码不能是前n次使用过的密码。配置如下：

Debian / Ubuntu：修改文件 # vi /etc/pam.d/common-password
CentOS / RHEL / RedHat / Fedora 修改文件 # vi /etc/pam.d/system-auth

在 password sufficient pam_unix.so use_authtok md5 shadow remember=10
在相应行的后面添加 remember=10，而不是添加一行！

SUSE比较恶心，找了半天才找到。man pw_check
在/etc/security/pam_pwcheck文件中添加remember=5
passwd:     nullok use_cracklib remember=5

http://www.deer-run.com/~hal/sysadmin/pam_cracklib.html
http://zhidao.baidu.com/link?url=xLcMH0cokvN585CPxKf3QVmmN1wDtgESTpAhl1_cxhPQZ0B3D41DhKZCcXr3E0-1nwfBtSKQWQCNKUGPhRvvcq
-->


## 杂项

简单记录常用的使用技巧。

### wheel

wheel 组是一特殊用户组，被一些 Unix 系统用来控制能否通过 su 命令来切换到 root 用户。

{% highlight text %}
$ grep 'wheel' /etc/group
wheel:x:10:foo,bar,foobar
{% endhighlight %}

可以配置成非 wheel 组的用户不能通过 su 命令切换到 root 用户。

{% highlight text %}
$ grep 'pam_wheel' /etc/pam.d/su
auth            required        pam_wheel.so use_uid

$ grep 'WHEEL' /etc/login.defs
SU_WHEEL_ONLY yes
{% endhighlight %}

这时非 wheel 组的成员用 su 命令切换到 root 时提示权限不够，而用 wheel 组的成员切换没任何问题。


### 忘记root密码

启动进入 Grub 时，通过 e 进入编辑方式，添加 single 参数 (也就是进入单用户模式)，登陆之后，通过 password 修改密码即可。

### 登陆提示信息

涉及的有两个配置文件 ```/etc/issue``` 以及 ```/etc/motd```，其中前者为登陆前的提示，后者为登陆后的提示信息。

如果切换到终端登陆 (注意，是终端，通常为类似 ```CTRL+ALT+F2```)，通常会显示提示信息，该信息是在 ```/etc/issue``` 中进行设置。

{% highlight text %}
----- 提示信息为
CentOS Linux 7 (Core)
Kernel 3.10.0-327.el7.x86_64 on an X86_64

----- 其中/etc/issue配置为
\S
Kernel \r on an \m

----- 配置文件中各个选项的含义为：
    \d 本地端时间的日期；
    \l 显示第几个终端机接口；
    \m 显示硬件的等级 (i386/i486/i586/i686)；
    \n 显示主机的网络名称；
    \o 显示 domain name；
    \r 操作系统的版本 (相当于 uname -r)
    \t 显示本地端时间的时间；
    \s 操作系统的名称；
    \v 操作系统的版本
{% endhighlight %}

其中 motd 为 ```Message Of ToDay``` 的简称，也就是布告栏信息，每次用户登陆的时候都会将该信息显示在终端，可以添加些维护信息之类的。不过缺点是，如果是图形界面，会看不到这些信息。

{% highlight text %}
{% endhighlight %}
