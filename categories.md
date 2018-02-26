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

<!--
于编程语言、Web、云计算、Linux 相关的漫画
http://turnoff.us/
-->

#### Network

对与内核中网络部分的介绍。

* [Linux 网络协议栈简介](/post/network-introduce.html)，简单介绍一下与 Linux 协议栈相关的内容。
* [Linux 网络设置](/post/network-setting.html)，Linux 中一些常见的网络设置。
* [Netstat VS. ss](/post/network-nettools-vs-iproute2.html)，netstat 和 ss 命令是比较典型的网络监控工具，在此介绍对比下。
* [Linux 常用网络命令](/post/network-commands.html)，简单记录常用网络命令，如 tcpdump、netcat 等。
* [Linux 网络常见监控项以及报错](/post/linux-monitor-network.html)，与网络相关的调试、查看方法，当然也包括了报错相关的内容。
* [Linux 通讯协议](/post/network-protocols.html)，简单记录下 Linux 常见的通讯协议，如 SNMP 协议。
* [Linux 的防火墙](/post/network-netfilter-iptables.html)，Linux 中的防火墙策略，包括 netfilter 和 iptables 。
* [TCP/IP 协议简介](/post/network-tcpip-introduce.html)，简单介绍一下与 Linux 协议栈相关的内容。
* [TCP/IP 协议之 TIME_WAIT](/post/network-tcpip-timewait.html)，简单介绍下 TCP 协议栈中，TIME_WAIT 这一特殊的状态值。

<!--
* [Linux 中的 socketfs](/post/network-socketfs.html)，也就是 Linux 中应用层与内核网络协议栈之间的中间层。
* [TCP/IP 简介之一](/post/network-tcpip-introduce-1.html)
* [TCP/IP 简介之二](/post/network-tcpip-introduce-2.html)
* [TCP/IP 之 timestamp 选项](/post/network-tcpip-timestamp.html)
* [Linux 网络超时与重传](/post/network-timeout-retries.html)，主要介绍TCP的三次握手、数据传输、链接关闭阶段都有响应的重传机制。
* [Linux IP 隧道技术](/post/network-ip-tunneling.html)，说明下网络协议栈是如何实现隧道的，实际上就是将不同协议进行封装。
* [Linux Wireshark](/post/network-wireshark.html)，介绍 Linux 中的 Wireshark 使用方式。
-->

#### Container

实际上现在很火的 Docker 的底层是基于容器的，这部分也比较复杂，所以就单独摘出来。

* [Linux Chroot](/post/linux-chroot.html)，这实际上是做目录隔离的方法，也是最初的一种方式。
* [LXC 简介](/post/linux-container-lxc-introduce.html)，对 Linux Container 的简单介绍，包括如何安装、新建、启动容器等操作。
* [LXC sshd 单进程启动](/post/linux-container-lxc-sshd.html)，介绍如何启动一个单进程，对于资源隔离有很大的参考意义。
* [容器之 CGroup](/post/linux-container-cgroup-introduce.html)，介绍 CentOS 中 systemd 以及 cgroup-tools 相关的内容。

<!--
* [LXC 网络设置相关](/post/linux-container-lxc-network.html)，关于 Container 中网络的介绍，主要介绍 veth、vlan、macvlan 等概念。

* [Bootstrap](/post/bootstrap-etc.html)，一个来自 Twitter 的前端框架，同时包括了一些 css、javascript 相关的内容介绍。
-->

#### WebServer

Nginx 一款轻量级且高性能的 Web 服务器、反向代理服务器，通过 C 语言编写；另外，还包括了前端相关的内容。

* [Nginx 入门](/post/nginx-introduce.html)，介绍一些常见的操作，例如安装、启动、设置等。
* [Nginx 监控](/post/nginx-monitor.html)，关于 Nginx 的简单监控内容。
* [Nginx 源码解析](/post/nginx-sourecode-analyze.html)，介绍主要的代码实现过程。
* [Nginx 通用网关](/post/nginx-cgi-introduce.html)，与 CGI 相关的内容，以及部分的实现。
* [Nginx 日志解析](/post/nginx-logs-introduce.html)，简单介绍 Nginx 中的日志，以及原子写入的简介。
* [Nginx https](/post/nginx-https.html)，简单介绍如何使用 Nginx 搭建 https 服务。
* [HTTP 协议简介](/post/network-http-introduce.html)，简单介绍下 HTTP 内容以及其演变过程。
* [HTTPS 协议详解](/post/https-introduce.html)，简单介绍下 HTTPS 协议是如何实现的

