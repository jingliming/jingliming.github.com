---
title: systemd 使用简介
layout: post
comments: true
language: chinese
category: [linux,misc]
keywords: collectd,monitor,linux
description: 介绍下 Collectd 中源码的实现。
---

现在一般新发行的版本会采用新的 init 进程，也就是 systemd ，其中启动过程可以通过 man bootup 查看。

在此，简单介绍一下 systemd 。

<!-- more -->

## 简介

Linux 内核通过执行 init 将 CPU 的控制权限，交给其它的任务，在 CentOS 中可以通过 rpm -qif \`which init\` 来查看 init 来自于那个包。

init 进程经历了，SysV、Upstart 以及 systemd，而 systemd 是最具争议的一个项目，因为其不只替换了 init，而且还包括了一整套的系统任务。

作为最新的系统和服务管理器，其设计思路借鉴了 Mac 系统的启动程序 Launchd，兼容 SysV 和 LSB 的启动脚本。事实上其作用远不仅是启动系统，它还接管了系统服务的启动、结束、状态查询和日志归档等职责，并支持定时任务和通过特定事件，如插入特定 USB 设备和特定端口数据触发的任务。

有以下特性：支持并行化任务、同时采用socket和D-BUS总线式激活服务、按需启动相应的守护进程、利用Linux的cgroup监控进程、支持快照和系统恢复、维护挂载点和自动挂载点，各服务间基于依赖关系进行精密控制。


<!--
Systemd同时也清晰地处理了系统关机过程。它在/usr/lib/systemd/目录下有三个脚本，分别叫systemd-halt.service，systemd-poweroff.service，systemd-reboot.service。这几个脚本会在用户选择关机，重启或待机时执行。在接收到关机事件时，systemd首先卸载所有文件系统并停止所有内存交换设备，断开存储设备，之后停止所有剩下的进程。
-->

可以通过 pstree 查看启动的进程树，接下来查看一下 systemd 的特性。


## Systemd

Systemd 使用 "target" 来处理引导和服务管理过程，这些 systemd 里的 "target" 文件被用于分组不同的引导单元以及启动同步进程。

如果 A 要求 B 在 A 之前运行，则在 [Unit] 段中添加 Requires=B 和 After=B，如果依赖关系是可选的，可添加 Wants=B 和 After=B；注意 Wants= 和 Requires= 并不意味着 After=，即如果 After= 选项没有设置，这两个单元将被并行启动。

### 执行顺序

执行的第一个目标是 /etc/systemd/system/default.target，通常指向 /usr/lib/systemd/system/graphical.target，该文件为文本，可以查看其实际会依次依赖于 multi-user.target => basic.target => sysinit.target。

local-fs.target 单元不会启动用户相关的服务，它只处理底层核心服务，会根据 /etc/fstab 和 /etc/inittab 来执行相关操作。

sysinit.target 会启动重要的系统服务例如系统挂载，内存交换空间和设备，内核补充选项等等。basic.target 会启动普通服务特别是图形管理服务，它通过 /etc/systemd/system/basic.target.wants 目录来决定哪些服务会被启动。

在这个阶段，会启动multi-user.target而这个target将自己的子单元放在目录“/etc/systemd/system/multi-user.target.wants”里。这个target为多用户支持设定系统环境。非root用户会在这个阶段的引导过程中启用。防火墙相关的服务也会在这个阶段启动。

登陆是通过 systemd-logind.service 进行，可以通过 systemctl help systemd-logind.service 查看帮助，通常是针对 XWindow，而终端登陆则通过 /usr/lib/systemd/system/getty@.service 执行。



### 自动登陆

在此查看下如何自动登陆，首先创建一个新的类似于 getty@.service 的服务。

{% highlight text %}
# cp /lib/systemd/system/getty@.service /etc/systemd/system/autologin@.service
# ln -s /etc/systemd/system/autologin@.service /etc/systemd/system/getty.target.wants/getty@tty8.service
# vi /etc/systemd/system/getty.target.wants/getty@tty8.service
...
ExecStart=-/sbin/mingetty --autologin USERNAME %I
Restart=no
...
Alias=getty.target.wants/getty@tty8.service
{% endhighlight %}

最后重新加载守护进程，运行服务：

{% highlight text %}
# systemctl daemon-reload
# systemctl start getty@tty8.service
{% endhighlight %}

需要注意的是，如果你退出了 tty8 的会话，你需要等到下次重新启动才能使用，除非你给 Restart 的值是 'always' ，这样你可以使用systemctl 手动开启（但是出于安全考虑，强烈建议你不要那么做）。

### 启动优化

sysvinit 只能一次一个串行地启动进程，而 Systemd 则并行地启动系统服务进程，并且最初仅启动确实被依赖的那些服务，极大地减少了系统引导的时间。

任何启动项，只要是在系统启动时有被执行到，不论启动成功还是失败，systemd 都能够记录下他们的状态，可以通过 systemctl 查看当前的服务。

详细信息可以通过 systemctl status systemd-logind.service 查看；启动的结构树可以通过 systemd-cgls 查看。

{% highlight text %}
# systemd-analyze                         // 查看系统引导用时
# systemd-analyze time                    // 同上
# systemd-analyze blame                   // 查看初始化任务所消耗的时间
# systemd-analyze plot > systemd.svg      // 将启动过程输出为svg图
# systemd-cgtop                           // 查看资源的消耗状态
{% endhighlight %}



### 常用 systemctl 命令

通过 systemctl 命令可以管理整个系统。

{% highlight text %}
# systemctl                               // 列出正在运行的单元
# systemctl list-units                    // 同上
# systemctl --failed                      // 查看失败的任务
# systemctl list-unit-files               // 所有已经安装的任务
{% endhighlight %}

所有可用的单元文件存放在 /usr/lib/systemd/system 和 /etc/systemd/system 目录中，一个单元配置文件可以为，系统服务(.service) 、挂载点(.mount)、sockets(.sockets)、系统设备、交换分区/文件、启动目标(target)、文件系统路径、由 systemd 管理的计时器，详见 man 5 systemd.unit 。

### 设置启动级别

在 sysVinit 的 runlevels 大多是以数字分级的，常用的命令如下。

{% highlight text %}
# systemctl isolate graphical.target       // 改变当前目标
# systemctl list-units --type=target       // 列出当前目标
# systemctl get-default                    // 查看默认目标
# systemctl set-default graphical.target   // 改变默认目标
{% endhighlight %}


### 管理目标

服务 systemctl 脚本存放在 /usr/lib/systemd/ 目录下，有系统 (system) 和用户 (user) 之分，常见的服务如 nginx 等存放在 /usr/lib/systemd/system 目录下；下面以 nginx 为例，编写脚本时可以直接参考 nginx 的编写方法。

{% highlight text %}
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=process
KillSignal=SIGQUIT
TimeoutStopSec=5
PrivateTmp=true

[Install]
WantedBy=multi-user.target
{% endhighlight %}

一个服务以 .service 结尾，一般会分为 3 部分：[Unit]、[Service] 和 [Install]。

* [Unit]<br>内容包括 Description 和 After，分别用于描述服务以及服务类别。

* [Service]<br>服务的一些具体运行参数的设置：Type=forking (后台运行)；PIDFile (PID文件路径)；ExecStart (服务的具体运行命令)；ExecReload (重启命令)；ExecStop (停止命令)；PrivateTmp=True (给服务分配独立的临时空间)；注意，命令应该使用绝对路径。

* [Install]<br>服务安装的相关设置，可设置为多用户。

服务脚本按照上面编写完成后，以 754 的权限保存在 /usr/lib/systemd/system 目录下，这时就可以利用 systemctl 进行配置了。


### 查看日志

journald 是 systemd 独有的日志系统，替换了 sysVinit 中的 syslog 守护进程，通过命令 journalctl 读取日志。

{% highlight text %}
# journalctl                               // 查看日志
# journalctl -b                            // 启动日志
# journalctl -f                            // 实时显示系统日志
# journalctl /usr/sbin/dnsmasq             // 查看特定任务的日志
{% endhighlight %}

### 电源管理

systemctl 命令也可以用来关机、重启、挂起、休眠。

{% highlight text %}
# systemctl poweroff
# systemctl reboot
# systemctl suspend
# systemctl hibernate
{% endhighlight %}


## 其它

systemd 带来了一整套与操作系统交互的新途径，如可以用 hostnamectl 获得机器的 hostname 和其它有用的独特信息。

{% highlight text %}
# hostnamectl                              // 查看机器信息
{% endhighlight %}

除了 restart 命令，也可以使用 try-start 选项，它只会在服务已经在运行中的时候重启服务。


### 对比

![SystemD VS. SysVinit]({{ site.url }}/images/linux/systemd-sysvinit.jpg "SystemD VS. SysVinit"){: .pull-center width="90%"}

<!--

在sysvinit的时代，如果需要结束一个服务及其启动的所有进程，可能会遇到一些糟糕的进程无法正确结束掉，即便是我们使用kill，killall等命令来结束它们，到了systemd的时代一切都变得不一样，systemd号称是第一个能正确的终结一项服务的程序，下面来看看具体如何操作的：
# systemctl kill crond.service
或者指定一个信号发送出去
# systemctl kill -s SIGKILL crond.service
例如可以像这样执行一次reload操作
# systemctl kill -s HUP --kill-who=main crond.service

http://www.csdn.net/article/2015-01-08/2823477/1<br><br>
http://www.linuxidc.com/Linux/2014-11/110023.htm<br><br>
http://www.linuxidc.com/Linux/2014-12/110777.htm<br><br>
http://www.ithov.com/linux/136324.shtml  比较详细介绍了原理

首先，使用systemctl start [服务名（也是文件名）]可测试服务是否可以成功运行，如果不能运行则可以使用systemctl status [服务名（也是文件名）]查看错误信息和其他服务信息，然后根据报错进行修改，直到可以start，如果不放心还可以测试restart和stop命令。

接着，只要使用systemctl enable xxxxx就可以将所编写的服务添加至开机启动即可。

这样看来，虽然systemctl比较陌生，但是其实比init.d那种方式简单不少，而且使用简单，systemctl能简化的操作还有很多，现在也有不少的资料，看来RHEL/CentOS比其他的Linux发行版还是比较先进的，此次更新也终于舍弃了Linux 2.6内核，无论是速度还是稳定性都提升不少。
-->


# 参考

[systemd System and Service Manager](http://www.freedesktop.org/wiki/Software/systemd/)，system daemon 官方网站。


{% highlight text %}
{% endhighlight %}
