---
layout: default
title: Categories
---

<style type="text/css"><!-- p {text-indent: 2em;} --></style>

## 分类

该页面开头是主要分类及其介绍，主要是可以根据分类进行查询，后面会有根据标签（tags）的全部文章的分类，和侧边相似。

----------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------

![Linux Logo]({{ site.url }}/images/linux/linux-logo.jpg "Linux Logo"){: .pull-center}

Linux 相关，包括了常用的方法，以及相应的内核介绍。

<!--

#### Network

对与内核中网络部分的介绍。

* [Linux 网络协议栈简介](/blog/network-introduce.html)，简单介绍一下 Linux 中网络协议栈的相关内容。
* [网络监控 netstat VS. ss](/blog/network-nettools-vs-iproute2.html)，netstat 和 ss 命令是比较典型的网络监控工具，在此介绍对比下。
* [Linux 网络常见监控项以及报错](/blog/network-monitor.html)，报错和监控之间的关系太紧密，就将两者合并到了一起。
* [Linux 中的 socketfs](/blog/network-socketfs.html)，也就是 Linux 中应用层与内核网络协议栈之间的中间层。
* [TCP/IP 简介之一](/blog/network-tcpip-introduce-1.html)
* [TCP/IP 简介之二](/blog/network-tcpip-introduce-2.html)
* [TCP/IP 之 TIME_WAIT 状态](/blog/network-tcpip-timewait.html)，如何处理服务器中经常出现的 TIME_WAIT 状态值。
* [TCP/IP 之 timestamp 选项](/blog/network-tcpip-timestamp.html)
* [Linux 网络超时与重传](/blog/network-timeout-retries.html)，主要介绍TCP的三次握手、数据传输、链接关闭阶段都有响应的重传机制。
* [Linux IP 隧道技术](/blog/network-ip-tunneling.html)，说明下网络协议栈是如何实现隧道的，实际上就是将不同协议进行封装。
* [Linux Wireshark](/blog/network-wireshark.html)，介绍 Linux 中的 Wireshark 使用方式。

#### Container

实际上现在很火的 Docker 的底层是基于容器的，这部分也比较复杂，所以就单独摘出来。

* [Linux Chroot](/blog/linux-chroot.html)，这实际上是做目录隔离的方法，也是最初的一种方式。
* [LXC 简介](/blog/linux-lxc-introduce.html)，对 Linux Container 的简单介绍，包括如何安装、新建、启动容器等操作。
* [LXC 网络设置相关](/blog/linux-lxc-network.html)，关于 Container 中网络的介绍，主要介绍 veth、vlan、macvlan 等概念。
* [LXC sshd 单进程启动](/blog/linux-lxc-sshd.html)，介绍如何启动一个单进程，对于资源隔离有很大的参考意义。

#### Monitor

记录与监控相关的内容。

* [Linux 监控](/blog/linux-monitor.html)，简单记录一下在 Linux 监控中一些比较常见的工具、网站、资料等信息。
* [Linux 内存监控](/blog/linux-monitor-memory.html)，记录下在 Linux 中，与内存相关的监控项以及工具。
* [Dstat 使用及其原理](/blog/details-about-dstat.html)，一个使用 Python 编写的跨平台监控工具。
* [Systemtap](/blog/linux-systemtap.html)，介绍内核神器 Systemtap 的使用方式，包括了如何使用最新的安全特性。

#### SSH

* [SSH 简介](/blog/ssh-introduce.html)，简单介绍 OpenSSH 相关的内容。
* [SSH Simplify Your Life](/blog/ssh-simplify-your-life.html)，用来配置一些常见的设置，简化登陆方式。
* [SSH 代理设置](/blog/ssh-proxy.html)，关于一些常见代理设置，如本地转发、远程转发、动态转发等。
* [SSH 杂项](/blog/ssh-tips.html)，记录一些常见的示例。


#### WebServer

Nginx 一款轻量级且高性能的 Web 服务器、反向代理服务器，通过 C 语言编写；另外，还包括了前端相关的内容。

