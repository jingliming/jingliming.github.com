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

从 1994.3 Linux1.0 发布到现在，几乎可以说 Linux 已经成为最流行的操作系统，涉及到了服务器、桌面、嵌入式等多种场景，而且支持绝大多数平台。

鄙人从大三开始用 Linux，开始就是看着 Ubuntu 的 3D 桌面比较酷，然后开始零零散散地使用，一直到现在，几乎所有的日常操作都在使用 Linux 的桌面发行版；目前使用的是 CentOS 版本。

在此，仅介绍 Linux 相关内容，包括了常用的方法，以及相应的内核介绍。

#### Network

对与内核中网络部分的介绍。

* [Linux 网络设置](/post/network-setting.html)，Linux 中一些常见的网络设置。

<!--

* [Linux 网络协议栈简介](/post/network-introduce.html)，简单介绍一下 Linux 中网络协议栈的相关内容。
* [网络监控 netstat VS. ss](/post/network-nettools-vs-iproute2.html)，netstat 和 ss 命令是比较典型的网络监控工具，在此介绍对比下。
* [Linux 网络常见监控项以及报错](/post/network-monitor.html)，报错和监控之间的关系太紧密，就将两者合并到了一起。
* [Linux 中的 socketfs](/post/network-socketfs.html)，也就是 Linux 中应用层与内核网络协议栈之间的中间层。
* [TCP/IP 简介之一](/post/network-tcpip-introduce-1.html)
* [TCP/IP 简介之二](/post/network-tcpip-introduce-2.html)
* [TCP/IP 之 TIME_WAIT 状态](/post/network-tcpip-timewait.html)，如何处理服务器中经常出现的 TIME_WAIT 状态值。
* [TCP/IP 之 timestamp 选项](/post/network-tcpip-timestamp.html)
* [Linux 网络超时与重传](/post/network-timeout-retries.html)，主要介绍TCP的三次握手、数据传输、链接关闭阶段都有响应的重传机制。
* [Linux IP 隧道技术](/post/network-ip-tunneling.html)，说明下网络协议栈是如何实现隧道的，实际上就是将不同协议进行封装。
* [Linux Wireshark](/post/network-wireshark.html)，介绍 Linux 中的 Wireshark 使用方式。

#### Container

实际上现在很火的 Docker 的底层是基于容器的，这部分也比较复杂，所以就单独摘出来。

* [Linux Chroot](/post/linux-chroot.html)，这实际上是做目录隔离的方法，也是最初的一种方式。
* [LXC 简介](/post/linux-lxc-introduce.html)，对 Linux Container 的简单介绍，包括如何安装、新建、启动容器等操作。
* [LXC 网络设置相关](/post/linux-lxc-network.html)，关于 Container 中网络的介绍，主要介绍 veth、vlan、macvlan 等概念。
* [LXC sshd 单进程启动](/post/linux-lxc-sshd.html)，介绍如何启动一个单进程，对于资源隔离有很大的参考意义。


* [Bootstrap](/post/bootstrap-etc.html)，一个来自 Twitter 的前端框架，同时包括了一些 css、javascript 相关的内容介绍。
* [Linux 内存监控](/post/linux-monitor-memory.html)，记录下在 Linux 中，与内存相关的监控项以及工具。
-->

#### WebServer

Nginx 一款轻量级且高性能的 Web 服务器、反向代理服务器，通过 C 语言编写；另外，还包括了前端相关的内容。

* [Nginx 入门](/post/nginx-introduce.html)，介绍一些常见的操作，例如安装、启动、设置等。

#### Monitor

记录与监控相关的内容。

* [Linux 监控](/post/linux-monitor.html)，简单记录一下在 Linux 监控中一些比较常见的工具、网站、资料等信息。
* [Dstat 使用及其原理](/post/details-about-dstat.html)，一个使用 Python 编写的跨平台监控工具。
* [Systemtap](/post/linux-systemtap.html)，介绍内核神器 Systemtap 的使用方式，包括了如何使用最新的安全特性。

#### SSH

主要介绍 SSH 一些常见的操作。

<!-- * [SSH 简介](/post/ssh-introduce.html)，简单介绍 OpenSSH 相关的内容。-->
* [SSH 代理设置](/post/ssh-proxy.html)，关于一些常见代理设置，如本地转发、远程转发、动态转发等。
* [SSH Simplify Your Life](/post/ssh-simplify-your-life.html)，用来配置一些常见的设置，简化登陆方式。
* [SSH 杂项](/post/ssh-tips.html)，记录一些常见的示例。