<!--
 post/encryption-introduce.html linux-aio.html
-->

#### Monitor

记录与监控相关的内容。

* [Linux 监控](/post/linux-monitor.html)，简单记录一下在 Linux 监控中一些比较常见的工具、网站、资料等信息。
* [Linux 监控之 CPU](/post/linux-monitor-cpu.html)，简单介绍下 Linux 中与 CPU 监控相关的内容。
* [Linux 监控之 Memory](/post/linux-monitor-memory.html)，简单介绍下 Linux 中与 Memory 监控相关的内容。
* [Linux 监控之 IO](/post/linux-monitor-io.html)，简单介绍下 Linux 中与 IO 监控、测试相关的内容。
* [Linux 监控杂项](/post/linux-monitor-misc.html)，简单列举一些常见的监控工具 (sar, top)，以及配置方式等。
* [Dstat 使用及其原理](/post/details-about-dstat.html)，一个使用 Python 编写的跨平台监控工具。
* [Systemtap](/post/linux-systemtap.html)，介绍内核神器 Systemtap 的使用方式，包括了如何使用最新的安全特性。
* [Collectd 简介](/post/collectd-introduce.html)，一个 C 语言编写的多线程监控采集程序，对其进行简单的介绍。
* [Collectd 源码解析](/post/collectd-source-code.html)，详细介绍 Collectd 的源码实现。

#### Kernel

介绍下 Linux 内核相关的内容。

* [Linux 内核编译](/post/kernel-compile.html)，简单介绍如何手动编译内核。
* [Linux 硬件启动](/post/kernel-hardware-startup.html)，从内核加电之后，简单介绍如何从硬件加载启动。
* [Linux 启动过程](/post/kernel-bootstrap.html)，通过 GRUB 启动之后，然后开始加载内核，再次简单介绍。
* [Linux 内核模块](/post/kernel-modules.html)，简单介绍下 Linux 中的内核模块编写，包括了内核签名机制的配置。
* [Linux 系统调用](/post/kernel-syscall.html)，简单介绍下 Linux 中系统调用相关的内容。
* [Linux 调度系统](/post/linux-kernel-scheduler.html)，与内核的进程调度相关的内容。
* [Linux 进程相关](/post/linux-kernel-process.html)，简单介绍进程相关的东西，如进程创建、优先级、进程之间的关系等。
* [Linux IO 多路复用](/post/linux-program-io-multiplexing.html)，通过 IO 复用，可以有效提高程序的效率，增加吞吐。
* [Linux 共享内存](/post/linux-program-shared-memory.html)，Linux 中与共享内存相关的内容。
* [Linux 物理映射](/post/kernel-memory-virtual-physical-map.html)，x86 中逻辑地址到物理地址的映射关系，包括了具体的实验。
* [Linux 映射文件](/post/kernel-memory-mmap-introduce.html)，也就是 mmap() 函数的使用方法。
* [Linux 内存-用户空间](/post/kernel-memory-management-from-userspace-view.html)，用户空间的内存管理，包括了内存的布局、内存申请等操作。
* [Linux 内存-内核空间](/post/kernel-memory-management-from-kernel-view.html)，包括了内核中与内存相关内容，包括了初始化、内存分配等。
* [Kernel 内存杂项](/post/kernel-memory-tips.html)，简单介绍下内核中与内存相关的内容，以及常见的故障处理。

#### C Program

简单介绍下与 C 语言相关的内容。

