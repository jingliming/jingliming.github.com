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


## Netcat

Netcat 用于调试、检查网络的工具包，可用于创建 TCP/IP 连接，最大的用途就是用来处理 TCP UDP 套接字。在网络安全领域被称作 "TCP IP 的瑞士军刀" (Swiss-army knife for TCP/IP)。


<!--
2.2.1  网络连接
　　类似于Telnet的功能，使用Netcat能够简便地登录到目标机上开放的端口。
　　例如，
nc mail.server.net 25
　　向mail.server.net的25号TCP端口发起连接。

2.2.2  端口扫描
　　Netcat同样可以进行端口扫描，但与Nmap相比，它的性能及使用范围都很有限。如果只想简单地探测几个端口的开放状态，使用Netcat也可行。
　　端口扫描格式如下：
nc –v –z hostnameport[s]
　　实例：
nc –v –z www.yahoo.com 80 22
　　扫描yahoo网站的80、22端口，判断其是否开放。

2.2.3  文件传输
　　Netcat最初的用途就是文件传输，它可以像Cat命令一样将读取的文件重定向到网络上的另外的文件。Netcat在网络应用中既可以当做服务器端，开启本机一个监听端口，也可以作为客户端向其他服务器端口发起连接。所以，文件传输，即是在两端分别运行Netcat。
　　在接受端，运行Netcat开启端口监听服务。
nc –L –p 4444 >receivedfile.zip
　　此处-L启动监听模式（即作为服务器端），开启4444端口，将接受到的数据写入到文件somefile.zip中。
　　而在发送端，只需连接该服务器端开放的端口，并选择需要发送的文件。
nc 192.168.1.43 4444< testfile.zip
　　使用<符号输入testfile.zip文件，并向目标机的4444端口发起连接。在建立连接成功后，发送端会将文件传送到接收端，接受端将收到的数据写入receivedfile.zip文件。整个过程，比较类似于cat命令，因为具体的网络传输过程由Netcat负责。

2.2.4  开启后门
　　Netcat甚至也可以用作后门程序。如果用户已经侵入到一台计算机，那么让该计算机在开机后（或其他条件下）自动启动Netcat，打开指定的端口，等待用户连接，在连接成功后执行特定程序（如cmd.exe，以便远程执行命令）。
nc –L –p 4444 –e cmd.exe
　　以监听模式启动Netcat，开启TCP端口4444，在与客户端成功建立连接后，执行cmd.exe程序（-e cmd.exe，此处为用户打开命令行执行窗口，用户可以通过命令操作此计算机）。
　　在客户端，直接连接目标机的4444端口即可。
nc 192.168.1.43 4444
　　连接后，客户端可以接收到一个命令行窗口。

2.2.5  端口转发
　　端口转发（PortForwarding）也是Netcat比较实用的用法。先将Netcat作为服务器接收其他主机的连接，然后将连接的数据转发另外的目标机端口。
　　比如：
mkfifo backpipe
nc -l 12345  0<backpipe | nc www.google.com 801>backpipe
　　比如，此处开启端口12345，作为www.google.com的代理。其他无法直接登陆google的用户可以通过此代理端口来与google进行交互。这里创建了一个fifo，是为实现双向数据通讯，因为管道运算符本身是单向的。

2.2.6  标语提取
　　标语提取（BannerGrabbing）的含义是抓取应用程序在建立连接后打印的标语提示信息，例如建立FTP连接后，FTP服务器可能打印出提示信息：FTP xxx.xxx等数据。
　　所以，根据服务器打印的信息，有时可以推断出对方服务程序的详细版本。这也是Nmap进行服务与版本侦测采用方法。
　　例如，首先创建一份文件，包含以下文本：
[plain] view plaincopy
HEAD / HTTP/1.0
<return>
<return>
　　然后，将此文件发送到目标服务器的80端口，诱发对方发送HTTP首部数据。
cat file>nc -vv -w 2 www.cnn.com 80 >output.txt
　　然后可从output.txt查看到对方的发送的HEAD的标语信息。


2.2.7  其他功能
　　Netcat其他常用的功能：
支持完全的DNS转发、逆向检查
支持用户指定源端口
支持用户指定源端IP地址
内置宽松源路由能力（loosesource-routing capability）
慢速发送模式，可指定每隔多少秒发送一行文本
将发送或接收数据以16进制格式导出


<pre style="font-size:0.8em; face:arial;">
-k     设置keepalive选项


