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

### REST-API

{% highlight text %}
{
	"id": "2719cb14-2da6-44a0-8349-1ea33b3ab2d9",
	"hosts": [ "eccb24a7-fe55-4d84-9c40-6a9192419ae9", "b19ffa7e-712e-46d4-b328-2d2892c96076" ],
	"method": "",
}
{% endhighlight %}




## Agents

通过基础 Agent 管理 监控、日志、安区、网络等客户端。

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
DROP TABLE IF EXISTS `aemon`;

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
-->



{% highlight text %}
{% endhighlight %}
