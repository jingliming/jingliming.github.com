---
title: Aspire 实现
layout: project
comments: true
language: chinese
category: [misc]
keywords:
description:
---

<!--
https://my.oschina.net/henrylee2cn/blog/741315
-->


<!--
#### 3. 常驻进程管理

主要用于管理一些子进程，包括了监控、日志、安全相关的进程管理。

##### 3.1 表结构设计

常驻进程相关的数据同样保存在 `gearman.db` 文件中。

{% highlight sql %}
DROP TABLE IF EXISTS `daemon`;

CREATE TABLE IF NOT EXISTS `daemon` (
    `name` CHAR(64) PRIMARY KEY NOT NULL,
    `version` CHAR(64) NOT NULL,
    `url` CHAR(512) NOT NULL,
    `precheck` CHAR(512),
    `precheck_retries` INT,
    `start` CHAR(512) NOT NULL,
    `start_delay` INT,
    `stop` CHAR(512) NOT NULL,
    `stop_timeout` INT,
    `check` CHAR(64),
    `check_interval` INT,
    `user` CHAR(64),
    `group` CHAR(64),
    `pidfile` CHAR(512),
    `socket` CHAR(512),
    `env` TEXT,
    `limits` TEXT,
    `gmt_modify` NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `gmt_create` NOT NULL DEFAULT CURRENT_TIMESTAMP
);
{% endhighlight %}


##### 3.2 下发任务

这里只会更新数据库。

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

其中 stop 中可以使用 `<process|kill>:argument` 这种方式，前者会执行一个命令，后者则会发送信号给对应的进程。


#### 4. 服务端表结构设计

{% highlight sql %}
DROP TABLE IF EXISTS `hosts`;
CREATE TABLE IF NOT EXISTS `hosts` (
	`id` INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
	`hostname` CHAR(256) NOT NULL COMMENT "可以是主机名、AgentSN等",
	`region` CHAR(64) NOT NULL DEFAULT "unkown" COMMENT "所属region信息",
	`status` ENUM('unknown', 'online', 'offline') DEFAULT 'unknown' COMMENT "主机状态",

	`gmt_modify` NOT NULL DEFAULT CURRENT_TIMESTAMP,
	`gmt_create` NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT "保存主机基本信息";

DROP TABLE IF EXISTS `plugins`;
CREATE TABLE IF NOT EXISTS `plugins` (
	`id` INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
	`name` CHAR(128) NOT NULL COMMENT "插件名称",
	`version` CHAR(64) NOT NULL COMMENT "插件版本号信息",

	`gmt_modify` NOT NULL DEFAULT CURRENT_TIMESTAMP,
	`gmt_create` NOT NULL DEFAULT CURRENT_TIMESTAMP,
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT "插件信息";
{% endhighlight %}
-->

{% highlight text %}
{% endhighlight %}
