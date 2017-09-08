---
title: Linux 常用网络命令
layout: post
comments: true
language: chinese
category: [linux, network]
keywords: linux,network,tcpdump,netcat
description: 简单记录一下 Linux 中的常用网络命令，如 tcpdump、netcat 等。
---

简单记录一下 Linux 中的常用网络命令，如 tcpdump、netcat 等。

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

在 CentOS 中，可以通过如下方式安装。

{% highlight text %}
----- CentOS 6版本安装
# yum install nc

----- CentOS 7版本安装
# yum install nmap-ncat
{% endhighlight %}

常用参数简单列举如下。

{% highlight text %}
常用参数：
  -k      保持打开，默认只监听一个客户端，如果某一端退出则退出
  -l      监听模式，用于接收链接请求，默认客户端退出后服务同时退出
  -n      直接使用IP地址，不做DNS解析
  -v      用于显示更详细的信息，可以使用多个
  -u      UDP模式
  -w secs 连接超时
  -t      使用TELNET协议
  -e file 链接后执行的命令
{% endhighlight %}


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
----- 扫描10.1.1.180主机上的80或者1~14000号TCP端口
$ nc -v -z -w2 10.1.1.180 80
(UNKNOWN) [10.1.1.180] 80 (www) open
$ nc -v -z -w2 10.1.1.180 1-14000

----- 扫描10.1.1.180主机上的1~14000号UDP端口
$ nc -u -v -z -w2 10.1.1.180 1-14000
{% endhighlight %}

##### REMOTE主机绑定SHELL

{% highlight text %}
----- 绑定到5354端口
$ nc -l -p 5354 -t -e c:\winnt\system32\cmd.exe
$ nc -l -p 5354 -t -e /bin/bash

----- 链接到服务器，可以执行Bash命令
$ nc 10.1.1.180 5354
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

##### 超时控制
多数情况我们不希望连接一直保持，那么我们可以使用 -w 参数来指定连接的空闲超时时间，该参数紧接一个数值，代表秒数，如果连接超过指定时间则连接会被终止。

{% highlight text %}
$ nc -l 2389
$ nc -w 10 localhost 2389                // 该连接将在 10 秒后中断。
{% endhighlight %}

注意: 不要在服务器端同时使用 -w 和 -l 参数，因为 -w 参数将在服务器端无效果。


## curl

curl 用于向服务器传输数据，它支持 http、https、ftp、ftps、scp、sftp、tftp、telnet 等协议。

### 常见示例


{% highlight text %}
----- 查看源码，保存到某个文件；默认输出到终端
$ curl -o filename www.sina.com

----- 自动调转到新地址，会自动跳转为www.sina.com.cn
$ curl -L www.sina.com

----- 显示头信息以及网页代码，如果只查看头则可以使用-I参数
$ curl -i www.sina.com

----- 显示通信过程，通过-v可显示http通信的整个过程，包括端口连接和http request头信息
$ curl -v www.sina.com                             # 显示交互信息
$ curl --trace output.txt www.sina.com             # 显示更加信息的信息
$ curl --trace-ascii output.txt www.sina.com

----- 发送GET表单
$ curl www.example.com/form.cgi?data=xxx

----- 发送POST表单信息，需要把数据和网址分开，也就是--data参数，通过第一个参数对表单编码
$ curl --data-urlencode --data "data=April 1" www.example.com/form.cgi
{% endhighlight %}


<!--
六、文件上传

假定文件上传的表单是下面这样：
<form method="POST" enctype='multipart/form-data' action="upload.cgi">
　　<input type=file name=upload>
　　<input type=submit name=press value="OK">
</form>

你可以用curl这样上传文件：
1

curl --form upload=@localfilename --form press=OK <a class="CODE" name="xxx"www.example.com</pre" href="http://www.jobbole.com/entry.php/1370"></a>

七、Referer字段

有时你需要在http request头信息中，提供一个referer字段，表示你是从哪里跳转过来的。
1

curl --referer http://www.example.com http://www.example.com

八、User Agent字段

这个字段是用来表示客户端的设备信息。服务器有时会根据这个字段，针对不同设备，返回不同格式的网页，比如手机版和桌面版。

iPhone4的User Agent是
1

Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_0 like Mac OS X; en-us)  AppleWebKit/532.9 (KHTML, like Gecko) Version/4.0.5 Mobile/8A293  Safari/6531.22.7

curl可以这样模拟：
1

curl --user-agent "[User Agent]" [URL]

九、cookie

使用–cookie参数，可以让curl发送cookie。
1

curl --cookie "name=xxx"www.example.com

至于具体的cookie的值，可以从http response头信息的Set-Cookie字段中得到。

十、增加头信息

有时需要在http request之中，自行增加一个头信息。–header参数就可以起到这个作用。
1

curl --header "xxx: xxxxxx" http://example.com

十一、HTTP认证

有些网域需要HTTP认证，这时curl需要用到–user参数。
1

curl --user name:password example.com


如何使用curl

