---
title: Collectd 简介
layout: post
comments: true
language: chinese
category: [linux,misc]
keywords: collectd,monitor,linux
description: collectd 是一个后台监控程序，用来采集其所运行系统上的系统信息，并提供各种存储方式来存储为不同值的格式，例如 RRD 文件形式、文本格式、MongoDB 等等。在此，简单介绍下 collectd 。
---

collectd 是一个后台监控程序，用来采集其所运行系统上的系统信息，并提供各种存储方式来存储为不同值的格式，例如 RRD 文件形式、文本格式、MongoDB 等等。

在此，简单介绍下 collectd 。

<!-- more -->

![collectd logo]({{ site.url }}/images/linux/collectd_logo.png "collectd logo"){: .pull-center width="40%" }

## 简介

Collectd 完全由 C 语言编写，故性能很高，可移植性好，它允许运行在系统没有脚本语言支持或者 cron daemon 的系统上，比如嵌入式系统；同时，它包含优化以及处理成百上千种数据集的新特性，目前包含了几十种插件。

其中数据采集和采集数据的写入都可以通过插件进行定义，目前支持的数据采集插件包括了几十种，写入包括了 Graphite、RRDtool、Riemann、MongoDB、HTTP 等。

简单来说，其特点如下：

1. C语言、插件、多线程(读写线程池)，支持 Python、Shell、Perl 等脚本语言；
2. 数据采集插件包括了 OS(CPU、Memory、Network、Disk等)、通用组件(Nginx、MySQL、PostgreSQL、Redis、HAProxy等)；
3. 写入插件支持 RRDTool、Kafka、MongoDB、Redis、Riemann、Sensu、TSDB等；
4. 支持级联方式，数据发送采用 UDP 报文；

如下是将采集的数据保存为 rrd 格式文件，并通过 rrdtool 工具绘制出来。

![collectd cpu rrd]({{ site.url }}/images/linux/collectd_cpu_example.png "collectd cpu rrd"){: .pull-center }

### 安装

对于 CentOS 可以直接安装 collectd 对应的 RPM 包，不同的插件可以安装不同的包。

{% highlight text %}
----- 查看当前版本，以及所支持的插件
$ yum --enablerepo=epel list all | grep collectd
----- 安装
# yum --enablerepo=epel install collectd
{% endhighlight %}

### 源码编译

编译选项可以直接通过 ```./configure --help``` 命令查看，例如可以通过如下配置开启调试选项，也就是 COLLECT_DEBUG 宏；对于插件会自动检查是否存在头文件，例如 rrdtool 需要 rrd.h 文件，也就是需要安装 rrdtool-devel 包。

{% highlight text %}
$ ./configure --enable-debug --enable-all-plugins
{% endhighlight %}

注意，在配置完之后会打印配置项信息，包括了编译、链接参数，以及支持哪些插件等，也可以查看 config.log 文件。

编译完成之后，可以直接通过 ```make check``` 进行检查；对于 RPM 包生成，在 contrib 中有相应的 spec 文件；对于帮助文档可以查看 ```man -l src/collectd.1```；关于支持哪些组件，可以查看源码中的 README 文件，例如测试时，可以直接开启 write_log 写入插件。

另外，编译后的动态库保存在 src/.libs 目录下，如果不想安装，可以直接调整配置文件即可。


### 常见操作

{% highlight text %}
----- 启动服务
$ collectd -C collectd.conf
----- 检查配置文件
$ collectd -C collectd.conf -t
----- 测试插件，会调用一次插件
$ collectd -C collectd.conf -T
----- 刷新，会生成单独线程执行do_flush()操作
$ kill -SIGUSR1 `pidof collectd`
----- 关闭服务，也可以使用SIGTERM信号
$ kill -SIGINT `pidof collectd`
----- 采用多线程，可以通过该命令查看当前的线程数
$ ps -Lp `pidof collectd`
{% endhighlight %}

#### 状态查看

Collectd 可以通过 unixsock 插件进行通讯、查看状态等，也可以使用 collectdctl 命令；通过如下配置打开 unixsock 。