* [Bootstrap](/blog/bootstrap-etc.html)，一个来自 Twitter 的前端框架，同时包括了一些 css、javascript 相关的内容介绍。
* [Nginx 入门](/blog/nginx-introduce.html)，介绍一些常见的操作，例如安装、启动、设置等。
-->


#### Miscellaneous

简单记录一些乱七八糟的东西。

* [CentOS 安装与配置](/blog/centos-config-from-scratch.html)，简单介绍 CentOS 在安装时需要作的一些常用配置。
* [Linux 常用技巧](/blog/linux-tips.html)，简单记录了一些在 Linux 中常用的技巧。
* [TMUX](/blog/tmux.html)，一个终端复用工具，类似 screen 但是更加方便使用，更加高端。
* [Linux 绘图工具](/blog/linux-gnuplot.html)，这是一个命令行驱动的绘图工具，支持多个平台。

<!--
* [Linux System Daemon](/blog/linux-systemd.html)，一般新发行版本采用的是 systemd，在此简单介绍下。
-->

----------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------

![MySQL Logo]({{ site.url }}/images/mysql/mysql-mariadb-percona-logo.png "MySQL Logo"){: .pull-center}

MySQL 是一款最流行的开源关系型数据库，最初由瑞典的 MySQL AB 公司开发，目前已被 Oracle 收购，现在比较流行的开源分支包括了 MariaDB 和 Percona。

其中 MariaDB 由 MySQL 创始人 Michael Widenius 主导开发，主要原因之一是：Oracle 收购了 MySQL 后，有将 MySQL 闭源的潜在风险，因此社区采用分支的方式来避开这个风险。为了与原 MySQL 区分，不再使用原来的版本号，而是采用新的 10.0。

Percona 是最接近官方 MySQL Enterprise 发行版的版本，也就是说它提供了一些 MySQL 企业版采用的功能，并且包括了一些比较好用的常用工具。其中的缺点是，为了确保对产品中所包含功能的控制，他们自己管理代码，并不接受社区开发人员的贡献。

文章列表：

* [MySQL 写在开头](/blog/mysql-begin.html)，主要保存一些经常使用的 MySQL 资源。
* [MySQL 常用工具](/blog/mysql-tools.html)，一些运维过程中常见的工具，包括压测工具。
* [MySQL 监控指标](/blog/mysql-monitor.html)，包括了一些 MySQL 常见的监控指标及其含义等。


<!--

* [MySQL 简介](/blog/mysql-introduce.html)，简单介绍 MySQL 常见的使用方法，包括安装启动、客户端使用、调试等。
* [MySQL 基本概念](/blog/mysql-basic.html)，介绍 MySQL 中一些基本的概念，包括了 SQL、JOIN、常见测试库等。
* [MySQL 配置文件](/blog/mysql-config.html)，关于配置相关的内容。
* [MySQL 用户管理](/blog/mysql-users.html)，一些用户相关的操作，包括了用户管理、授权、密码恢复等。
* [MySQL 链接方式](/blog/mysql-connection.html)，实际上就是线程与链接的处理方式，主要包括了三种。
* [MySQL Handler 监控](/blog/mysql-handler.html)，实际上时监控中的 handler 相关的内容。
* [MySQL MyISAM](/blog/mysql-myisam.html)，关于 MySQL 中经典的 MyISAM 的介绍。
* [MySQL 代码导读](/blog/mysql-skeleton.html)，也就是代码脉络的大致导读。
* [MySQL 事务处理](/blog/mysql-transaction.html)，也就是 MySQL 中的事务处理方法。
* [MySQL 插件](/blog/mysql-plugin.html)，关于 MySQL 中一些插件功能的实现，主要是一些通用插件的介绍。
* [MySQL 存储引擎](/blog/mysql-storage-engine-plugin.html)，实际是插件的一个特例，不过使用比较复杂，所以就单独作为一篇。
* [MySQL 备份](/blog/mysql-backup.html)，介绍 MySQL 一些常见的备份方法。
* [MySQL 日志](/blog/mysql-log.html)，一些常见的日志，包括了 binlog 。
* [MySQL 复制](/blog/mysql-replication.html)，MySQL 的数据复制同步方法，这通常也是一些高可用解决方案的基础。
* [MySQL 高可用](/blog/mysql-high-availability.html)，介绍 MySQL 中的常用高可用解决方案。
* [MySQL 安全设置](/blog/mysql-security.html)，也就是一些对 MySQL 进行加固的方法。