curl并不是双击即可运行，你需要在命令提示符下使用它
如何进入命令提示符
点击“开始”——“运行”——输入CMD

或，WIN+R，输入CMD

下载我下载版本的CURL后，解压后文件夹放在如：D:\curl的文件夹里（curl路径）

命令提示符窗口中输入"d:"回车，然后输入“cd curl”即可跳转到curl文件夹，

至此可以输入curl命令了(根据你的curl类推)或你也可以将curl加入到系统环境变量如何将curl所在文件夹加入系统变量
右键单击“我的电脑”——“属性”——“高级”——“环境变量”——

“系统变量”——“Path”——“编辑”——加入“;D:\curl”(注意分号，

D:\curl换成你的curl路径)——“确定”加入到系统环境变量后可在命令提示符内直接运行如

“curl -O http://curl.haxx.se/download/curl-7.19.5-win32-ssl-sspi.zip”

这样的命令，无需进入curl所在文件夹curl命令用法

一个不错的Curl教程
1)
二话不说，先从这里开始吧！
curl http://www.yahoo.com

回车之后，www.yahoo.com 的html就稀里哗啦地显示在屏幕上了~~~~~

2)
嗯，要想把读过来页面存下来，是不是要这样呢？
curl http://www.yahoo.com > page.html

当然可以，但不用这么麻烦的！
用curl的内置option就好，存下http的结果，用这个option: -o
curl -o page.html http://www.yahoo.com

这样，你就可以看到屏幕上出现一个下载页面进度指示。等进展到100%，自然就OK咯

3)
什么什么？！访问不到？肯定是你的proxy没有设定了。
使用curl的时候，用这个option可以指定http访问所使用的proxy服务器及其端口： -x
curl -x 123.45.67.89:1080 -o page.html http://www.yahoo.com


4)
访问有些网站的时候比较讨厌，他使用cookie来记录session信息。
像IE/NN这样的浏览器，当然可以轻易处理cookie信息，但我们的curl呢？.....
我们来学习这个option: -D <-- 这个是把http的response里面的cookie信息存到一个特别的文件中去
curl -x 123.45.67.89:1080 -o page.html -D cookie0001.txt http://www.yahoo.com

这样，当页面被存到page.html的同时，cookie信息也被存到了cookie0001.txt里面了


5）
那么，下一次访问的时候，如何继续使用上次留下的cookie信息呢？要知道，很多网站都是靠监视你的cookie信息，
来判断你是不是不按规矩访问他们的网站的。
这次我们使用这个option来把上次的cookie信息追加到http request里面去： -b
curl -x 123.45.67.89:1080 -o page1.html -D cookie0002.txt -b cookie0001.txt http://www.yahoo.com

这样，我们就可以几乎模拟所有的IE操作，去访问网页了！


6）
稍微等等~~~~~我好像忘记什么了~~~~~
对了！是浏览器信息~~~~

有些讨厌的网站总要我们使用某些特定的浏览器去访问他们，有时候更过分的是，还要使用某些特定的版本~~~~
NND，哪里有时间为了它去找这些怪异的浏览器呢！？

好在curl给我们提供了一个有用的option，可以让我们随意指定自己这次访问所宣称的自己的浏览器信息： -A
curl -A "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)" -x 123.45.67.89:1080 -o page.html -D cookie0001.txt http://www.yahoo.com

这样，服务器端接到访问的要求，会认为你是一个运行在Windows 2000上的IE6.0，嘿嘿嘿，其实也许你用的是苹果机呢！

而"Mozilla/4.73 [en] (X11; U; Linux 2.2; 15 i686"则可以告诉对方你是一台PC上跑着的Linux，用的是Netscape 4.73，呵呵呵


7）
另外一个服务器端常用的限制方法，就是检查http访问的referer。比如你先访问首页，再访问里面所指定的下载页，这第二次访问的 referer地址就是第一次访问成功后的页面地址。这样，服务器端只要发现对下载页面某次访问的referer地址不是首页的地址，就可以断定那是个盗连了~~~~~

讨厌讨厌~~~我就是要盗连~~~~~！！
幸好curl给我们提供了设定referer的option： -e
curl -A "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)" -x 123.45.67.89:1080 -e "mail.yahoo.com" -o page.html -D cookie0001.txt http://www.yahoo.com

这样，就可以骗对方的服务器，你是从mail.yahoo.com点击某个链接过来的了，呵呵呵


8）
写着写着发现漏掉什么重要的东西了！----- 利用curl 下载文件

刚才讲过了，下载页面到一个文件里，可以使用 -o ，下载文件也是一样。
比如， curl -o 1.jpg http://cgi2.tky.3web.ne.jp/~zzh/screen1.JPG
这里教大家一个新的option： -O
大写的O，这么用： curl -O http://cgi2.tky.3web.ne.jp/~zzh/screen1.JPG
这样，就可以按照服务器上的文件名，自动存在本地了！

