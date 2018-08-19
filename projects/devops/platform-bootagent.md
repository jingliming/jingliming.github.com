---
title: BootAgent 实现
layout: project
comments: true
language: chinese
category: [misc]
keywords:
description:
---



<!-- more -->

用来管理上述列表中最基本的 Agent，提供基本的功能。

通过最简单的方式与服务端进行通讯，也就是利用 `HTTP/1.2` 短链接与服务端通讯，只提供简单的 PUSH 机制，由 Agent 主动发起。

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
