---
title: ICMP Ping
layout: post
comments: true
language: chinese
category: [network]
keywords: network,icmp,ping
description:
---


<!-- more -->

<!--
![nmap logo]({{ site.url }}/images/network/nmap-logo.jpg "nmap logo"){: .pull-center width="50%" }









Ping result

tcpdump -nni eth0 icmp

----- ICMP echo request
tcpdump -nni eth0 -e icmp[icmptype] == 8

----- ICMP echo reply
tcpdump -nni eth0 -e icmp[icmptype] == 0

----- 保存到文件中
tcpdump -nni eth0 -w /tmp/icmp.pcap -e icmp[icmptype] == 0

----- 读取文件
tcpdump -nn -r icmp.pcap


systemctl restart siteagent
bin/siteagentctl config setlog debug0




## struct msghdr

常用于 socket 的发送接收消息，函数声明如下。

#include <sys/types.h>
#include <sys/socket.h>

ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);

#include <sys/socket.h>

struct iovec {                   /* Scatter/gather array items */
   void  *iov_base;              /* Starting address */
   size_t iov_len;               /* Number of bytes to transfer */
};

struct msghdr {
   void         *msg_name;       /* optional address */
   socklen_t     msg_namelen;    /* size of address */
   struct iovec *msg_iov;        /* scatter/gather array */
   size_t        msg_iovlen;     /* # elements in msg_iov */
   void         *msg_control;    /* ancillary data, see below */
   size_t        msg_controllen; /* ancillary data buffer len */
   int           msg_flags;      /* flags on received message */
};

看起来比较复杂，实际上结构成员可以分为四组：

* 套接口地址成员 msg_name 与 msg_namelen；
* IO向量引用 msg_iov 与 msg_iovlen；
* 附属数据缓冲区成员 msg_control 与 msg_controllen；
* 接收信息标记位 msg_flags，详细可以查看 man 手册。

其中 msg_control 指向的是一个 struct cmsghdr 结构体。

https://blog.csdn.net/u014209688/article/details/71311973
https://www.cnblogs.com/jimodetiantang/p/9190958.html
https://ivanzz1001.github.io/records/post/linux/2017/11/04/linux-msghdr



GitHub 上不错的Hack技巧
ICMP Tunnel

### Socket 缓冲区

Linux 中可以设置 socket 缓冲区的大小，貌似对于 RAW_SOCKET 和 TCP 等都有效，没有从代码上确认。

在使用 RAW_SOCKET 时，例如发送 ICMP 报文，因为不存在 TCP 中的滑动窗口、限流等机制，在流量过大时极易引起报文在缓冲区的静默丢失。此时可以通过 tcpdum 获取，但是通过 read、recvmsg 等系统接口无法接收到数据。

接收缓冲区的大小可以使用 `setsockopt()` 设置 `SO_RCVBUF` 选项，其默认和最大值可以通过如下命令查看。

$ cat /proc/sys/net/core/rmem_default
$ cat /proc/sys/net/core/rmem_max

$ cat /proc/sys/net/core/wmem_default
$ cat /proc/sys/net/core/wmem_max

在设置 Socket 的时候，如果期望设置的缓存大于上述的 max ，实际上会得到两倍于 max 值。

### TCP 缓冲区

其中 TCP 的读写缓存可以通过如下方式查看，因为可以通过协议自身可以进行限流，那么由于缓存不足导致丢包的概率极小，只是效率问题：

$ cat /proc/sys/net/ipv4/tcp_rmem
4096	87380	   6291456
4*1024  85*1024    6*1024*1024

$ cat /proc/sys/net/ipv4/tcp_wmem
4096	16384	   4194304
4*1024  16*1024    4*1024*1024

$ cat /proc/sys/net/ipv4/tcp_mem
88371	117831	176742


其中对应了三个数值，分别为最小、默认、最大值。

##

在大多数的 Linux 中 rmem_max 和 wmem_max 被分配的值为 128 k，在一个低延迟的网络环境中，一般是足够用的，对于负载和延迟较高的网络，就需要调整内存使用方法。