再来一个更好用的。
如果screen1.JPG以外还有screen2.JPG、screen3.JPG、....、screen10.JPG需要下载，难不成还要让我们写一个script来完成这些操作？
不干！
在curl里面，这么写就可以了：
curl -O http://cgi2.tky.3web.ne.jp/~zzh/screen[1-10].JPG

呵呵呵，厉害吧？！~~~

9）
再来，我们继续讲解下载！
curl -O http://cgi2.tky.3web.ne.jp/~/[001-201].JPG

这样产生的下载，就是
~zzh/001.JPG
~zzh/002.JPG
...
~zzh/201.JPG
~nick/001.JPG
~nick/002.JPG
...
~nick/201.JPG

够方便的了吧？哈哈哈

咦？高兴得太早了。
由于zzh/nick下的文件名都是001，002...，201，下载下来的文件重名，后面的把前面的文件都给覆盖掉了~~~

没关系，我们还有更狠的！
curl -o #2_#1.jpg http://cgi2.tky.3web.ne.jp/~/[001-201].JPG

--这是.....自定义文件名的下载？
--对头，呵呵！

#1是变量，指的是这部分，第一次取值zzh，第二次取值nick
#2代表的变量，则是第二段可变部分---[001-201]，取值从001逐一加到201
这样，自定义出来下载下来的文件名，就变成了这样：
原来： ~zzh/001.JPG --- 下载后： 001-zzh.JPG
原来： ~nick/001.JPG --- 下载后： 001-nick.JPG

这样一来，就不怕文件重名啦，呵呵

9）
继续讲下载
我们平时在windows平台上，flashget这样的工具可以帮我们分块并行下载，还可以断线续传。
curl在这些方面也不输给谁，嘿嘿

比如我们下载screen1.JPG中，突然掉线了，我们就可以这样开始续传
curl -c -O http://cgi2.tky.3wb.ne.jp/~zzh/screen1.JPG

当然，你不要拿个flashget下载了一半的文件来糊弄我~~~~别的下载软件的半截文件可不一定能用哦~~~

分块下载，我们使用这个option就可以了： -r
举例说明
比如我们有一个http://cgi2.tky.3web.ne.jp/~zzh/zhao1.mp3 要下载（赵老师的电话朗诵 :D ）
我们就可以用这样的命令：
curl -r 0-10240 -o "zhao.part1" http:/cgi2.tky.3web.ne.jp/~zzh/zhao1.mp3 &\
curl -r 10241-20480 -o "zhao.part1" http:/cgi2.tky.3web.ne.jp/~zzh/zhao1.mp3 &\
curl -r 20481-40960 -o "zhao.part1" http:/cgi2.tky.3web.ne.jp/~zzh/zhao1.mp3 &\
curl -r 40961- -o "zhao.part1" http:/cgi2.tky.3web.ne.jp/~zzh/zhao1.mp3

这样就可以分块下载啦。
不过你需要自己把这些破碎的文件合并起来
如果你用UNIX或苹果，用 cat zhao.part* > zhao.mp3就可以
如果用的是Windows，用copy /b 来解决吧，呵呵

上面讲的都是http协议的下载，其实ftp也一样可以用。
用法嘛，
curl -u name:passwd ftp://ip:port/path/file
或者大家熟悉的
curl ftp://name:passwd@ip:port/path/file



10)
说完了下载，接下来自然该讲上传咯
上传的option是 -T

比如我们向ftp传一个文件： curl -T localfile -u name:passwd ftp://upload_site:port/path/

当然，向http服务器上传文件也可以
比如 curl -T localfile http://cgi2.tky.3web.ne.jp/~zzh/abc.cgi
注意，这时候，使用的协议是HTTP的PUT method

刚才说到PUT，嘿嘿，自然让老服想起来了其他几种methos还没讲呢！
GET和POST都不能忘哦。

http提交一个表单，比较常用的是POST模式和GET模式

GET模式什么option都不用，只需要把变量写在url里面就可以了
比如：
curl http://www.yahoo.com/login.cgi?user=nickwolfe&password=12345

而POST模式的option则是 -d

比如，curl -d "user=nickwolfe&password=12345" http://www.yahoo.com/login.cgi
就相当于向这个站点发出一次登陆申请~~~~~

到底该用GET模式还是POST模式，要看对面服务器的程序设定。

一点需要注意的是，POST模式下的文件上的文件上传，比如
<form method="POST" enctype="multipar/form-data" action="http://cgi2.tky.3web.ne.jp/~zzh/up_file.cgi">
<input type=file name=upload>
<input type=submit name=nick value="go">
</form>
这样一个HTTP表单，我们要用curl进行模拟，就该是这样的语法：
curl -F upload=@localfile -F nick=go http://cgi2.tky.3web.ne.jp/~zzh/up_file.cgi

罗罗嗦嗦讲了这么多，其实curl还有很多很多技巧和用法
比如 https的时候使用本地证书，就可以这样
curl -E localcert.pem https://remote_server

再比如，你还可以用curl通过dict协议去查字典~~~~~
curl dict://dict.org/d:computer 
-->







{% highlight text %}
{% endhighlight %}