* [C 持续集成](/post/program-c-continuous-integration.html)，一些与 C 语言的持续集成相关的工具集。
* [libev 事件库](/post/linux-libev.html)，一个 C 编写的高性能事件循环库，类似库还有 libevent、libubox 等。
* [libev 时间处理](/post/linux-libev-timers.html)，简单介绍下 libev 库中与时间相关的内容。
* [C 语言的字符串](/post/program-c-string-stuff.html)，简单介绍下 C 语言中与字符串、内存操作相关的函数。
* [C 语言的奇技淫巧](/post/program-c-tips.html)，整理下 C 语言中常用的技巧。
* [C 编译链接](/post/program-c-complie-link.html)，与 C 语言相关的编译链接概念
* [C 加载过程](/post/program-c-load-process.html)，通过动态库可以减小空间，提高效率，这里简单介绍加载过程。
* [Linux 线程编程](/post/program-c-linux-pthreads.html)，简单介绍下 Linux 中与线程相关的编程。
* [Linux 线程同步](/post/program-c-linux-pthreads-synchronize.html)，线程编程时经常使用的同步方式，如锁、条件变量、信号量等。
* [Linux 时间函数](/post/linux-timer-functions.html)，简单介绍下 Linux 中与时间相关的函数。
* [Linux IO 多路复用](/post/linux-program-io-multiplexing.html)，通过 IO 多路复用提高系统性能，包括了 select、poll、epoll 。
* [Linux AIO](/post/linux-program-aio.html)，简单介绍下 Linux 平台下的异步读写模型。

#### Java Program

简单介绍下与 Java 语言相关的内容。

* [Java 环境搭建](/post/java-environment.html)，简单记录一下 Linux 下的 Java 环境搭建，以及使用方法。
* [Java JDBC 驱动介绍](/post/java-jdbc-introduce.html)，先看 JDBC 的使用方法，然后看看其具体的实现原理
* [Java C 程序调用](/post/program-c-java.html)，简单介绍下 Java 和 C 程序的相互调用。

#### 安全

主要是 Linux 下与安全相关的内容。

* [加密算法简介](/post/security-encryption-introduce.html)，简单介绍一些常见的加密算法等。
* [PGP 简介](/post/security-pgp-introduce.html)，一个基于公钥加密体系的加密软件。
* [Linux 密码管理](/post/security-how-to-save-password.html)，简单介绍下 Linux 中的密码管理。
* [SELinux 简介](/post/linux-selinux-introduce.html)，一种强制存取控制的实现。

#### Miscellaneous

简单记录一些乱七八糟的东西。

* [Linux 用户管理](/post/linux-user-management.html)，简单介绍 Linux 用户管理相关的内容。
* [CentOS 安装与配置](/post/centos-config-from-scratch.html)，简单介绍 CentOS 在安装时需要作的一些常用配置。
* [Linux 自动编译](/post/linux-package.html)，简单介绍 Linux 下的自动编译工具，如 Makefile、Autotools 等。
* [RPM 包制作](/post/linux-create-rpm-package.html)，如何在 CentOS 中创建 RPM 包。
* [Linux 常用技巧](/post/linux-tips.html)，简单记录了一些在 Linux 中常用的技巧。
* [Linux 常用命令 \-\- 文本处理](/post/linux-commands-text.html)，简单介绍下 Linux 常用的文本处理方式。
* [Linux 常用命令 \-\- 杂项](/post/linux-commands-tips.html)，常用命令，如 find 。
* [TMUX 简介](/post/tmux-introduce.html)，一个终端复用工具，类似 screen 但是更加方便使用，更加高端。
* [Linux 绘图工具](/post/linux-gnuplot.html)，这是一个命令行驱动的绘图工具，支持多个平台。
* [Bash 相关内容](/post/linux-bash-related-stuff.html)，一些与 Bash 相关的内容，如命令执行顺序、配置文件等。
* [Bash 重定向](/post/linux-bash-redirect-details.html)，简单介绍 Bash 中 IO 重定相关的内容，包括其使用方法。
* [Logrotate 使用方法](/post/logrotate-usage.html)，一个不错的日志切割管理程序。
* [Linux 后台服务管理](/post/linux-daemon-tools.html)，介绍目前常用的后台服务管理。
* [你所不知道的定时任务](/post/details-about-cronie.html)，也就是 Linux 中如何使用 crontab，以及常见错误。
* [Linux 时间同步](/post/linux-sync-time.html)，介绍部分与时间相关的概念，例如时区、闰秒、夏令时、NTP 等。
* [Systemd 使用简介](/post/linux-systemd.html)，一般新发行版本采用的是 systemd，在此简单介绍下。
* [Rsync & Inotify](/post/rsync-inotify.html)，通过这两个命令可以快速实现文件的同步。
* [VIM 使用简介](/post/linux-vim-introduce.html)，一个功能强大、高度可定制的文本编辑器，简单介绍。

