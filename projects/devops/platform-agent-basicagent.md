---
title: BasicAgent 实现
layout: project
comments: true
language: chinese
category: [misc]
keywords:
description:
---


## 启动流程

{% highlight text %}
1. 通过 DNS 解析获取 config.cargo.com 的 IP 地址信息，会依次尝试建立链接。
   可以先通过/etc/hosts配置，通过多行将该域名设置多个 IP 地址
       127.0.0.1   config.cargo.com
       127.0.0.2   config.cargo.com
       127.0.0.3   config.cargo.com

2. 发送初始化报文，同时可以根据上层策略决定是否重定向进行负载均衡。

3. 建立TLS链接。
{% endhighlight %}

### 0.1 初始化报文

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

## 命令通道

用于在服务器上执行命令。

### 同步命令

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

### 异步任务

### 周期任务

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

## 查询接口

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

## 命令行

可以通过命令行直接加载动态库获取数据。

{% highlight text %}
BasicAgentCtl [options]
参数：
    -c <command>    指定命令，例如getinfo.kernel、getinfo.users等；
    -a <arguments>  上述命令的参数，不同的指标参数略有区别；

===> getinfo.users 获取用户信息，注意执行需要ROOT权限
./BasicAgentCtl -c "getinfo.users" -a "-U root" | python -m json.tool  # 指定用户
./BasicAgentCtl -c "getinfo.users" -a "-A" | python -m json.tool  # 所有
./BasicAgentCtl -c "getinfo.users" -a "-u" | python -m json.tool  # 普通用户
./BasicAgentCtl -c "getinfo.users" -a "-S" | python -m json.tool  # 系统用户

===> getinfo.kernel 获取内核信息
./BasicAgentCtl -c "getinfo.kernel" | python -m json.tool
{% endhighlight %}


{% highlight text %}
{% endhighlight %}