{% highlight text %}
LoadPlugin unixsock
<Plugin unixsock>
  SocketFile "/tmp/collectd.sock"
  SocketGroup "collectd"
  SocketPerms "0660"
  DeleteSocket true                  # 启动时如果存在sock，是否尝试删除
</Plugin>
{% endhighlight %}

在开启了 UnixSock 之后，可以通过如下命令与 Collectd 进行交互；对于 collectdctl 命令，可直接通过 ```man -l src/collectdctl.1``` 查看帮助。

{% highlight text %}
$ collectdctl -s /tmp/collectd.sock listval
$ echo "PUTNOTIF severity=okay time=$(date +%s) message=hello" | \
    socat - UNIX-CLIENT:/path/to/socket
{% endhighlight %}


### 日志配置

collectd 支持多种日志格式，包括了 syslog、logstash、本地日志文件等；如果调试，也可以通过 write_log 插件将采集数据写入到日志文件中。

<!--
另外，可以使用 logrotate 管理日志数据，配置如下。TODO:如何确认logrotate配置有效%%%%%无法自动重新打开,logrotate失效!!!

cat <<- EOF > /etc/logrotate.d/collectd.conf
var/log/nginx/*.log /var/log/tomcat/*log {   # 可以指定多个路径
    daily                      # 每天切换一次日志
    rotate 15                  # 默认保存15天数据，超过的则删除
    compress                   # 切割后压缩
    delaycompress              # 切割时对上次的日志文件进行压缩
    dateext                    # 日志文件切割时添加日期后缀
    missingok                  # 如果没有日志文件也不报错
    notifempty                 # 日志为空时不进行切换，默认为ifempty
    create 644 monitor monitor # 使用该模式创建日志文件
    postrotate # TODO
        if [ -f /var/run/nginx.pid ]; then
            kill -USR1 `cat /var/run/nginx.pid`
        fi
    endscript
}

FIXME: 如果启用logfile，则尝试打开默认文件，没有权限则报错。用于打印 [2017-04-27 13:57:32] plugin_load: plugin "logfile" successfully loaded. 信息。
-->

## 推荐配置

源码中有一个 src/collectd.conf 参考配置文件，如下是一些常见的配置项。

{% highlight text %}
# 指定上报的主机名，可以为IP或者按照规则定义主机名称(service_name.componet_name)
Hostname    "foobar"
# 是否允许主机名查找，如果DNS配置可能有误，建议不要开启
FQDNLookup   false

# 各种文件、目录的设置
BaseDir     "/var/lib/collectd"
PIDFile     "/var/run/collectd.pid"
PluginDir   "/home/andy/Workspace/packages/collectd/collectd-5.7.1/src/.libs"
TypesDB     "/home/andy/Workspace/packages/collectd/collectd-5.7.1/src/types.db"

# 设置为true时，对于<Plugin XXX>可以自动加载插件，无需指定LoadPlugin，详见dispatch_block_plugin()
AutoLoadPlugin false

# 是否同时上报collectd自身的状态，会通过plugin_register_read()注册读取函数，也即会通过
# 执行plugin_update_internal_statistics()函数上报当前collectd的状态
CollectInternalStats false

# 设置全局的数据采集时间间隔，单位是秒，可以在插件中进行覆盖；关于最大采集时间间隔，详见后面的介绍
Interval                 1
MaxReadInterval        180

# 注意这里单位不是秒，实际超时时间是timeout*interval
Timeout                  2

# 用于配置读写线程数
WriteThreads             2
ReadThreads              5

# 配置缓存的上下限
WriteQueueLimitLow    8000
WriteQueueLimitHigh  12000

# 创建一个Unix Socket用于一些命令发送，状态查看等
LoadPlugin unixsock
<Plugin unixsock>
  SocketFile "/tmp/collectd.sock"
  SocketGroup "collectd"
  SocketPerms "0660"
  DeleteSocket true                  # 启动时如果存在sock，是否尝试删除
</Plugin>

# 设置日志文件，保存到本地文件中，可以通过logrotate管理
LoadPlugin logfile
<Plugin logfile>
    LogLevel info
    File "/var/log/collectd/collectd.log"  # 也可以配置为STDOUT，作为标准输出
    Timestamp true
    PrintSeverity true
</Plugin>

# 通过write_log插件将采集数据保存到日志中，默认采用的是Graphite格式
LoadPlugin write_log
<Plugin write_log>
  Format JSON
</Plugin>
{% endhighlight %}

如下简单列举一些常用的配置。

1\. 数据缓存上下限设置。

可以通过 ```WriteQueueLimitHigh```、```WriteQueueLimitLow``` 参数可以配置缓存队列的大小，在源码中对应了 ```write_limit_high```、```write_limit_low``` 变量。

当小于 ```WriteQueueLimitLow``` 时不会丢弃所采集的数据，当大于上述值而小于 ```WriteQueueLimitHigh``` 时会概率性丢弃，而大于 High 则直接丢弃。

2\. 插件采集时间间隔。

如果调用插件读取数据时失败，则会将下次采集时间 double，但是最大值为配置文件中的设置值；当一次读取成功时，则会直接恢复到正常的采集时间间隔。

3\. 关于Timeout参数

主线程会检查是否有数据采集超时，注意这里真正的超时时间是 timeout*interval ，如果超时则会调用注册的 missing 回调函数，而且会在 global cache 中删除这个对象。

4\. types.db 的使用

type.db 每行由两个字段组成，由空格或者 tab 分隔，分别表示：A) 数据集名称；B) 数据源说明的列表，对列表单元都以逗号分隔。

每个数据源通过冒号分为 4 部分：数据源名、类型、最小值和最大值 ```ds-name:ds-type:min:max```；其中 ds-type 包含 4 种类型 (```ABSOULUTE```, ```COUNTER```, ```DERIVE```, ```GAUSE```)，其中，mix 和 max 定义了固定值范围，U 表示范围未知或者不限定。

每行数据对应了一个 ```value_list_t```，可以通过 ```format_values()``` 函数进行格式化。

5\. 如何查看 collectd 的状态？

自身状态查看，可以在配置文件中添加 ```CollectInternalStats true``` 选项，或者开启 unixsocket，然后通过 collectdctl 命令进行查看。

显然，一个是可以远程采集数据的，一个是只能本地访问的。

## Filter-Chain

从 4.6 版本开始支持 filter-chain 功能，可用于将某个值发送给特定的输出，其工作模式类似于 ip_tables，包括了 matches 和 targets，前者用于匹配一个规则，后者用于执行某个操作。

如下是处理流程图。

{% highlight text %}
 +---------+
 ! Chain   !
 +---------+
      !
      V
 +---------+  +---------+  +---------+  +---------+
 ! Rule    !->! Match   !->! Match   !->! Target  !
 +---------+  +---------+  +---------+  +---------+
      !
      V
 +---------+  +---------+  +---------+
 ! Rule    !->! Target  !->! Target  !
 +---------+  +---------+  +---------+
      !
      V
      :
      :
      !
      V
 +---------+  +---------+  +---------+
 ! Rule    !->! Match   !->! Target  !
 +---------+  +---------+  +---------+
      !
      V
 +---------+
 ! Default !
 ! Target  !
 +---------+
{% endhighlight %}

简单介绍下如上的常见概念：

* Match，用于匹配规则，例如采集指标名称以及当前采集的值，通常规则使用 match_ 插件，不过在使用前需要提前加载。
* Target，就是将要执行的一些操作，插件以 target_ 开头。
* Rule，零到多个 match 规则以及至少一个 target 组成 rule ，如果没有规则对所有的值匹配。
* Chain，包括了一系列的 rule 以及可能的默认目标，会按照规则进行匹配，如果满足则指定 target 执行。

Chain 规则，目前只支持 Pre-Chain 和 Post-Chain 两种，在配置文件中可通过 PreCacheChain、PostCacheChain 用于配置 filter-chain 功能。


<!--
There are four ways to control which way a value takes through the filter mechanism:

jump
    The built-in jump target can be used to ``call'' another chain, i. e. process the value with another chain. When the called chain finishes, usually the next target or rule after the jump is executed. 
stop
    The stop condition, signaled for example by the built-in target stop, causes all processing of the value to be stopped immediately. 
return
    Causes processing in the current chain to be aborted, but processing of the value generally will continue. This means that if the chain was called via Jump, the next target or rule after the jump will be executed. If the chain was not called by another chain, control will be returned to the daemon and it may pass the value to another chain. 
continue
    Most targets will signal the continue condition, meaning that processing should continue normally. There is no special built-in target for this condition. 
-->

### 示例

一个简单的示例如下。

{% highlight text %}
PostCacheChain "PostCache"
<Chain "PostCache">
  <Rule "ignore_mysql_show">
    <Match "regex">
      Plugin "^mysql$"
      Type "^mysql_command$"
      TypeInstance "^show_"
    </Match>
    <Target "stop">
    </Target>
  </Rule>
  <Target "write">
    Plugin "rrdtool"
  </Target>
</Chain> 
{% endhighlight %}

上述的配置中，会忽略 ```plugin=mysql + type=mysql_command + type_instance=show_``` 采集的值，不过需要注意的是，这个操作是在 ```PostCache``` 链表中，所以仍然可以通过 unixsock 插件查看采集值。




<!--
## 优化建议




通过cdtime()函数获取当前时间，实际上调用clock_gettime()系统函数，统计从1970.1.1开始时间；然后通过nanosleep()休眠。

struct timespec {
    time_t tv_sec; /* seconds */
    long tv_nsec;  /* nanoseconds */
};
typedef uint64_t cdtime_t;
#define NS_TO_CDTIME_T(ns)                                                     \
  (cdtime_t) {                                                                 \
    ((((cdtime_t)(ns)) / 1000000000) << 30) |                                  \
        ((((((cdtime_t)(ns)) % 1000000000) << 30) + 500000000) / 1000000000)   \
  }
