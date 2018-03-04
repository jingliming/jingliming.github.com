---
title: 搭建基础平台
layout: project
comments: true
language: chinese
category: [misc]
keywords: hello world,示例,sample,markdown
description: 简单记录一下一些与 Markdown 相关的内容，包括了一些使用模版。
---

<!-- more -->



## Polaris

服务端，支持 HTTP 请求。

## 基础 Gearman

### 执行命令

##### 1. 同步命令

{% highlight text %}
-----> 请求报文
{
    "id": "xxxxxx",
    "method": "sync.bash",    # 必选，指定同步命令类型
    "cmd": "ls",              # 必选，需要执行的命令
    "timeout": 10,            # 可选，超时时间，默认120，最小1s
    "user": "root",           # 可选，执行用户，默认root
    "group": "root"           # 可选，执行用户组，默认root
}

<---- 响应报文，成功
{
    "id": "xxxxxxx",
    "resp": "success",
    "retcode": 2,
    "message": "Normal termination"
    "data": "xxxx"
}
<---- 响应报文，失败
{
    "id": "xxxxxxx",
    "resp": "failed",
    "retcode": 3021,
    "message": "Invalid params"
}
{% endhighlight %}

### 常驻进程管理

主要用于管理一些子进程，包括了监控、日志、安全相关的进程管理。

##### 0. 表结构设计



##### 1. 下发任务

{% highlight text %}
{
    "id": "xxxxx",
    "method": "daemon.init",
    "name": "plugin_name",                   // 必选，任务名称 64
    "version": "1.0.1",                      // 必选，版本号 64
    "url": "http://your/monitor.rpm",        // 必选，下载安装包路径 512
    "precheck": "/your/program/path/check",  // 启动前检查命令 512
    "precheck_retries": 3,                   // 检查重试次数
    "start": "/your/program/path/start",     // 必选，启动命令 512
    "start_delay": 10,                       // 启动任务后sleep多久之后检查是否启动成功
    "stop": "/your/program/path/stop",       // 必选，停止任务 512
    "stop_timeout": 10,                      // 停止任务超时时间，超时直接kill -9
    "check": "process",                      // 存活检查方式
    "check_interval": 10,                    // 存活检查时间间隔
    "user": "user name",
    "group": "group name",
    "pidfile": "/var/run/program.pid",
    "socket": "/var/run/program.sock",
    "env": {
         "PATH":"/usr/bin:/usr/sbin"
    },
    "limits": {
        "hits": 5,
        "cpu": "5%",
        "memory": "100M"
    }
}
{% endhighlight %}


## 监控 Nodus


<!--
https://www.circonus.com/
-->

## 日志 Agent

## 安全 Agent

### 参考

[Advanced Intrusion Detection Environment, AIDE](http://aide.sourceforge.net/) 是一个目录以及文件的完整性检查工具。

<!-- https://linux.cn/article-4242-1.html -->











<!--
前端介绍
https://github.com/brickspert/blog
-->



{% highlight text %}
{% endhighlight %}
