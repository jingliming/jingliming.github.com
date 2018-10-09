---
title: Linux saltstack安装和使用
layout: post
comments: true
language: chinese
category: [linux,misc]
keywords: linux,op saltstack
description: SaltStack是基于Python开发的服务器基础架构集中管理平台，也称为自动化运维工具，具备远程执行、配置管理、云管理三大功能。管理端称为Master，被管理端称为Minion，Master和Minion通过密钥认证进行加密通信，通过消息队列软件ZeroMQ进行内容传输，使用的默认端口为4505和4506。本文的主要内容为SaltStack的安装配置与远程执行测试。
---

SaltStack是基于Python开发的服务器基础架构集中管理平台，也称为自动化运维工具，具备远程执行、配置管理、云管理三大功能。管理端称为Master，被管理端称为Minion，Master和Minion通过密钥认证进行加密通信，通过消息队列软件ZeroMQ进行内容传输，使用的默认端口为4505和4506。本文的主要内容为SaltStack的安装配置与远程执行测试。

<!-- more -->

## 环境准备

## 安装

安装需要分别安装master和minion，直接使用yum安装即可。

### 安装master

在 Windows 上创建 RAM DISK 需要用到第三方软件，而 Linux 只要几条命令即可。

{% highlight text %}
----- 新建RAM DISK的挂载点
$ mkdir /tmp/ramdisk
----- 查看可用内存
$ free -h
----- 创建并挂载RAM DISK
$ mount -t tmpfs -o size=128M ramdisk /tmp/ramdisk
----- 查看挂载是否成功
$ df
$ mount
{% endhighlight %}

此时创建了 128M 的 RAM DISK，文件格式为 `tmpfs`，挂载目录是 `/tmp/ramdisk` 。

### 安装minion

直接使用系统自带的 `dd` 进行写入测试。

{% highlight text %}
----- RAM DISK
$ dd if=/dev/zero of=/tmp/ramdisk/test bs=1024 count=102400 conv=fdatasync

----- 普通硬盘
$ dd if=/dev/zero of=~/test bs=1024 count=102400 conv=fdatasync
{% endhighlight %}

## 其它

{% highlight text %}
----- 卸载RAM DISK，释放内存空间
$ umount /tmp/ramdisk
----- 编辑fstab文件，设置开机启动
$ vim /etc/fstab
ramdisk /tmp/ramdisk tmpfs defaults,size=1G,x-gvfs-show 0 0
{% endhighlight %}

其中 `x-gvfs-show` 选项会在文件管理器中显示挂载的 RAM DISK 。



{% highlight text %}
{% endhighlight %}