InnoDB:

* [InnoDB 简介](/blog/mysql-innodb-introduce.html)，介绍一下与 InnoDB 相关的资料。
* [InnoDB 线程](/blog/mysql-innodb-threads.html)，介绍下 InnoDB 中与线程相关的资料。
* [InnoDB Buffer Pool](/blog/mysql-innodb-buffer-pool.html)，
* [InnoDB Insert Buffer](/blog/mysql-innodb-insert-buffer.html)，
-->

----------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------

![Python Logo]({{ site.url }}/images/python/python-logo.png "Python Logo"){: .pull-center width="420"}

通常当我们讨论 Python 时，指的是 Python 语言以及 CPython 实现。而实际上 Python 只是一种语言的规范，可以根据该规范使用不同的语言去实现相应的解析器，除了 CPython 之外，常见的还有 PyPy、Jython、IronPython、MicroPython 等。

对于传统语言，如 C/C++ 等，会直接将代码编译为机器语言后运行，而对于不同的平台或者 CPU 需要重新编译才可以，而 Python 可以直接跨平台运行。

CPython 通过 C 语言实现，也是目前使用最为广泛的版本，虽然 PyPy 现在的发展势头不错，不过估计短时间内还是不会替代 CPython。CPython 也需要编译 (编译成字节码)，然后运行，其核心实际上是一个字节码解析器 (Bytecode Interpreter)，用于模拟堆栈操作，或者称之为 Virtual Stack Machines。

如果没有特殊说明的话，在此特指 CPython；另外，比较想提一下的是 MicroPython，这是一个用于微控制器的 Python 实现 ^_^

Just More Pythonic ~~~

#### CPython

记录 C 语言实现的 Python 的简介。

* [Python 模块简介](/blog/python-modules.html)，简单介绍一下 Python 中的模块，以及一些常用的模块。
* [Python 杂项](blog/python-tips.html)，记录了 Python 中常见技巧，一些乱七八糟的东西。

<!--
* [Python 的垃圾回收机制](blog/python-garbage-collection.html)，详细介绍 Python 特有的垃圾回收机制。
* [Python 异常处理](/blog/python-exception.html)，介绍如何处理 Python 的异常。
* [Python Greenlet](/blog/python-greenlet.html)，
* [Python Gevent](/blog/python-gevent.html)，
-->

#### Flask

一个使用 Python 编写的轻量级 Web 应用框架，采用 BSD 授权。

* [Flask 简介](/blog/flask-introduce.html)，简单介绍 flask 的安装、配置、使用，常用的三方模块等。
* [Flask 常见示例](/blog/flask-tips.html)，包括了 Flask 中的一些常见示例，可以作为参考使用。

<!--
* [Flask 请求处理流程](/blog/flask-request-process.html)，介绍一次请求所经过的处理过程。
* [Flask 上下文理解](/blog/flask-context.html)，主要介绍上下文、session 的使用以及源码的实现。
* [Flask 路由控制](/blog/flask-route.html)，介绍 flask 中 URL 是如何进行路由的。
* [Flask 单元测试](/blog/flask-unittest.html)，简单介绍对 flask 进行单元测试。
* [Flask 完整例子](/blog/flask-examples.html)，实际上就是 Flask 中的完整示例，包括了单元测试等相关的内容。
-->


<!--
http://marklodato.github.io/visual-git-guide/index-en.html
-->

## Tags

{% for category in site.categories %}
<h3 id="{{ category | first }}">{{ category | first }}</h3>
<ul>{% for post in category[1] %}<li>{{ post.date | date: "%Y-%m-%d" }} <a href="{{post.url}}">{{ post.title }}</a></li>{% endfor %}</ul>
{% endfor %}