#define TIMESPEC_TO_CDTIME_T(ts)                                               \
  NS_TO_CDTIME_T(1000000000ULL * (ts)->tv_sec + (ts)->tv_nsec)

在内部维护了list以及heap两个结构，分别用于保存当前的插件，以及下个周期需要调用那个插件。


LoadPlugin rrdtool
对于rrdtool插件，会在DataDir目录下创建 {hostname}/{measurement} 目录保存采集数据。
-->

## 通讯协议

Collectd 提供了两种协议 [Plain text protocol](https://collectd.org/wiki/index.php/Plain_text_protocol) 以及 [Binary protocol](https://collectd.org/wiki/index.php/Binary_protocol) 。

### Plain text protocol

这一协议目前可以在使用 Exec 执行脚本时使用，或者通过 UnixSock 查看当前 Collectd 服务的状态时使用。注意：一行的请求需要小于 1024 字节，否则报 "Parsing options failed" 错误。

#### LISTVAL

语法是 ```LISTVAL``` ，用于查看当前监控项，会打印出上次采集的时间点。

{% highlight text %}
$ collectdctl -s /tmp/collectd.sock listval
$ echo "LISTVAL" | nc -U /tmp/collectd.sock
$ echo "LISTVAL" | socat - UNIX-CLIENT:/tmp/collectd.sock
{% endhighlight %}

#### GETVAL

语法是 ```GETVAL Identifier``` ，获取某个指标的采集值，指标通过上述的标示符 (Identifier) 定义。

{% highlight text %}
$ echo 'GETVAL "localhost/memory/memory-buffered"' | nc -U /tmp/collectd.sock
{% endhighlight %}

#### PUTVAL

语法是 ```PUTVAL Identifier [OptionList] Valuelist``` ，用于将某个监控项提交给 Collectd 服务，会自动转给写插件。

* OptionList: 选项列表，每项通过 KV 表示，目前只支持 interval=seconds 这一模式。
* Valuelist: 通过冒号分割的时间戳以及值，支持的数据类型有 uint(COUNTER+ABSOLUTE), int(DERIVE), double(GAUGE)；对于 GAUGE 可以使用 "U" 标示未定义值，但是不能对 COUNTER 使用 "U"；时间戳采用 UNIX epoch，也可以使用 "N" 表示使用当前时间。

{% highlight text %}
$ echo 'PUTVAL "localhost/load/load" interval=10 N:13:456:5000' | nc -U /tmp/collectd.sock
{% endhighlight %}

<!--
##### PUTNOTIF

echo "PUTNOTIF severity=okay time=$(date +%s) message=hello" | \
  socat - UNIX-CLIENT:/path/to/socket
-->

## Thresholds

4.3.0 版本之后，开始支持监控，实际上也就是说不仅支持数据的采集上报，而且会对采集数据进行判断，如果定义的事件发生则会上报。

这一操作主要是通过 threshold 插件完成，然后发送 notification 消息；而其它的插件则可以注册接收该消息并作进一步的处理。

注意，这里没有对采集的数据进行处理，也就是没有做类似移动平均线 (Moving Average) 之类的处理，也就意味着比较敏感，可能会有误报。

如下是一个简单的配置文件。

{% highlight text %}
LoadPlugin "threshold"
<Plugin "threshold">
  <Type "foo">
    WarningMin    0.00
    WarningMax 1000.00
    FailureMin    0.00
    FailureMax 1200.00
    Invert false
    Instance "bar"
  </Type>

  <Plugin "interface">
    Instance "eth0"
    <Type "if_octets">
      FailureMax 10000000
      DataSource "rx"
    </Type>
  </Plugin>

  <Host "hostname">
    <Type "cpu">
      Instance "idle"
      FailureMin 10
    </Type>

    <Plugin "memory">
      <Type "memory">
        Instance "cached"
        WarningMin 100000000
      </Type>
    </Plugin>

    <Type "load">
       DataSource "midterm"
       FailureMax 4
       Hits 3
       Hysteresis 3
    </Type>
  </Host>
</Plugin>
{% endhighlight %}

在插件中可以通过 Host Plugin Type 配置块决定需要监控的指标 (注意需要按顺序嵌套)，而且 Plugin Type 还可以通过 Instance 决定哪个示例。

<!--
(WarningMax, FailureMax] + [FailureMin, WarningMin) Warning信息
(FailureMax, ) + ( , FailureMin) Failure信息

DataSource DSName
  某些监控项包括了多个数据源，例如 if_octets 包括了 rx 和 tx、load 包括了 shortterm、midterm、longterm 等等；默认会检查所有的采集项，或者通过该参数指定需要检查的采集项。
Invert true|false(default)
  监控时间策略默认如上所示，当然可以通过该参数取反，也就是在 FailureMin~FailureMax 范围内是异常的。
Persist true|false(default)
  设置Notify通知的频率，true 每个采集点都会发送一个通知；false 越过边界时发送通知，也就是上次正常，本次异常。
PersistOK true|false

    Sets how OKAY notifications act. If set to true one notification will be generated for each value that is in the acceptable range. If set to false (the default) then a notification is only generated if a value is in range but the previous value was not.
Percentage true|false

    If set to true, the minimum and maximum values given are interpreted as percentage value, relative to the other data sources. This is helpful for example for the "df" type, where you may want to issue a warning when less than 5 % of the total space is available. Defaults to false.
Hits Value

    Sets the number of occurrences which the threshold must be raised before to dispatch any notification or, in other words, the number of Intervals that the threshold must be match before dispatch any notification.
Hysteresis Value

    Sets the hysteresis value for threshold. The hysteresis is a method to prevent flapping between states, until a new received value for a previously matched threshold down below the threshold condition (WarningMax, FailureMin or everything else) minus the hysteresis value, the failure (respectively warning) state will be keep.
Interesting true|false

    If set to true (the default), a notification with severity FAILURE will be created when a matching value list is no longer updated and purged from the internal cache. When this happens depends on the interval of the value list and the global Timeout setting. See the Interval and Timeout settings in collectd.conf(5) for details. If set to false, this event will be ignored.

https://collectd.org/wiki/index.php/Notifications_and_thresholds
-->



<!--
## 脚本

### Bash脚本

https://collectd.org/wiki/index.php/Plugin:Exec
-->

## 常用插件

### tail

主要用于日志文件。



## 参考

关于告警可以参考 [Collectd Thresholds and alerting](https://www.netways.de/fileadmin/images/Events_Trainings/Events/OSMC/2015/Slides_2015/collectd_Thresholds_Plugin_and_Icinga_-_Florian_Forster.pdf) 。

<!--
http://www.linuxhowtos.org/manpages/5/collectd.conf.htm
https://collectd.org/wiki/index.php/Chains
-->


{% highlight text %}
{% endhighlight %}