详细可以参考
https://segmentfault.com/a/1190000000473365
https://my.oschina.net/guol/blog/115837

https://jvns.ca/blog/2016/08/24/find-out-where-youre-dropping-packets/
https://jvns.ca/blog/2017/09/05/finding-out-where-packets-are-being-dropped/

http://veithen.github.io/2014/01/01/how-tcp-backlog-works-in-linux.html

sysctl net.core.rmem_max

sysctl net.core.rmem_max=16777216
sysctl net.core.rmem_default=16777216
sysctl net.core.wmem_max=8388608
sysctl net.core.wmem_default=8388608

echo 'net.core.wmem_max=8388608' >> /etc/sysctl.conf
echo 'net.core.rmem_max=16777216' >> /etc/sysctl.conf

int rc, optval;
int optlen = sizeof(optval);

rc = getsockopt(fd, SOL_SOCKET, SO_RCVBUF, &optval, &optlen);
printf("xxxxxxxxxxxx %d %d\n", rc, optval);

int sbuf = 1024 * 1024 * 8;
err = setsockopt(fd, SOL_SOCKET, SO_RCVBUF, &bufsize, sizeof(bufsize));

echo 8388608 > /proc/sys/net/core/rmem_default


















称之为应用、服务开发的十二个关注点
https://12factor.net/

Python 的双因子验证
http://blog.51cto.com/cwtea/2068137
https://www.secpulse.com/archives/4606.html
http://www.freebuf.com/articles/network/150071.html
http://www.freebuf.com/articles/web/165139.html


/post/mysql-core-file.html