<!--
* [Bash 安全编程](/post/linux-bash-pitfalls_init.html)，
-->

----------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------

![MySQL Logo]({{ site.url }}/images/databases/mysql/mysql-mariadb-percona-logo.png "MySQL Logo"){: .pull-center}

MySQL 是一款最流行的开源关系型数据库，最初由瑞典的 MySQL AB 公司开发，目前已被 Oracle 收购，现在比较流行的开源分支包括了 MariaDB 和 Percona。

其中 MariaDB 由 MySQL 创始人 Michael Widenius 主导开发，主要原因之一是：Oracle 收购了 MySQL 后，有将 MySQL 闭源的潜在风险，因此社区采用分支的方式来避开这个风险。为了与原 MySQL 区分，不再使用原来的版本号，而是采用新的 10.0。

Percona 是最接近官方 MySQL Enterprise 发行版的版本，也就是说它提供了一些 MySQL 企业版采用的功能，并且包括了一些比较好用的常用工具。其中的缺点是，为了确保对产品中所包含功能的控制，他们自己管理代码，并不接受社区开发人员的贡献。

#### 文章

* [MySQL 写在开头](/post/mysql-begin.html)，主要保存一些经常使用的 MySQL 资源。
* [MySQL 简单介绍](/post/mysql-introduce.html)，简单介绍 MySQL 常见的使用方法，包括安装启动、客户端使用、调试等。
* [MySQL 基本概念](/post/mysql-basic.html)，介绍 MySQL 中一些基本的概念，包括了 SQL、JOIN、常见测试库等。
* [MySQL 监控指标](/post/mysql-monitor.html)，包括了一些 MySQL 常见的监控指标及其含义等。
* [MySQL Handler 监控](/post/mysql-handler.html)，主要介绍下监控指标中与 handler_read_* 相关的内容。
* [MySQL 用户管理](/post/mysql-users.html)，一些用户相关的操作，包括了用户管理、授权、密码恢复等。
* [MySQL 通讯协议](/post/mysql-protocol.html)，简单介绍 MySQL 的服务器与客户端是如何进行通讯的。
* [MySQL 语法解析](/post/mysql-parser.html)，SQL 的处理过程包括了词法分析、语法分析、语义分析、构造执行树等。
* [MySQL 插件详解](/post/mysql-plugin.html)，关于 MySQL 中一些插件功能的实现，主要是一些通用插件的介绍。
* [MySQL 存储引擎](/post/mysql-storage-engine-plugin.html)，介绍下与存储引擎相关的内容，包括了提供的接口，实现方法等。
* [MySQL 线上部署](/post/mysql-deploy-online.html)，简单记录一些线上部署时常见的配置内容。
* [MySQL 执行简介](/post/mysql-executor.html)，简单介绍 MySQL 中的查询最终是如何执行的。
* [MySQL 自带工具](/post/mysql-tools-internal.html)，简单介绍下 MySQL 中自带的工具集。
* [MySQL 常用工具](/post/mysql-tools.html)，一些运维过程中常见的三方工具，包括压测工具。
* [MySQL Sandbox](/post/mysql-sandbox.html)，本地搭建多个 MySQL 实例的工具，包括主备、循环复制、一主多备等等。
* [MySQL 启动脚本](/post/mysql-mysqld-safe.html)，详细介绍下 mysqld_safe 脚本的执行流程。
* [MySQL Core 文件](/post/mysql-core-file.html)，一些关于 CoreDump 文件以及 debuginfo 的介绍。
* [MySQL 关闭过程](/post/mysql-shutdown.html)，简单分析下 mysqld 进程关闭的过程，以及关闭过程中执行的操作。
* [MySQL 杂项](/post/mysql-tips.html)，简单记录下 MySQL 常见的一些操作。

#### 高可用

