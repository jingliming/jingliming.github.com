---
title: Linux 常用网络命令
layout: post
comments: true
language: chinese
category: [linux, network]
keywords: hello world,示例,sample,markdown
description: 简单记录一下一些与 Markdown 相关的内容，包括了一些使用模版。
---


<!-- more -->

## tcpdump

tcpdump 会根据表达式 (expression) 来决定是否过滤报文，如果满足条件则会捕获报文，如果没有给出任何条件则会将所有的报文捕获；tcpdump 命令参数如下：

{% highlight text %}
tcpdump [ -AbdDefhHIJKlLnNOpqRStuUvxX ] [ -B buffer_size ] [ -c count ]
        [ -C file_size ] [ -G rotate_seconds ] [ -F file ]
        [ -i interface ] [ -j tstamp_type ] [ -m module ] [ -M secret ]
        [ -r file ] [ -s snaplen ] [ -T type ] [ -w file ]
        [ -W filecount ]
        [ -E spi@ipaddr algo:secret,...  ]
        [ -y datalinktype ] [ -z postrotate-command ] [ -Z user ]
        [ expression ]

参数详见：
  -n           : 不转换地址；-nn 不转换地址和端口号。
  -i interface : 指明要监听的介面，如eth0、lo等，any表示所有的，可以通过tcpdump -D查看对应的编号。
  -X           : 显示ASCII和十六进制。
  -XX          : 同-X，但同时显示包的头部信息。
  -S           : 使用绝对地址而非相对地址显示序列号。
  -s NUM       : 默认96字节的数据，0(或1514)表示捕获所有数据。
  -v -vv -vvv  : 显示详细信息，对应不同等级。
  -c           : 获取一定数量的包后停止。
  -e           : 同时获取ethernet header。
  -q           : 减少协议信息的显示。
  -E           : 通过提供的密码解密IPSEC包。
  -w/r         : 写入或者读取文件。

在表达式 (expression) 中通常有如下几种关键字:
  类型
    host: 指明是一台主机，如host 192.168.9.100。
    net: 捕获与网络地址的通讯包，如net 192.168.0.0/16。
    port: 指明是端口号，如port 21；也可以指定范围 portrange 21-23。

  传输方向
    src: 源地址，如src 192.168.9.100。
    dst: 目的地址，如dst 192.168.9.100。
    dst and src: 是目的地址且是源地址。
    dst or src(default): 是目的地址或是源地址。
    src|dst port: 指定目的或源端口号。

  协议
    fddi: 在FDDI(分布式光纤数据接口网络)上的特定网络协议，是ether的别名。
    ip arp rarp tcp udp icmp: 指定协议的类型。

  过滤包的大小
    less/greater 32 接受小于或大于32bytes的包，或者< >=等，其它运算符也可以。

  其它
    gateway broadcast less greater
    逻辑运算: 非'not'/'!'，与'and'/'&&'，或'or'/'||'，通过逻辑可以将上述的条件连接起来。
{% endhighlight %}

另外常见的示例举例如下：

<ul><li>
指定主机，可以设定源和目的地，可以为主机名或者 IP 。<br>

{% highlight text %}
tcpdump host 192.168.1.0                                      # 所有进出的数据包
tcpdump host 192.168.1.0 and \(192.168.1.2 or 192.168.1.3 \)  # 截获0和2或3之间的通讯
tcpdump ip host foo and ! bar                                 # 截获foo和非bar之间的通讯，!可换为not
tcpdump src host hostname                                     # 主机发送的数据
tcpdump dst host hostname                                     # 发送到主机的数据
{% endhighlight %} </li><li>

指定端口，协议，协议可以为tcp/udp<br>

{% highlight text %}
tcpdump tcp port 23 host 192.168.1.0
{% endhighlight %} </li><li>

截获ftp发送的信息，获取用户名和密码<br>

{% highlight text %}
tcpdump -i lo -nn -X port 21
{% endhighlight %} </li><li>

从192.168网络法往10或172.16网络的数据包<br>

{% highlight text %}
tcpdump -nvX src net 192.168.0.0/16 and dst net 10.0.0.0/8 or 172.16.0.0/16
{% endhighlight %} </li><li>

从172.16网络发往192.168.0.2的非ICMP包<br>

{% highlight text %}
tcpdump -nvvXSs 1514 dst 192.168.0.2 and src net 172.16.0.0/16 and not icmp
{% endhighlight %}
</li></ul>

对于复杂的条件可以使用 ```()```，终端中需要```\( \)```转义，或者用 ```''``` 包裹，还可指定标志如 ACK 。

### 高级选项

常见的，只想监控 TCP 的三次握手建立链接以及四次握手断开连接，也就是说只需要捕获 TCP 控制包，如 SYN、ACK 或 FIN 标记相关的包。

基于 libpcap 的 tcpdump 支持标准的包过滤规则，如基于包头的过滤，也就是基于源目的 IP 地址、端口和协议类型，而且也支持更多通用分组表达式。包中的任意字节范围都可以使用关系或二进制操作符进行检查；对于字节范围表达，可以使用以下格式：

{% highlight text %}
proto [ expr : size ]
参数详解：
  proto: 常见的协议之一，如 ip、arp、tcp、udp、icmp、ipv6
  expr : 与指定的协议头开头相关的字节偏移量，如直接偏移量(tcpflags)、取值常量(tcp-syn、tcp-ack、tcp-fin)
  size : 可选，从字节偏移量开始检查的字节数量
{% endhighlight %}

常见的场景：

{% highlight text %}
# tcpdump -i eth0 "tcp[tcpflags] & (tcp-syn) != 0"           // 只捕获TCP SYNC包
# tcpdump -i eth0 "tcp[13] & (2) != 0"                       // 同上

# tcpdump -i eth0 "tcp[tcpflags] & (tcp-ack) != 0"           // 只捕获TCP ACK包
# tcpdump -i eth0 "tcp[tcpflags] & (tcp-fin) != 0"           // 只捕获TCP FIN包
# tcpdump -i eth0 "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"   // 只捕获TCP SYN或ACK包
{% endhighlight %}

可见 tcpflags==13，其它的还有 SYNCHRONIZE(2)、ACKNOWLEDGE(16)、URGENT(32)、PUSH(8)、RESET(4)、FINISH(1)，如果是捕获 SYNC+ACK(18) 。

<!--
Traffic with the ‘Evil Bit’ Set
# tcpdump ‘ip[6] & 128 != 0‘
通过如下方式方便记忆。
Unskilled Attackers Pester Real Security Folks
Unskilled = URG
Attackers = ACK
Pester = PSH
Real = RST
Security = SYN
Folks = FIN
-->










{% highlight text %}
{% endhighlight %}