在 CentOS 中，对应的 debuginfo 信息可以直接从 [CentOS DebugInfo](http://debuginfo.centos.org/7/x86_64/) 。

program-c-complie-link.html
http://www.cirosantilli.com/elf-hello-world/

## Hash Table
允许多线程读写的并发Hash库，以及murmur3
https://github.com/efficient/libcuckoo
https://github.com/savoirfairelinux/opendht

非加密 Hash 算法使用比较多的是 [xxHash](https://github.com/Cyan4973/xxHash)、


IP 报文头为 20 字节 [IP头部结构详解](http://codingstone.com/content.php?blockTableName=network&blogID=2) 。

ICMP 报文头根据 type 和 code 的不同，其对应的大小也有所区别，详见 [使用Python的Socket模块构建一个UDP扫描工具](http://www.cnnetsec.com/2308.html) 。
关于 ICMP 也可以参考 http://courses.cs.vt.edu/cs4254/fall04/slides/raw_6.pdf


ICMP 报文格式
http://www.cnblogs.com/jingmoxukong/p/3811262.html

Ping 使用的是 ICMP 报文，其报头为 8 字节，数据报长度最大为 64K 字节。


ping4_run()
 |-ping4_parse_reply()
 | |-is_ours() 对于ICMP_ECHOREPLY返回类型
 | |-gather_statistics()
 |
 |  其它类型
 |

type of service (ToS


对于通过 SOCK_RAW + ICMP 实现套接字，会接收到所有发送到本机的报文。

1. Destination Host Unreachable

* 校验和算法，把被校验的数据 16 位进行累加，若数据字节长度为奇数，则数据尾部补一个字节的 0 以凑成偶数，然后取反码，。

此算法适用于IPv4、ICMPv4、IGMPV4、ICMPv6、UDP和TCP校验和，更详细的信息请参考RFC1071，校验和字段为上述ICMP数据结构的icmp_cksum变量。
标识符�D�D用于唯一标识ICMP报文, 为上述ICMP数据结构的icmp_id宏所指的变量。
顺序号�D�Dping命令的icmp_seq便由这里读出，代表ICMP报文的发送顺序，为上述ICMP数据结构的icmp_seq宏所指的变量。



setsockopt()
* SOL_SOCKET, SO_BINDTODEVICE  绑定到某个设备上。
* SOL_SOCKET, SO_MARK 用来添加标记。
* SOL_SOCKET, SO_TIMESTAMP 让协议栈接受到一个网络帧时为其打上时间戳，并将此时间戳作为一笔附加数据，与网络帧数据一起递交到上层协议。

/post/network-netfilter-iptables.html

### Mark 标记

用于将特定的数据包打上标签，供 iptables 配合 TC 做 QOS 流量限制、应用策略路由。

系统可用模块，对 CentOS 而言可通过 `ls /usr/lib64/xtables/ | grep -i mark` 查看，其中大写的为标记模块，小写的为匹配模块。

ls /usr/lib/iptables/|grep -i mark

----- 查看关于Mark标记的帮助信息
# iptables -j MARK --help
# iptables -m mark --help

----- 将所有TCP数据标记1
# iptables -t mangle -A PREROUTING -p tcp -j MARK --set-mark 1

----- 匹配标记1的数据并保存数据包中的MARK到连接中
# iptables -t mangle -A PREROUTING -p tcp -m mark --mark 1 -j CONNMARK --save-mark


#### 策略路由

标签并不是设置在数据包内容中，而是在内核中数据包的载体上，如果需要在数据包内容中设置标签，可以使用 TOS 规则目标，也就是修改 IP 数据包头的 TOS 值。

----- 将从网络接口tun0进入的、目标端口为5222的TCP数据包设置mark值为1
# iptables -t mangle -A PREROUTING -j MARK --set-mark 1 -i tun0 -p tcp --dport 5222

接着，设置的 mark 值可用来设定策略路由，比如，把 mark 值为 1 的数据包交由网关 192.168.0.1 转发。

----- 1. 确定一张空路由表，这里选定300
# ip route show table 300
----- 2. 在表中添加路由条目
# ip route add default via 192.168.0.1 table 300
----- 3. 查看当前路由规则
# ip rule list
----- 4. 为mark值为1的数据包指定路由表策略
# ip rule add fwmark 0x1 table 300

通过这种方法，可以使用 iptables 根据匹配规则设置 mark，再由路由模块根据 mark 值进行路由决策，从而实现复杂的策略路由。

switch (icmp_hdr->icmp_type) {
case ICMP_ECHOREPLY:
        ident = ntohs(icmp_hdr->icmp_id);
        seq   = ntohs(icmp_hdr->icmp_seq);
        break;

case ICMP_ECHO: /* Maybe send by another ping tools */
        return NULL;

case ICMP_DEST_UNREACH:    /*  3 */
case ICMP_SOURCE_QUENCH:   /*  4 */
case ICMP_REDIRECT:        /*  5 */
case ICMP_TIME_EXCEEDED:   /* 11 */
case ICMP_PARAMETERPROB: { /* 12 */
        struct iphdr *iph = (struct iphdr *)(buffer + sizeof(struct icmphdr));
        struct icmphdr *icmph = (struct icmphdr *)(buffer +
                        (sizeof(struct icmphdr) + iph->ihl * 4));

        if (buffer_len < sizeof(struct iphdr) + 2 * ICMP_MINLEN ||
                        buffer_len < (size_t)iph->ihl * 4 + 2 * ICMP_MINLEN) {
                log_error(LIBNAME "error package too short, len %d, IP header %d",
                                buffer_len, iph->ihl);
                return NULL;
        }

        if (icmph->type != ICMP_ECHO) {
                log_error(LIBNAME "invalid type %d(expect %d) for %d",
                                icmph->type, ICMP_ECHO, icmp_hdr->icmp_type);
                return NULL;
        }

        ident = ntohs(icmph->un.echo.id);
        seq   = ntohs(icmph->un.echo.sequence);
        status = (icmp_hdr->icmp_type << 8) + icmp_hdr->icmp_code;

        break;
}
default:
        log_warning(LIBNAME "IPv4 unexpected ICMP type 0x%02x", icmp_hdr->icmp_type);
        return NULL;
}
-->

{% highlight text %}
{% endhighlight %}
