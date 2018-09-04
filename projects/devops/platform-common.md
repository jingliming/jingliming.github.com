---
title: 通用规范
layout: project
comments: true
language: chinese
category: [misc]
keywords:
description:
---

## 规范

### 变量名

这里的命名规范包括了 `Tags` 。

可用字符为 ASCII ，包括了 `A-Z` `a-z` `0-9` `_` `@` `#` 。

### 包格式

{% highlight text %}
BootAgent-1.3.1-1.x86_64
{% endhighlight %}


## 统一错误码

错误码通过 `32bits` 指定，其中高 `16bits` 用来表示分类。

### Server(Aspire)

也就是 BootAgent 的服务端，高位是 `0x10` 。

{% highlight text %}
0x1001   AgentSN冲突无法更新
{% endhighlight %}

## 安装路径

对于 Agent 来说，其安装路径如下。

{% highlight text %}
----- 配置文件
/etc/cargo/gearman.conf

----- 二进制程序
/usr/bin/gearman

----- 动态库
/usr/lib/cargo/nodus/cpu.so
/usr/lib/cargo/gearman/userinfo.so

----- 文档以及特定的配置文件等
/usr/share/cargo
/usr/share/cargo/nodus/types.db
/usr/share/doc/cargo/AUTHORS
/usr/share/doc/cargo/COPYING
/usr/share/doc/cargo/ChangeLog
/usr/share/doc/cargo/README
/usr/share/doc/cargo/TODO
/usr/share/man/man1/cargo.1.xz
/usr/share/man/man5/cargo-exec.5.xz

----- 日志以及PID文件
/var/log/cargo/cargo.log
/var/run/cargo/cargo.pid
{% endhighlight %}



{% highlight text %}
{% endhighlight %}
