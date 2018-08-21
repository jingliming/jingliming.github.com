---
title: BootAgent 实现
layout: project
comments: true
language: chinese
category: [misc]
keywords:
description:
---

## 简介

用来管理上述列表中最基本的 Agent，提供基本的功能。通过最简单的方式与服务端进行通讯，也就是利用 `HTTP/1.2` 短链接与服务端通讯，只提供简单的 PUSH 机制，由 BootAgent 主动发起。

## 实现功能

其中主要功能点包括了。

{% highlight text %}
1. 任务管理。
   1.1 子Agent安装。
   1.2 修改配置。
2. 子进程管理。
   2.1 子Agent退出自动拉起。
   2.2 子Agent资源限制。
3. 事件上报机制。
   3.1 子进程异常。
{% endhighlight %}

如果任务执行失败，只能等待下次 BootAgent 上报数据时处理。

## REST-API 接口

### 启动注册接口

{% highlight text %}
----- POST /agent/register
{
	"hostname": "127.0.0.1",
	"agentsn": "bfdcc18c-b6b9-4725-9c47-37fd93dba5b6"
}

----- 返回信息
{
	"status": 0,
	"agentsn": "dcb886e9-04ed-41bb-9c12-4d2de12cd59b",  # 如果上层判断有冲突，则执行AgentSN
	"tags": "service=ecs,component"
}
{% endhighlight %}

### 状态上报接口

{% highlight text %}
----- POST /agent/status 上报当前Agent状态
{
	"hostname": "127.0.0.1",                 # 主机名
	"version": "bootagent-1.2.4-x86_64",     # BootAgent的版本号
	"uptime": 1234,                          # 进程已经启动时间，单位秒
	"timestamp": 1232,                       # 上报时的时间戳
	"step": 600,                             # 上报的时间间隔，可以做修改，默认10min
	"agents": [{                             # 当前主机Agent信息
		"name": "BasicAgent",            # Agent名称，需要保证唯一
		"version": "1.2.3",              # Agent当前版本号
		"status": "stoped",              # Agent的状态
		"uptime": 123,                   # 已经运行时间
	}]
}

----- 返回信息，可以包含请求
{
	"errcode": 2,                            # 返回状态，0表示正常，目前直接忽略返回值
	"jobs": [{
		"id": "ddc8a9b9-55bd-4ddd-b53d-47095ee19466",  # 任务ID信息，由服务端指定，客户端上报执行结果时会带上
		"action": "install",                           # 指定任务操作，也开始是"upgrade" "uninstall" "restart" "stop"等
		"name": "BasicAgent",                          # 需要操作的子Agent名称
		"url": "ftp://server:port/BasicAgent/BasicAgent-1.5.6..rpm",
		"checksum": "SHA256:4a34b8d7d3009bb9ef9475fbf33e7bbe4a1e8db003aefc578a241c2f51c2c2f2",
		"envs": {                                      # 运行时的环境变量，一般在初次安装时配置，可以每次更新
			"PATH": "/usr/bin;/usr/local/bin"
		},
		"limits": {                                    # 资源使用限制
			"CPU": "10%",
			"memory": "20M"
		}
	}, {
		"id": "ddc8a9b9-55bd-4ddd-b53d-47095ee19466",
		"action": "config",                            # 修改BootAgent的相关配置
		"svrlist": "192.168.9.1:1234,",                # 服务端列表，不会修改默认的列表
		"step": 1200,                                  # 状态上报时间间隔
	}]
}

备注:
    Agent状态
      * stoped 主动停止
      * running 正在运行
      * fatal 多次重试后失败
      * killed 资源超限等原因被强制杀死
{% endhighlight %}

### 任务执行结果

{% highlight text %}
----- POST /agent/job/result 上报任务执行的状态
[
	{
		"id": "ddc8a9b9-55bd-4ddd-b53d-47095ee19466",
		"errcode": 0,
		"message": "success"
	}
]

----- 返回信息
{
	"errcode": 0                                           # 一般都是0，非0会一直尝试上报上述结果
}
{% endhighlight %}

## TODO

{% highlight text %}
1. 安全通讯。
{% endhighlight %}

## 测试用例

## FAQ

#### Agent 重启策略

每隔 60s 检查一次，如果进程异常退出在 3min 内不会重启，如果在 15min 内异常则标记一次异常事件，当连续异常超过 3 次后则认为程序异常，不再尝试重启。


<!--
1. 进程管理
   1.1 配置文件中有多个 Name 相同的配置文件。后续的配置文件解析时会报错。
   1.2 执行用户相关。
       1.2.0 用户存在。以指定用户执行。
       1.2.1 用户不存在。直接报错退出。
       1.2.2 用户没有指定。默认通过root执行。
       1.2.3 属组非默认。指定属组执行。
   1.3 进程检查。


ps -eo ppid,pid,user,group,euser,egroup,cmd | grep gearman
usermod -a -G root monitor 将monitor用户添加到root组中
-->


{% highlight text %}
{% endhighlight %}
