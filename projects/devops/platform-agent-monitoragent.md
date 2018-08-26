---
title: MonitorAgent 实现
layout: project
comments: true
language: chinese
category: [misc]
keywords:
description:
---

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


{% highlight text %}
{% endhighlight %}
