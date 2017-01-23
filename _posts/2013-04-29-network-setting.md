---
title: Linux 网络设置
layout: post
comments: true
category: [linux, network]
language: chinese
---

主要记录下在 Linux 中，一些常见的网络配置，例如 IP 地址、路由、MAC 地址、主机名等设置方式。

<!-- more -->









## 常见配置

简单列举一些常见的网络配置。

{% highlight text %}
# ifconfig eth0 down                                   ← 关闭网络设备
# ifconfig eth0 hw ether xx:xx:xx:xx:xx:xx             ← 设置MAC地址
# ifconfig eht0 up
# ifconfig eth0 210.72.137.115 netmask 255.255.255.0   ← 配置IP
# route add default gw 192.168.0.100 dev eth0          ← 添加默认路由
{% endhighlight %}

如果 IP 和 GW 不在同一网段会出现 SIOCADDRT:No such process. 的错误，可以在上一条命令之前添加如下命令。

{% highlight text %}
# route add -host 192.168.0.100 [netmask 0.0.0.0] dev eth0
{% endhighlight %}

对于 ARP 可以通过如下方式查看。

{% highlight text %}
# arp -n                                               ← 查看当前系统的ARP表
# arp -s 10.15.72.25 408:d0:9f:e2:4d:c3                ← 网关绑定ip和网关的mac
# arp -d 10.15.72.73                                   ← 删除arp记录
{% endhighlight %}

另外，是和无线网络相关的命令。

{% highlight text %}
# ifconfig eth0 promisc                                ← 设置混杂模式
# ifconfig eth0 -promisc                               ← 取消混杂模式
{% endhighlight %}

### 常见文件

在 Linux 中，包括了一些常见的系统文件，简单列举如下：

* /etc/protocols 主机使用的协议以及各个协议的协议号；
* /etc/services 主机的不同端口的网络服务。

#### /etc/networks

另外这个文件主要完成域名与网络地址的映射，其内容如下，在此简单介绍下。

{% highlight text %}
default 0.0.0.0
loopback 127.0.0.0

foobarname 192.168.1.0
{% endhighlight %}

其中的网络地址是以 .0 结尾，而且只支持 Class A, B, C 网络，不能直接指定 IP 地址，对于 IP 应该保存在 /etc/hosts 文件中。

例如，上述的最后一条记录，假设有 IP 地址为 192.168.1.5，那么查看路由时会显示如下内容。

{% highlight text %}
$ route
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.1.1     0.0.0.0         UG    0      0        0 eth0
foobarname      *               255.255.255.0   U     0      0        0 eth0
{% endhighlight %}







## hostname 修改

hostname 是 Linux 系统下的一个内核参数，可以通过 /proc/sys/kernel/hostname 访问该参数当前值，不过其值一般是在系统启动时通过初始化脚本设置的。当然，这也就导致不同的发行版本，甚至不同的版本，其设置方式也各有区别。

常见的一些操作可以通过如下方式查看。

{% highlight text %}
----- 查看(三者值相同)
$ hostname                                     ← 查看主机名
$ hostname -i                                  ← 查看主机的IP
$ cat /proc/sys/kernel/hostname
$ sysctl kernel.hostname

----- 临时修改(重启后会失效)，立即生效
$ echo foobar > /proc/sys/kernel/hostname
$ sysctl kernel.hostname=foobar
$ hostname foobar
$ export HOSTNAME=foobar                       ← 解决bash的PS显示问题

----- 永久修改 (需要重启网络)   
$ cat /etc/sysconfig/network                   ← CentOS/RedHat
NETWORKING=yes
HOSTNAME=foobar.localdomain
$ cat /etc/hostname                            ← Debian/Ubuntu
foobar

----- 重启网络
$ /etc/init.d/network restart
{% endhighlight %}

其中，有几点需要注意的：

* 在 CentOS 中，如果在上述的配置文件中没有配置，实际也会读取 /etc/hostname 文件。查看 man hostname 帮助时，也需要同时设置 /etc/hostname 。
* 通过 strace 命令查看系统调用时，实际是调用的 uname() 这个内核 API 。

以 CentOS 为例，在系统启动时，会执行 /etc/rc.d/init.d/network 脚本，该脚本会根据不同的配置文件尝试获取 hostname 。

另外，一个比较容易混淆的文件是 /etc/hosts，该文件实际上就只提供了一个本地设置的 DNS 映射关系，只是有些启动脚本会通过该文件读取 hostname，这也就导致会认为 hostname 的修改是通过该文件配置的。 

当然，最好也设置好该文件内容。

{% highlight text %}
$ cat /etc/hosts
127.0.0.1 foobar localhost
{% endhighlight %}




{% highlight text %}
{% endhighlight %}
