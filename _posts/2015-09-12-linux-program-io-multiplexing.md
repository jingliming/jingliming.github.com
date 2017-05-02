---
title: Linux IO 多路复用应用
layout: post
comments: true
language: chinese
category: [linux,program]
keywords: linux,io,multiplexing,多路复用
description: 通过 IO 多路复用技术，系统内核缓冲 IO 数据，当某个 IO 准备好后，系统通知应用程序该 IO 可读或可写，这样应用程序可以马上完成相应的 IO 操作，而不需要等待系统完成相应 IO 操作，从而应用程序不必因等待 IO 操作而阻塞。这里简单介绍下 Linux 中 IO 多路复用的使用。
---

通过 IO 多路复用技术，系统内核缓冲 IO 数据，当某个 IO 准备好后，系统通知应用程序该 IO 可读或可写，这样应用程序可以马上完成相应的 IO 操作，而不需要等待系统完成相应 IO 操作，从而应用程序不必因等待 IO 操作而阻塞。

这里简单介绍下 Linux 中 IO 多路复用的使用。

<!-- more -->

## 简介

在 Linux 中，会将很多资源通过文件描述符表示，包括了文件系统、网络、物理设备等等。

文件描述符就是一个整数，默认会选择最小未使用的数值，其中有三个比较固定，比较特殊的值，0 是标准输入，1 是标准输出，2 是标准错误输出。

0、1、2 是整数表示的，对应的 FILE* 结构的表示就是 stdin、stdout、stderr。

## select

select 可以用来监视多个文件句柄的状态变化，程序会阻塞在 select 直到被监视的文件句柄有一个或多个发生了状态改变，然后通知应用程序，应用程序 <font color='blue'>轮询</font> 所有的 FD 集合，判断监控的 FD 是否有事件发生，并作相应的处理。

select 相关的声明和宏，通过宏来设置参数，相应的执行流程如下。

{% highlight text %}
int select(int maxfd, fd_set * readfds, fd_set * writefds, fd_set * exceptfds, struct timeval * timeout);
FD_CLR(inr fd, fd_set* set);                               // 清除描述词组set中相关fd的位
FD_ISSET(int fd, fd_set *set);                             // 用来测试描述词组set中相关fd的位是否为真
FD_SET(int fd, fd_set*set);                                // 用来设置描述词组set中相关fd的位
FD_ZERO(fd_set *set);                                      // 用来清除描述词组set的全部位
struct timeval {
   time_t tv_sec;
   time_t tv_usec;
};

----- 调用流程
fd_set readfso;                                            // 1.1 定义需要监视的描述符
FD_ZERO(&readfso);                                         // 1.2 清空
FD_SET(fd, &readfso);                                      // 1.3 设置需要监控的描述符，如fd=1
while(1) {
    readfs = readfso;
    ret = select(maxfd+1, &readfds, NULL, NULL, timeout);  // 2.1 监听所关注的事件，在此为监听读事件
    if (ret > 0)                                           // 3.1.1 监听事件发生
        for (i = 0; i > FD_SETSIZE; i++)                   // 3.1.2 需要遍历所有fd
            if (FD_ISSET(fd, &readfds))
                handleEvent();
    else if (ret == 0)                                     // 3.2 超时
        handle timeout
    else if (ret < 0)                                      // 3.3 发生错误
        handle error
}
{% endhighlight %}

在 select 函数中，各个参数的含义如下。
* maxfd 为最大文件描述符+1，因此在程序中需要知道最大描述符的值。
* 接着的三个参数分别设置所监听的读、写、异常事件对应的描述符。该值传入需要监控事件的描述符，返回值保存了发生了的事件对应的描述符。
* 设置超时时间，NULL 表示一直等待。

返回值：成功则返回文件描述符状态已改变的总数；如果返回 0 代表在描述词状态改变前已超过 timeout 时间；当有错误发生时则返回 -1，错误原因存于 errno ，此时参数 readfds, writefds, exceptfds 和 timeout 的值变成不可预测。

在有返回值时，需要通过 FD_ISSET 循环遍历各个描述符，从而文件描述符越多，效率也就越低。而且，每次调用 select 时都需要重新设置需要监控的文件描述符。

fd_set 在 ```sys/select.h``` 中定义，大致可以简化为如下的结构，也就是实际用数组表示，其中每一位代表一个文件描述符，最大可以表示 1024 个，也就是说 select 能监视的描述符最大为 1024 。最大可以监控的文件描述符数可以通过 FD_SETSIZE 获得，描述符需要通过对应的宏来配置。

{% highlight c %}
typedef struct {
    long int  __fds_bits[__FD_SETSIZE(1024) / __NFDBITS(8 * sizeof(long int))];
} fd_set;
{% endhighlight %}

## poll

和 select() 不一样，poll() 没有使用低效的三个基于位的文件描述符 set ，而是采用了一个单独的结构体 pollfd 数组，由 fds 指针指向这个数组，采用链表保存文件描述符。