* [MySQL 日志相关](/post/mysql-log.html)，一些常见的日志介绍，同时也包括了 binlog 的详细介绍。
* [MySQL 组提交](/post/mysql-group-commit.html)，主要是关于 binlog 的组提交实现，介绍各个阶段的实现原理。
* [MySQL 复制方式](/post/mysql-replication.html)，数据复制方法，也是一些高可用解决方案的基础，介绍概念、配置方式。
* [MySQL 复制源码解析](/post/mysql-replication-sourcecode.html)，从源码的角度看看复制是如何执行的。
* [MySQL 半同步复制](/post/mysql-semisync.html)，关于半同步复制的详细解析，包括了源码的实现方式。
* [MySQL GTID 简介](/post/mysql-gtid.html)，主要介绍下 GTID 配置、实现方式，有那些限制，运维场景等。
* [MySQL Crash-Safe 复制](/post/mysql-crash-safe-replication.html)，在主备复制时，如何保证数据的一致性，当然主要是备库。
* [MySQL 主备数据校验](/post/mysql-replication-pt-table-checksum.html)，由于各种原因，主从架构可能会出现数据不一致，简单介绍校验方式。
* [MySQL 高可用 MHA](/post/mysql-replication-mha.html)，相对成熟的方案，能做到30秒内自动故障切换，且尽可能保证数据一致性。
* [MySQL 组复制](/post/mysql-group-replication.html)，也就是基于 Paxos 协议变体实现，提供了一种高可用、强一致的实现。

#### InnoDB

* [InnoDB 简单介绍](/post/mysql-innodb-introduce.html)，介绍一下与 InnoDB 相关的资料。
* [InnoDB 隔离级别](/post/mysql-innodb-isolation-level.html)，主要介绍下 InnoDB 中如何使用事务的隔离级别。
* [InnoDB Redo Log](/post/mysql-innodb-redo-log.html)，介绍 redo log 相关。
* [InnoDB Checkpoint](/post/mysql-innodb-checkpoint.html)，介绍 InnoDB 中与 checkpoint 相关内容，包括如何写入、何时写入。
* [InnoDB Double Write Buffer](/post/mysql-innodb-double-write-buffer.html)，介绍为什么会有 dblwr 机制，以及 InnoDB 中如何实现。
* [InnoDB 崩溃恢复](/post/mysql-innodb-crash-recovery.html)，简单介绍下在服务器启动的时候执行崩溃恢复的流程。


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

* [InnoDB 线程](/post/mysql-innodb-threads.html)，介绍下 InnoDB 中与线程相关的资料。
* [InnoDB Buffer Pool](/post/mysql-innodb-buffer-pool.html)，
* [InnoDB Insert Buffer](/post/mysql-innodb-insert-buffer.html)，
-->

----------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------

![PostgreSQL Logo]({{ site.url }}/images/databases/postgresql/postgresql-logo.png "PostgreSQL Logo"){: .pull-center width="521"}

PostgreSQL 可以说是目前功能最强大、特性最丰富和结构最复杂的开源数据库管理系统，其中有些特性甚至连商业数据库都不具备。这个起源于加州大学伯克利分校的数据库，现已成为一项国际开发项目，并且拥有广泛的用户群，尤其是在海外，目前国内使用者也越来越多。

PostgreSQL 基本上见证了数据库理论和技术的发展历程，由 UCB 计算机教授 Michael Stonebraker 于 1986 年创建。在此之前，Stonebraker 教授主导了关系数据库 Ingres 研究项目，88 年，提出了 Postgres 的第一个原型设计。

MySQL 号称是使用最广泛的开源数据库，而 PG 则被称为功能最强大的开源数据库。

#### 文章

* [PostgreSQL 简单介绍](/post/postgresql-introduce.html)，简单介绍 PG 的使用方法，如安装启动、客户端使用、调试等。
* [PostgreSQL 结构及权限](/post/postgresql-structure-privileges.html)，介绍 PG 中常见的概念，如 Schema、Database 等，以及权限管理。
* [PostgreSQL C 语言编程](/post/postgresql-c-language-pgcenter.html)，PG 通过 libpq 库提供 C 语言对外的 API 接口，这里简单介绍使用方法。

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
* [Python 的垃圾回收](/post/python-garbage-collection.html)，详细介绍 Python 特有的垃圾回收机制。
* [Python 动态执行](/post/python-eval.html)，允许通过 exec 和 eval 执行以字符串形式表示的代码片段，这里简单介绍。
* [Python 杂项](/post/python-tips.html)，记录了 Python 中常见技巧，一些乱七八糟的东西。