root@10.1.1.43:~# nc -h
[v1.10-38]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound: nc -l -p port [-options] [hostname] [port]
options:
    -c shell commands   as `-e'; use /bin/sh to exec [dangerous!!]
    -e filename     program to exec after connect [dangerous!!]
    -b          allow broadcasts
    -g gateway      source-routing hop point[s], up to 8                          设置路由器跃程通信网关，最高可设置8个。
    -G num          source-routing pointer: 4, 8, 12, ...                         设置来源路由指向器，其数值为4的倍数。
    -h          this cruft
    -i secs         delay interval for lines sent, ports sca                      延时的间隔
    -l          listen mode, for inbound connects                             监听模式,入站连接
    -n          numeric-only IP addresses, no DNS                             直接使用ip地址,而不用域名服务器
    -o file         hex dump of traffic                                           指定文件名称，把往来传输的数据以16进制字码倾倒成该文件保存。
    -p port         local port number                                             本地端口
    -r          randomize local and remote ports                              随机本地和远程端口
    -q secs         quit after EOF on stdin and delay of secs
    -s addr         local source address
    -T tos          set Type Of Service
    -t          answer TELNET negotiation
    -u          UDP mode                                                      udp  模式
    -v          verbose [use twice to be more verbose]                        显示过程,vv 更多
    -w secs         timeout for connects and final net reads                      等待连接超时
    -z          zero-I/O mode [used for scanning]                             使用输入/输出模式，只在扫描通信端口时使用。
</pre>
-->



### 常见示例

在此简单列举下常见的示例。

##### 运行于服务器模式，侦听指定端口

默认使用 ipv4(-4) ，可以显示使用 ipv6(-6)；默认当客户端退出后，服务端同时也会退出，可以通过 -k 防止服务端退出；通过 -u 指定是 UDP 连接。

{% highlight text %}
$ nc -l 2389                                     // 创建服务端，监听2389端口
$ nc 127.1 2389                                  // 使用客户端模式来连接到2389端口
{% endhighlight %}

##### 端口扫描

{% highlight text %}
$ nc -v -w 10 10.1.1.180 80
(UNKNOWN) [10.1.1.180] 80 (www) open

$ nc -v -w 10 10.1.1.180 -z 80-30000
(UNKNOWN) [10.1.1.180] 22000 (?) open
(UNKNOWN) [10.1.1.180] 80 (www) open
{% endhighlight %}

##### 传输文件/文件夹

将客户端的 testfile 传输到服务器端的 test，可以是文本文件，也可以是二进制文件。对于目录需要通过 tar 打包，然后再发送。

{% highlight text %}
$ nc -l 2389 > test                              // 服务器端将文件重定向到test
$ cat testfile | nc 127.1 2389                   // 客户端传输文件，或者如下
$ nc 127.1 2389 &lt; testfile

$ tar -cvf -- DIR-NAME | nc -l 1567              // 打包目录
$ nc -n 172.1 1567 | tar -xvf -
$ tar -cvf -- DIR-NAME | bzip2 -z | nc -l 1567   // 同时进行压缩，减小带宽的使用
$ nc -n 172.1 1567 | bzip2 -d |tar -xvf -
{% endhighlight %}

##### 加密传输

{% highlight text %}
$ nc 127.1 1567 | mcrypt --flush --bare -F -q -d -m ecb > file.txt   // 在此使用mcrypt工具加密数据
$ mcrypt --flush --bare -F -q -m ecb < file.txt | nc -l 1567
{% endhighlight %}

以上两个命令会提示需要密码，确保两端使用相同的密码，也可以使用其它任意加密工具。

##### 视频流

虽然不是生成流视频的最好方法，但如果服务器上没有特定的工具，使用 netcat 。

{% highlight text %}
$ cat video.avi | nc -l 1567
$ nc 172.31.100.7 1567 | mplayer -vo x11 -cache 3000 -
{% endhighlight %}

##### 克隆一个设备

如果已经安装配置一台 Linux 机器并且需要重复同样的操作对其他的机器，而你不想在重复配置一遍。

{% highlight text %}
$ dd if=/dev/sda | nc -l 1567             // 假设系统在磁盘/dev/sda上
$ nc -n 172.1 1567 | dd of=/dev/sda       // 复制到其它机器
{% endhighlight %}

如果已经做过分区并且只需要克隆 root 分区，可以根据系统 root 分区的位置，更改 sda 为 sda1，sda2 等等。

<!--
超时控制????<br>
多数情况我们不希望连接一直保持，那么我们可以使用 -w 参数来指定连接的空闲超时时间，该参数紧接一个数值，代表秒数，如果连接超过指定时间则连接会被终止。
<pre style="font-size:0.8em; face:arial;">
$ nc -l 2389
$ nc -w 10 localhost 2389                // 该连接将在 10 秒后中断。
</pre>
注意: 不要在服务器端同时使用 -w 和 -l 参数，因为 -w 参数将在服务器端无效果。
-->








{% highlight text %}
{% endhighlight %}