{% highlight text %}
struct pollfd {
    int fd;              /* 文件描述符 */
    short events;        /* 等待的事件 */
    short revents;       /* 实际发生了的事件 */
} ;
typedef unsigned long   nfds_t;
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

struct pollfd fds[MAX_CONNECTION] = {0};
fds[0].fd = 0;
fds[0].events = POLLIN;
ret = poll(fds, sizeof(fds)/sizeof(struct pollfd), -1)
if (ret > 0)
    for (int i = 0; i < MAX_CONNECTION; i++)
        if (fds[0].revents & POLLIN)
            handleEvent(allConnection[i]);
else if (ret == 0)
    handle timeout
else if (ret > 0)
    handle error
{% endhighlight %}

fds 表示需要监控的文件描述符；nfds 表示 fds 的大小，也即需要监控的描述符数量。如果 fd 为负，则忽略 events，revents 返回 0。

返回值与 select 的返回值含义相同。


## epoll

epoll 是 Linux 内核为处理大批量句柄而作了改进的 poll，是 Linux 下多路复用 IO 接口 select/poll 的增强版本，它能显著减少程序在大量并发连接中只有少量活跃的情况下的系统 CPU 利用率。

### 对比

select 模型的缺点：

* 最大并发数限制，因为一个进程所打开的文件描述符是有限制的，由 FD_SETSIZE 设置，默认值是 1024/2048 ，因此 select 模型的最大并发数就被相应限制了。
* 效率问题，每次 select 调用返回都需要线性扫描全部的 FD 集合，确定那个描述符发生了相应事件，这样效率就会呈现线性下降。
* 内核/用户空间内存拷贝问题，在通知用户事件发生时采用内存拷贝。
* 触发方式采用的是水平触发，应用程序如果没有完成对一个已经就绪的文件描述符的 IO 操作，那么之后每次 select 调用还是会将这些文件描述符通知进程。

对于 poll 而言，2 和 3 都没有改掉，相比来说，epoll 要好的多。
* epoll 没有最大并发连接的限制，上限是最大可以打开文件的数目，这个数字一般远大于 2048, 一般来说这个数目和系统内存关系很大 ，具体数目可以在 /proc/sys/fs/epoll/max_user_watches 中查看。
* 效率提升，epoll 最大的优点就在于它只管你“活跃”的连接，而跟连接总数无关，因此在实际的网络环境中，epoll 的效率就会远远高于 select 和 poll。
* 内存拷贝，epoll 在这点上使用了“共享内存”，这个内存拷贝也省略了。mmap 加速内核与用户空间的信息传递，epoll 是通过内核于用户空间 mmap 同一块内存，避免了无谓的内存拷贝。

<!--
select 可以在某一时间监视最大达到 FD_SETSIZE 数量的文件描述符， 通常是由在 libc 编译时指定的一个比较小的数字。
poll 在同一时间能够监视的文件描述符数量并没有受到限制，即使除了其它因素，更加的是我们必须在每一次都扫描所有通过的描述符来检查其是否存在己就绪通知，它的时间复杂度为 O(n) ，是缓慢的。

epoll 没有以上所示的限制，并且不用执行线性扫描。因此， 它能有更高的执行效率且可以处理大数量的事件。

当描述符被添加到epoll实例中， 有两种添加模式： level triggered（水平触发） 和 edge triggered（边沿触发） 。 当使用 level triggered 模式并且数据就绪待读， epoll_wait总是会返加就绪事件。如果你没有将数据读取完， 并且调用epoll_wait 在epoll 实例上再次监听这个描述符， 由于还有数据是可读的，它会再次返回。在 edge triggered 模式时， 你只会得一次就绪通知。 如果你没有将数据读完， 并且再次在 epoll实例上调用 epoll_wait ， 由于就绪事件已经被发送所以它会阻塞。
-->


### 函数接口

epoll 相关的函数主要有如下三种，分别用于创建，注册、修改、删除，等待事件发生。

#### 创建

创建 epoll 文件描述符，```int epoll_create (int size)``` 。

需要注意的是，当创建好 epoll 句柄后，它就是会占用一个 fd 值，在 linux 下如果查看 ```/proc/<PID>/fd/```，是能够看到这个 fd 的，所以在使用完 epoll 后，必须调用 close() 关闭，否则可能导致 fd 被耗尽。

调用成功则返回与实例关联的文件描述符，该文件描述符与真实的文件没有任何关系，仅作为接下来调用的函数的句柄，实际就是申请一个内核空间，用来保存所关注的 fd 上发生的时将。size 为了与之前兼容，应该大于 0 ，现在是动态分配，而非指定支持事件的数目。正常返回值大于 0；发生错误时，返回 -1，errno 表明错误类型。

#### 修改

控制文件描述符，```int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);``` 。

类似于相对于 select 模型中的 FD_SET 和 FD_CLR 宏，用于注册、修改、删除；与 select 不同的是，select 在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。