#### Miscellaneous

简单记录一些乱七八糟的东西。

* [CentOS 安装与配置](/post/centos-config-from-scratch.html)，简单介绍 CentOS 在安装时需要作的一些常用配置。
* [RPM 包制作](/post/linux-create-rpm-package.html)，如何在 CentOS 中创建 RPM 包。
* [Linux 常用技巧](/post/linux-tips.html)，简单记录了一些在 Linux 中常用的技巧。
* [TMUX](/post/tmux.html)，一个终端复用工具，类似 screen 但是更加方便使用，更加高端。
* [Linux 绘图工具](/post/linux-gnuplot.html)，这是一个命令行驱动的绘图工具，支持多个平台。
* [你所不知道的定时任务](/post/details-about-cronie.html)，也就是 Linux 中如何使用 crontab，以及常见错误。
* [Git 分支管理](/post/git-branch-model.html)，主要介绍 git 的分支处理常用操作，以及比较经典的版本分支管理方式。

<!--
* [YUM 使用](/post/yum-usage.html)，包括了如何搭建私有镜像、定制 RPM 包等。
* [Linux System Daemon](/post/linux-systemd.html)，一般新发行版本采用的是 systemd，在此简单介绍下。
-->

----------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------

![MySQL Logo]({{ site.url }}/images/databases/mysql/mysql-mariadb-percona-logo.png "MySQL Logo"){: .pull-center}

MySQL 是一款最流行的开源关系型数据库，最初由瑞典的 MySQL AB 公司开发，目前已被 Oracle 收购，现在比较流行的开源分支包括了 MariaDB 和 Percona。

其中 MariaDB 由 MySQL 创始人 Michael Widenius 主导开发，主要原因之一是：Oracle 收购了 MySQL 后，有将 MySQL 闭源的潜在风险，因此社区采用分支的方式来避开这个风险。为了与原 MySQL 区分，不再使用原来的版本号，而是采用新的 10.0。

Percona 是最接近官方 MySQL Enterprise 发行版的版本，也就是说它提供了一些 MySQL 企业版采用的功能，并且包括了一些比较好用的常用工具。其中的缺点是，为了确保对产品中所包含功能的控制，他们自己管理代码，并不接受社区开发人员的贡献。

文章列表：

* [MySQL 写在开头](/post/mysql-begin.html)，主要保存一些经常使用的 MySQL 资源。
* [MySQL 简单介绍](/post/mysql-introduce.html)，简单介绍 MySQL 常见的使用方法，包括安装启动、客户端使用、调试等。
* [MySQL 基本概念](/post/mysql-basic.html)，介绍 MySQL 中一些基本的概念，包括了 SQL、JOIN、常见测试库等。
* [MySQL 监控指标](/post/mysql-monitor.html)，包括了一些 MySQL 常见的监控指标及其含义等。
* [MySQL 用户管理](/post/mysql-users.html)，一些用户相关的操作，包括了用户管理、授权、密码恢复等。
* [MySQL 插件详解](/post/mysql-plugin.html)，关于 MySQL 中一些插件功能的实现，主要是一些通用插件的介绍。
* [MySQL 存储引擎](/post/mysql-storage-engine-plugin.html)，介绍下与存储引擎相关的内容，包括了提供的接口，实现方法等。
* [MySQL 线上部署](/post/mysql-deploy-online.html)，简单记录一些线上部署时常见的配置内容。
* [MySQL 执行简介](/post/mysql-executor.html)，简单介绍 MySQL 中的查询最终是如何执行的。
* [MySQL 自带工具](/post/mysql-tools-internal.html)，简单介绍下 MySQL 中自带的工具集。
* [MySQL 常用工具](/post/mysql-tools.html)，一些运维过程中常见的三方工具，包括压测工具。
* [MySQL 杂项](/post/mysql-tips.html)，简单记录下 MySQL 常见的一些操作。

高可用:

* [MySQL 日志相关](/post/mysql-log.html)，一些常见的日志介绍，同时也包括了 binlog 的详细介绍。
* [MySQL 组提交](/post/mysql-group-commit.html)，主要是关于 binlog 的组提交实现，介绍各个阶段的实现原理。
* [MySQL 复制方式](/post/mysql-replication.html)，数据复制方法，也是一些高可用解决方案的基础，介绍概念、配置方式。
* [MySQL 复制源码解析](/post/mysql-replication-sourcecode.html)，从源码的角度看看复制是如何执行的。
* [MySQL 半同步复制](/post/mysql-semisync.html)，关于半同步复制的详细解析，包括了源码的实现方式。
* [MySQL Crash-Safe 复制](/post/mysql-crash-safe-replication.html)，在主备复制时，如何保证数据的一致性，当然主要是备库。
* [MySQL 组复制](/post/mysql-group-replication.html)，也就是基于 Paxos 协议变体实现，提供了一种高可用、强一致的实现。

InnoDB:

* [InnoDB 隔离级别](/post/mysql-innodb-isolation-level.html)，主要介绍下 InnoDB 中如何使用事务的隔离级别。


<!--
* [MySQL 配置文件](/post/mysql-config.html)，关于配置相关的内容。
* [MySQL 链接方式](/post/mysql-connection.html)，实际上就是线程与链接的处理方式，主要包括了三种。
* [MySQL Handler 监控](/post/mysql-handler.html)，实际上时监控中的 handler 相关的内容。
* [MySQL MyISAM](/post/mysql-myisam.html)，关于 MySQL 中经典的 MyISAM 的介绍。
* [MySQL 代码导读](/post/mysql-skeleton.html)，也就是代码脉络的大致导读。
* [MySQL 事务处理](/post/mysql-transaction.html)，也就是 MySQL 中的事务处理方法。
* [MySQL 存储引擎](/post/mysql-storage-engine-plugin.html)，实际是插件的一个特例，不过使用比较复杂，所以就单独作为一篇。
* [MySQL 备份](/post/mysql-backup.html)，介绍 MySQL 一些常见的备份方法。
* [MySQL 高可用](/post/mysql-high-availability.html)，介绍 MySQL 中的常用高可用解决方案。
* [MySQL 安全设置](/post/mysql-security.html)，也就是一些对 MySQL 进行加固的方法。

InnoDB:

* [InnoDB 简介](/post/mysql-innodb-introduce.html)，介绍一下与 InnoDB 相关的资料。
* [InnoDB 线程](/post/mysql-innodb-threads.html)，介绍下 InnoDB 中与线程相关的资料。
* [InnoDB Buffer Pool](/post/mysql-innodb-buffer-pool.html)，
* [InnoDB Insert Buffer](/post/mysql-innodb-insert-buffer.html)，
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

* [Python 模块简介](/post/python-modules.html)，简单介绍一下 Python 中的模块，以及一些常用的模块。
* [Python 杂项](/post/python-tips.html)，记录了 Python 中常见技巧，一些乱七八糟的东西。

<!--
* [Python 的垃圾回收机制](/post/python-garbage-collection.html)，详细介绍 Python 特有的垃圾回收机制。
* [Python 异常处理](/post/python-exception.html)，介绍如何处理 Python 的异常。
* [Python Greenlet](/post/python-greenlet.html)，
* [Python Gevent](/post/python-gevent.html)，
-->

#### Flask

一个使用 Python 编写的轻量级 Web 应用框架，采用 BSD 授权。

* [Flask 简介](/post/flask-introduce.html)，简单介绍 flask 的安装、配置、使用，常用的三方模块等。
* [Flask 常见示例](/post/flask-tips.html)，包括了 Flask 中的一些常见示例，可以作为参考使用。

<!--
* [Flask 请求处理流程](/post/flask-request-process.html)，介绍一次请求所经过的处理过程。
* [Flask 上下文理解](/post/flask-context.html)，主要介绍上下文、session 的使用以及源码的实现。
* [Flask 路由控制](/post/flask-route.html)，介绍 flask 中 URL 是如何进行路由的。
* [Flask 单元测试](/post/flask-unittest.html)，简单介绍对 flask 进行单元测试。
* [Flask 完整例子](/post/flask-examples.html)，实际上就是 Flask 中的完整示例，包括了单元测试等相关的内容。
-->


<!--
http://marklodato.github.io/visual-git-guide/index-en.html
-->

## Tags

{% for category in site.categories %}
<h3 id="{{ category | first }}">{{ category | first }}</h3>
<ul>{% for post in category[1] %}<li>{{ post.date | date: "%Y-%m-%d" }} <a href="{{post.url}}">{{ post.title }}</a></li>{% endfor %}</ul>
{% endfor %}