<!--
* [Python 异常处理](/post/python-exception.html)，介绍如何处理 Python 的异常。
-->

#### Flask

一个使用 Python 编写的轻量级 Web 应用框架，采用 BSD 授权。

* [Flask 简介](/post/flask-introduce.html)，简单介绍 flask 的安装、配置、使用，常用的三方模块等。
* [Flask 常见示例](/post/flask-tips.html)，包括了 Flask 中的一些常见示例，可以作为参考使用。
* [Nginx uWSGI Flask](/post/nginx-uwsgi-flask.html)，这里简单介绍如何通过 Nginx + uWSGI 搭建 Flask 运行环境。
* [Flask 请求处理流程](/post/flask-request-process.html)，介绍一次请求所经过的处理过程。
* [Flask 上下文理解](/post/flask-context.html)，主要介绍上下文、session 的使用以及源码的实现。
* [Flask 路由控制](/post/flask-route.html)，介绍 flask 中 URL 是如何进行路由的。
* [Flask 单元测试](/post/flask-unittest.html)，简单介绍对 flask 进行单元测试。



<!--
* [Flask 完整例子](/post/flask-examples.html)，实际上就是 Flask 中的完整示例，包括了单元测试等相关的内容。
-->

#### Others

记录乱七八糟的东西。

* [SaltStack 简介](/post/saltstack-introduce.html)，一个轻量级的运维工具，具备配置管理、远程执行、监控等功能。
* [Ansible 简介](/post/python-ansible.html)，一个配置管理工具，无需安装服务端和客户端，只要有ssh即可，而且使用简单。
* [Python 异步任务队列](/post/python-async-queue.html)，介绍一些常用的调度系统，如APScheduler、Redis Queue、Celery等。
* [ZeroMQ 简介](/post/zeromq-introduce.html)，一个 C++ 编写的高性能分布式消息队列，非常简单好用的传输层。

----------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------

![Golang Logo]({{ site.url }}/images/go/golang-logo-3.jpg "Golang Logo"){: .pull-center width="350"}

Golang 从 2007 年末由 Robert Griesemer、Rob Pike、Ken Thompson 主持开发，后来还加入了 Ian Lance Taylor、Russ Cox 等人，最终于 2009 年 11 月开源，在 2012 年发布了稳定版本。

实际上，Golang 基于现有的技术实现，例如协程 (Coroutine)、IO 多路复用 (multiplexing)、异步 IO 等，然后在此之上进行了一些原语的封装。开始 Golang 包含了很多 C 语言代码，在 1.5 版本开始，包括运行时 (runtime)、编译器 (compiler)和连接器 (linker) 也都全部是由 Golang 所编写。

现在 Golang 的开发已经是完全开放的，并且拥有一个活跃的社区。简单来说，Golang 是一个开源、高并发、高效的编程语言，支持垃圾回收，具有很好的可伸缩性。

而且，越来越多的项目开始使用 Golang 进行开发，例如 Docker、LXD、InfluxDB、etcd 等等。另外，与 Golang 类似的高并发语言还可以参考 Rust、Elixir 。

#### 文章

* [Golang 简介](/post/golang-introduce.html)，主要介绍 Golang 的环境搭建，常用工具等。
* [Golang 如何编码？](/post/golang-how-to-write-go-code.html)，一篇官方文章的翻译，介绍如何进行编写代码。

#### InfluxDB

一个开源分布式时序、事件和指标数据库。

* [InfluxDB 简介](/post/influxdata-influxdb.html)，简单介绍常见概念，如何安装，常用操作等。

----------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------

一些杂七杂八的东西。


![Lua Logo]({{ site.url }}/images/programs/lua-logo.png "Lua Logo"){: .pull-center width="125" }

Lua 在葡萄牙语中是 “月亮” 的意思，是一个小巧的脚本语言，官方版本只包括一个精简的核心和最基本的库，使得其体积小、启动速度快，从而特别适合嵌入到其它程序里。

这里简单介绍其使用方法。