其中<font color='blue'>参数 epfd 是 epoll_create()</font> 创建 epoll 专用的文件描述符；op 用于表示对应的操作(EPOLL_CTL_ADD、EPOLL_CTL_MOD、EPOLL_CTL_DEL)；fd 表示需要监控的描述符；event 表示需要监控的事件。

#### 等待

等待事件 ```int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);```，内核通过 events 返回发生事件的集合，maxevents 告之内核最多可以返回的事件数，该函数返回需要处理的事件数目，如返回 0 表示已超时。

等待 IO 事件的发生，epfd 是由 epoll_create() 生成的 epoll 专用的文件描述符；epoll_event 用于回传事件发生对应的数组；maxevents 表示能处理的最多的事件数；timeout 等待 IO 事件发生的超时值，-1 表示永久。

epoll_wait() 首先判断参数的有效性，最后会调用 ep_epoll() 函数。epoll 不仅会告诉应用程序有 IO 事件到来，还会告诉应用程序相关的信息，这些信息是应用程序填充的，因此根据这些信息应用程序就能直接定位到事件，而不必遍历整个 FD 集合。

### epoll() 的使用

如下是 epoll 使用的数据结构，执行过程。

{% highlight text %}
struct epoll_event {
    __uint32_t events;      // Epoll events
    epoll_data_t data;      // User data variable
};
typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;


struct epoll_event event;
struct epoll_event *events;
int efd = epoll_create(1);                        // 1. 创建文件描述符，大于0即可，实际内核中动态分配
if (efd == -1 || error == 0)
    error

event.data.fd = fd;                               // 2.1 设置需要监听的fd
event.events = EPOLLIN | EPOLLET;                 // 2.2 设置需要监听的事件
ret = epoll_ctl(efd, EPOLL_CTL_ADD, 0, &event)    // 2.3 可以添加、修改、删除
if (ret == -1)
    error

events = calloc(MAX_EVENTS, sizeof event);        // 3.1 申请内存，可返回的最大事件数
while (1) {
    n = epoll_wait(efd, events, MAX_EVENTS, -1);  // 3.2 等待事件发生
    if (n == 0)                                   // 3.3 超时
        timeout
    else if (n < 0)                               // 3.4 发生错误
        if(EINTT == errno)                        // 3.5 由于中断，忽略
            n = 0; continue;
        else
            error
    else                                          // 3.6 处理发生的事件
        for(i = 0; i &lt; n; i++)
            if (events[i].data.fd == fd)
                handleEvent
            else if (enents[i].events & EPOLLIN)   // 某些事件发生
                //
}
free(events);
close(fd);
{% endhighlight %}

结构体 epoll_event 用于注册所感兴趣的事件和回传所发生待处理的事件。epoll_data 联合体用来保存触发事件相关的描述符所对应的信息，可以保存很多类型的信息，如 fd 、指针等等，有了它，应用程序就可以直接定位目标了。

对于 epoll_data ，如一个 client 连接到服务器时，服务器通过调用 accept 函数可以得到 client 对应的 socket 文件描述符，并通过 epoll_data 的 fd 字段返回。

<!--
epoll_event 结构体的 events 字段是表示感兴趣的事件和被触发的事件，可能的取值为：<ul><li>
EPOLLIN： 表示对应的文件描述符可以读；</li><li>
EPOLLOUT：表示对应的文件描述符可以写；</li><li>
EPOLLPRI：表示对应的文件描述符有紧急的数据可读；</li><li>
EPOLLERR：表示对应的文件描述符发生错误；</li><li>
EPOLLHUP：表示对应的文件描述符被挂断；</li><li>
EPOLLET：表示对应的文件描述符有事件发生；
</li></ul>

LT(level triggered)，默认的工作方式，同时支持 block 和 no-block socket。对于该模式，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的 fd 进行 IO 操作。如果你不作任何操作，或者还没有处理完，内核还会继续通知你的，所以，这种模式编程出错误可能性要小一点。传统的 select/poll 都是这种模型的代表。<br><br>

ET(edge-triggered)，高速工作方式，只支持 no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过 epoll 告诉你，然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了，比如，你在发送、接收或者接收请求，或者发送接收的数据少于一定量时导致了一个 EWOULDBLOCK 错误。但是请注意，如果一直不对这个 fd 作 IO 操作（从而导致它再次变成未就绪），内核不会发送更多的通知(only once)，不过在TCP协议中，ET模式的加速效用仍需要更多的benchmark确认。<br><br>

ET和LT的区别在于LT事件不会丢弃，而是只要读buffer里面有数据可以让用户读，则不断的通知你。而ET则只在事件发生之时通知。可以简单理解为LT是水平触发，而ET则为边缘触发。<br><br>

ET模式仅当状态发生变化的时候才获得通知,这里所谓的状态的变化并不包括缓冲区中还有未处理的数据,也就是说,如果要采用ET模式,需要一直read/write直到出错为止,很多人反映为什么采用ET模式只接收了一部分数据就再也得不到通知了,大多因为这样;而LT模式是只要有数据没有处理就会一直通知下去的.<br><br>
-->



{% highlight text %}
{% endhighlight %}
