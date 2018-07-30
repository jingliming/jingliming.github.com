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

### 文档服务器搭建

增加 HTTP/FTP 服务器，分别使用 nginx 和 vsftpd 搭建。

#### HTTP 服务

在 `/etc/nginx/conf.d` 目录下包含了各种配置文件，如果要增加新配置，可以直接在该目录下新增配置文件 `fileserver.conf` 。

{% highlight text %}
server {
	listen 80;                       # 监听端口
	server_name 127.0.0.1;           # 如果没有DNS解析，可以设置IP地址
	client_max_body_size 4G;         # 设置最大文件大小
	charset utf-8;                   # 防止出现中文乱码
	root /files;                     # 指定相对路径的根目录，如下的location会相对该路径
	location /packages {             # 实际存放文件的目录为/files/packages/
		auth_basic "Restricted"; # 输入密码时的提示语
		auth_basic_user_file /etc/nginx/pass_file; # 认证时用户密码文件存放路径
		autoindex on;            # 自动生成文件索引
		autoindex_exact_size on; # 显示文件大小
		autoindex_localtime on;  # 显示本地文件时间
	}
}
{% endhighlight %}

然后通过 `nginx -t` 检查语法是否正确，通过 `nginx -s reload` 重新加载配置；然后通过 [http://127.0.0.1/files/packages](http://127.0.0.1/files/packages) 访问。

另外，简单的可以采用 python 提供模块，不过只能采用单线程。

{% highlight text %}
----- 启动一个简单的文件服务器
$ python -m SimpleHTTPServer 9000

$ curl http://127.0.0.1:9000
{% endhighlight %}

可以直接通过 [http://127.0.0.1:8000](http://127.0.0.1:9000) 地址访问，可以看到目录下的文件列表。


#### FTP 服务

直接启动 vsftpd 服务器即可。

### 后台服务搭建

这里基于 Flask 和 MySQL 搭建后台服务器。

#### MySQL

{% highlight text %}
----- 安装社区版本MySQL，新增配置
# rpm -Uvh http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
# yum install mysql-community-server
# cat << EOF > /etc/my.cnf
[client]
password        = new_password
port            = 5506
socket          = /tmp/mysql-5506.sock
[mysqld]
user            = mysql
port            = 5506
socket          = /tmp/mysql-5506.sock
basedir         = /usr
datadir         = /tmp/data-5506
pid-file        = /tmp/data-5506/mysqld.pid
EOF

----- 初始化数据，可通过空白密码登陆
# mkdir /tmp/data-5506
# /usr/sbin/mysqld --initialize-insecure --basedir=/usr --user=mysql \
    --datadir=/tmp/data-5506
# systemctl start mysqld

----- 登陆并修改密码，第一次登陆时密码空白即可，分别对应了5.6以及5.7版本
$ mysql -h 127.1 -p
mysql> UPDATE mysql.user SET password=PASSWORD('new_password') WHERE user='root';
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
mysql> FLUSH PRIVILEGES;
{% endhighlight %}





### REST-API

{% highlight text %}
-----> 发送请求
{
	"id": "2719cb14-2da6-44a0-8349-1ea33b3ab2d9",
	"hosts": [ "eccb24a7-fe55-4d84-9c40-6a9192419ae9", "b19ffa7e-712e-46d4-b328-2d2892c96076" ],
	"method": "sync.bash",
	"cmd": "ls"
}

<----- 响应报文
{
	"id": "2719cb14-2da6-44a0-8349-1ea33b3ab2d9",
	"response": [ {
		"host": "eccb24a7-fe55-4d84-9c40-6a9192419ae9",
		"resp": "failed",
		"retcode": 203,
		"message": "No such host"
	}, {
		"host": "b19ffa7e-712e-46d4-b328-2d2892c96076",
		"resp": "success",
		"retcode": 2,
		"message": "Normal termination",
		"data": "xxxxxx"
	} ],
}


{% endhighlight %}




## Agents

通过基础 Agent 管理 监控、日志、安区、网络等客户端。

* BootAgent 装机时安装，负责安装、升级、监控、重启下述的Agent，通过短链接主动上报如下Agent状态信息
* BasicAgent 基本命令执行(同步、异步、查询)，安装部署
* LogAgent 按照固定格式采集日志数据
* MonitorAgent 采集监控数据
* SecureAgent 安全相关

### BootAgent

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
	"errcode": 2,                            # 返回状态，0表示正常
	"jobs": [{
		"id": "ddc8a9b9-55bd-4ddd-b53d-47095ee19466",  # 任务ID信息，由服务端指定，客户端上报执行结果时会带上
		"action": "install",                           # 指定任务操作，也开始是"upgrade" "uninstall" "restart" "stop"等
		"name": "BasicAgent",                          # 需要操作的子Agent名称
		"url": "ftp://server:port/BasicAgent/BasicAgent-1.5.6..rpm",
		"checksum": "SHA256:4a34b8d7d3009bb9ef9475fbf33e7bbe4a1e8db003aefc578a241c2f51c2c2f2",
		"env": {                                       # 运行时的环境变量，一般在初次安装时配置，可以每次更新
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

### 基础 Gearman

#### 0. 核心流程

##### Agent 启动流程

{% highlight text %}
1. 通过 DNS 解析获取 config.cargo.com 的 IP 地址信息，会依次尝试建立链接。
   可以先通过/etc/hosts配置，通过多行将该域名设置多个 IP 地址
       127.0.0.1   config.cargo.com
       127.0.0.2   config.cargo.com
       127.0.0.3   config.cargo.com

2. 发送初始化报文，同时可以根据上层策略决定是否重定向进行负载均衡。

3. 建立TLS链接。
{% endhighlight %}

##### 0.1 初始化报文

{% highlight text %}
-----> 请求报文
{
    "method": "init",                                      # 必选
    "hostname": "edd6c192-52cb-4133-a17a-e7d8aec03de7",    # 必选，指定同步命令类型
}

<----- 响应报文
{
    "method": "redirect",                  # 必选
    "server": "127.0.10.1,10.92.1.130",    # 必选，重定向的地址
}
{% endhighlight %}



{% highlight text %}
./boot -L /tmp/booter.log -P /tmp/booter.pid -f
{% endhighlight %}

#### 1. 执行命令

用于在服务器上执行命令。

##### 1.1 同步命令

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

##### 1.2 异步任务

##### 1.3 周期任务

{% highlight text %}
 .------------------ second (0~59) - * /
 |  .--------------- minute (0~59) - * /
 |  |  .------------ hour (0~23) - * /
 |  |  |  .--------- day of month (1~31) - * / ? L W C
 |  |  |  |  .------ month (1~12), jan,feb,mar,apr ...
 |  |  |  |  |  .--- day of week (0~7) (Sunday=0/7) OR sun,mon,tue,wed,thu,fri,sat L C
 |  |  |  |  |  |
 *  *  *  *  *  * user-name  command to be executed
 0  1  2  3  4  5
{% endhighlight %}

这里包含了 6 个通过空格分开的字段，其表示的时间段如上所示，可以通过如下方式表示。

{% highlight text %}
/    递增方式
? L  只有 month 和 weekday 有效
{% endhighlight %}

其中 `?` 只能用在 `Day of Month` 和 `Day of Week` 两个域，它也匹配域的任意值，但实际不会，因为两者会相互影响。假设，想每月 20 号触发，而不关心具体是星期几，则可以使用 `0 0 0 10 * ?` ，最后必须是 `?` ，如果是 `*` 则表示无论是星期几都触发。










示例如下：

{% highlight text %}
"0 0 * * * *"                 the top of every hour of every day
"*/10 * * * * *"              every ten seconds
"0 0 8-10 * * *"              8, 9 and 10 o'clock of every day
"0 0/30 8-10 * * *"           8:00, 8:30, 9:00, 9:30 and 10 o'clock every day
"0 0 9-17 * * MON-FRI"        on the hour nine-to-five weekdays
"0 0 0 25 12 ?"               every Christmas Day at midnight


*/10 * * * *     echo "Ten minutes ago." >> /tmp/foo.txt    // 每十分钟执行一次
0 6 * * *        echo "Good morning." >> /tmp/foo.txt       // 每天早上6点
0 */2 * * *      echo "Have a break now." >> /tmp/foo.txt   // 每两个小时
45 4 1,10,22 * * echo "Restart server." >> /tmp/foo.txt     // 每月1、10、22日的4:45
0 23-7/2,8 * * * echo "Have a good dream." >> /tmp/foo.txt  // 晚上11点到早上8点之间每两个小时，早上八点
0 11 4 * 1-3     echo "Just kidding." >> /tmp/foo.txt       // 每月4号和每周的周一到周三的早上11点
45 11 * * 0,6    echo "Have a good lunch." >> /tmp/foo.txt  // 每周六、周日的11点45分
0 9 * * 1-5      echo "Work hard." >> /tmp/foo.txt          // 从周一到周五的9点
2 8-16/3 * * *   echo "Some examples." >> /tmp/foo.txt      // 8:02、11:02、14:02执行

0,30 18-23 * * *    echo "Same." >> /tmp/foo.txt            // 每天18:00到23:00之间每隔30分钟
0-59/30 18-23 * * * echo "Same." >> /tmp/foo.txt            // 同上
*/30 18-23 * * *    echo "Same." >> /tmp/foo.txt            // 同上
{% endhighlight %}

在表达式中，每位表示一个时间，例如秒、分、时等。

#### 2. 获取信息

提供通用接口获取主机信息，例如内核信息、用户信息等。

{% highlight text %}
-----> 请求报文
{
    "id": "xxxxxx",
    "method": "getinfo.users",  # 必选
    "args": "",                 # 可选，不同方法参数不同
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

##### 命令行

可以通过命令行获取数据，将会直接加载动态库然后执行。

{% highlight text %}
gearctl [options]
参数：
    -c <command>    指定命令，例如getinfo.kernel、getinfo.users等；
    -a <arguments>  上述命令的参数，不同的指标参数略有区别；


===> getinfo.users 获取用户信息，注意执行需要ROOT权限
./gearctl -c "getinfo.users" -a "-U root" | python -m json.tool  # 指定用户
./gearctl -c "getinfo.users" -a "-A" | python -m json.tool  # 所有
./gearctl -c "getinfo.users" -a "-u" | python -m json.tool  # 普通用户
./gearctl -c "getinfo.users" -a "-S" | python -m json.tool  # 系统用户

===> getinfo.kernel 获取内核信息
./gearctl -c "getinfo.kernel" | python -m json.tool

{% endhighlight %}

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




### 监控 Nodus

负责周期性的采集数据，并将数据 PUSH 到服务器。

<!--
为了简单起见，队列没有采用超时机制，简单来说，也就是读写线程采用协作型。
对于读线程，不会设置超时时间，完全由采集线程控制，不过如果采集超时，会对读线程进行惩罚，简单来说就是增加采集的时间间隔。
对于写线程，每次发送时都会直接从队列中摘除，如果有多个写插件，那么只要有一个插件写成功那么就认为发送成功，失败时则会直接丢弃，不会维护队列进行重试。
  0. 每次接收到一个采集指标时，都会触发写操作，而且会通过单个线程写所有写插件。
  1. 每个写插件需要维护自己的队列进行重试，而且可以在线程中阻塞。
  2. 当所有的线程都阻塞时，那么就没有线程处理写请求了，此时就会阻塞到缓冲队列中。
  3. 要其每个写插件是可重入的。
-->

#### 0. 参数列表

详细参数可以直接通过 `nodus -h` 命令查看。

##### 0.1 检查配置

通过 `-t` 参数检查配置文件是否合法，一般使用命令 `nodus -t -L test.log` ，也就是同时指定日志文件地址，防止日志被混淆。

如果成功则会显示 `Success` 否则 `Failed!!!` ，同时程序返回值分别为 `0` 和 `1` 。

假设要测试某个配置文件是否合法，可以通过如下命令测试。

{% highlight text %}
$ nodus -f -C nodus.server.conf -L server.log
{% endhighlight %}

##### 0.2 测试插件

也就是通过 `-T` 参数，同样一般使用命令 `nodus -T -L test.log` ，会将插件的一次采集打印出来，格式以及其中的示例如下。

{% highlight text %}
采集时间/周期 插件名(采集指标数/types.db指标数) 采集值1/类型1 采集值2/类型2 ... 采集值N/类型N

1511965574.110/10.000 load(3/3) 2.71/G 2.53/G 2.43/G
{% endhighlight %}

#### 1. 控制命令

提供了一个常见的控制命令用于操作，也就是 `nodusctl` ，如果要查看与服务端的详细通讯过程，可以在执行命令之前设置环境变量，也就是 `NODUS_TRACE=1;` ，常见示例如下。

{% highlight text %}
----- 查看当前加载了那些插件
$ NODUS_TRACE=1; ./nodusctl plugin list

----- 删除某个插件
$ NODUS_TRACE=1; ./nodusctl plugin remove load

----- 停止某个插件的数据采集
$ NODUS_TRACE=1; ./nodusctl plugin stop load

----- 启动某个插件的数据采集
$ NODUS_TRACE=1; ./nodusctl plugin start load
{% endhighlight %}

#### 5. 常见问题

##### 5.1 查看读函数

查看读插件的调用情况，会显示读线程以及各个读函数的调用过程，需要打开 DEBUG 日志。

{% highlight text %}
$ tail -f nodus.log | grep -E '([Rr]ead-function|[Rr]ead thread)'
{% endhighlight %}



<!--
https://www.circonus.com/
-->

### 日志 Agent

### 安全 Agent

[Advanced Intrusion Detection Environment, AIDE](http://aide.sourceforge.net/) 是一个目录以及文件的完整性检查工具。


## 其它


### 安装路径

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



### 参考

<!-- https://linux.cn/article-4242-1.html -->











<!--
前端介绍
https://github.com/brickspert/blog

前端使用的FontAwesom字体
http://corporate.joostrap.com/features/fontawesome-icons

http://sharadchhetri.com/2013/12/13/how-to-lock-and-unlock-user-account-in-linux/
https://github.com/Distrotech/shadow-utils/blob/distrotech-shadow-utils/src/chage.c

一些常用库
https://github.com/fmela/libdict
https://github.com/attractivechaos/klib
https://github.com/nanomsg/nanomsg

GoLang ZeroMQ
http://blog.haohtml.com/archives/14496

using zeromq with libev
https://github.com/pijyoi/zmq_libev
buffered socket library for libev
https://github.com/mreiferson/libevbuffsock

Introspected tunnels to localhost
https://github.com/koolshare/ngrok-libev
https://github.com/inconshreveable/ngrok

Proxy
https://github.com/isayme/socks5
https://github.com/z3APA3A/3proxy


Message Queue
https://github.com/je-so/iqueue *****
https://github.com/circonus-labs/fq
https://github.com/liexusong/mx-queued
https://github.com/chaoran/fast-wait-free-queue
https://github.com/haipome/lock_free_queue
https://github.com/supermartian/lockfree-queue
https://github.com/slact/nchan
https://github.com/darkautism/lfqueue
https://github.com/tylertreat/gatling
https://github.com/bangadennis/networking
https://github.com/fcten/webit

HTTP Server
https://github.com/bachan/ugh
https://github.com/dexgeh/webserver-libev-httpparser
https://github.com/Lupus/libevfibers
https://github.com/h2o/h2o
https://github.com/monkey/monkey
https://github.com/lpereira/lwan


http://www.tildeslash.com/libzdb/#





DROP TABLE IF EXISTS `hosts`;
CREATE TABLE IF NOT EXISTS `hosts` (
	`id` BIGINT NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(128) DEFAULT NULL COMMENT 'TAG、服务或主机标示(如AgentSN、IP、HOSTNAME)',


	`gmt_create` datetime DEFAULT CURRENT_TIMESTAMP,
	`gmt_modify` datetime ON UPDATE CURRENT_TIMESTAMP,
	UNIQUE KEY uk_name (name),
	PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

DROP TABLE IF EXISTS `jobs`;
CREATE TABLE IF NOT EXISTS `jobs` (
	`id` BIGINT NOT NULL AUTO_INCREMENT,


	`gmt_create` datetime DEFAULT CURRENT_TIMESTAMP,
	`gmt_modify` datetime ON UPDATE CURRENT_TIMESTAMP,
	UNIQUE KEY uk_name (name),
	PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;


DROP TABLE IF EXISTS `host_job`;
CREATE TABLE IF NOT EXISTS `host_job` (
	`hostid` BIGINT NOT NULL,
	`jobid` BIGINT NOT NULL,
	UNIQUE KYE (`jobid`, `hostid`),
	PRIMARY KEY (`hostid`, `jobid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;


假设产品树采用的是三层结构，
产品，对外售卖的服务(如ECS、RDS等)、内部使用平台(数据库管理平台、运维系统等)
服务，一般也就是一个团队独立开发维护的。
组件/微服务，可以独立安装部署的最小单元，例如Console、DB、Server等。

那么保存的任务就会涉及了继承的层级关系，一般来说层级越低的优先级也越高 ECS < OpenStack < Console < Host ，而这里的优先级处理则是直接通过服务端进行处理。



获取主机对应的任务



内存不足测试
https://oomake.com/question/12305
https://code-examples.net/zh-CN/q/1a9c8
kill信号的排查
https://hongjiang.info/shell-script-background-process-ignore-sigint/


1. 超过最大buffer则丢弃后续的数据，目前设置为 64K。
2. 内存不足、read返回失败(认为内部错误，会打印错误信息) 时会强制 KILL 子进程。
   此时进程会返回状态9(直接在回调函数中kill进程进行测试)
3. fork进程后没有通过exevp()执行，而是直接退出。
   此时通过valgrind会看到有reachable的报错，主要是子进程继承的资源未被释放，如果子进程退出时释放所有资源，那么就不会报错。
   不过可以忽略，操作系统会在进程退出时对这部分的内存进行回收。
4. 执行超时3次以后则会强制退出。测试脚本需要忽略 SIGINT(2) 信号，也就是 trap '' 2
5. libev在新创建的进程里面，实际不需要手动关闭正常运行流程，但是以防后面添加了其它的处理逻辑，还是关闭掉。

-->



{% highlight text %}
{% endhighlight %}