* [Lua 简介](/post/lua-introduce.html)，简单介绍常见概念，包括安装、语法规则、常用模块等。
* [Lua 协程](/post/lua-coroutine.html)，作为一种简单的语言，仍支持闭包、协程等较新的特性，简单介绍协程使用。
* [Lua 源码解析](/post/lua-sourcecode.html)，其核心代码总共才 2W 行左右，但是却实现了很多不错的特性。
* [Lua CAPI 使用](/post/lua-how-capi-works.html)，简单介绍 Lua 和 C 之间的调用，常见的概念如栈、CAPI等概念。


![OpenSSH Logo]({{ site.url }}/images/misc/openssh-logo-1.jpg "OpenSSH Logo"){: .pull-center width="150" }

OpenSSH 是 SSH (Secure SHell) 协议的免费开源实现，一种命令行的远程登录工具，使用加密的远程登录实现，可以有效保护登录及数据的安全，同时提供了安全的文件传输功能。

这里主要介绍 SSH 一些常见的操作。

* [SSH 简介](/post/ssh-introduce.html)，简单介绍 OpenSSH 相关的内容。
* [SSH 代理设置](/post/ssh-proxy.html)，关于一些常见代理设置，如本地转发、远程转发、动态转发等。
* [SSH Simplify Your Life](/post/ssh-simplify-your-life.html)，用来配置一些常见的设置，简化登陆方式。
* [SSH 杂项](/post/ssh-tips.html)，记录一些常见的示例。


![Git Logo]({{ site.url }}/images/misc/git-logo.jpg "Git Logo"){: .pull-center width="130" }

Git 是一免费、开源的分布式版本控制系统，可有效、高速的处理从很小到非常大的项目版本管理，该工具是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发，其本意是为了替换 BitKeeper 。

这里简单介绍一下常见概念及其操作。

* [Git 简明教程](/post/git-simple-guide.html)，简单介绍常见操作。
* [Git 分支管理](/post/git-branch-model.html)，主要介绍 git 的分支处理常用操作，以及比较经典的版本分支管理方式。
* [Git 使用杂项](/post/git-tips.html)，记录 git 常见的示例，可以用来作为参考使用。

## Tags

{% for category in site.categories %}
<h3 id="{{ category | first }}">{{ category | first }}</h3>
<ul>{% for post in category[1] %}<li>{{ post.date | date: "%Y-%m-%d" }} <a href="{{post.url}}">{{ post.title }}</a></li>{% endfor %}</ul>
{% endfor %}

<!--
各个语言的排名
https://www.tiobe.com/tiobe-index/

git介绍
http://marklodato.github.io/visual-git-guide/index-en.html

一个不错的网站，包含了各种书籍。
http://apprize.info/

当浏览器输入地址时发生了什么
https://github.com/alex/what-happens-when

内存的战争，不错的文章

一个web tty共享
https://tsl0922.github.io/ttyd/


1. Hesitate 犹豫不决
2. Procastination 拖延，逃避问题和懒惰
3. Never last long 三分钟热度
4. Afraid of rejection 害怕拒绝
5. Limit yourself 自我设限
6. Runaway from reality 逃避现实
7. Always find execuess 总是寻找接口
8. Fearness 恐惧
9. Refuse to learn 拒绝学习

Python 资源大全中文版
ttps://github.com/jobbole/awesome-python-cn

SQLite源码解析
http://huili.github.io/srcAnaly/selectExec.html

CVE库
https://www.cvedetails.com/
WebSockets库
https://github.com/uNetworking/uWebSockets
C++使用mysql,断线重连问题
http://www.paobuke.com/zh-cn/develop/c/pbk1821.html
蛋疼的mysql_ping()以及MYSQL_OPT_RECONNECT
https://www.felix021.com/blog/read.php?2102
CA
http://www.barretlee.com/blog/2016/04/24/detail-about-ca-and-certs/
StatsD Python上报示例
https://github.com/etsy/statsd/blob/master/examples/python_example.py
使用C写的editline库，用于替换readline()函数
https://github.com/troglobit/editline
MYSQL C使用
http://zetcode.com/db/mysqlc/
PG用户管理
http://www.davidpashley.com/articles/postgresql-user-administration/
ZeroMQ
https://github.com/anjuke/zguide-cn

-->
