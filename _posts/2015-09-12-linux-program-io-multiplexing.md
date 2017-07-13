---
title: Linux IO 多路复用
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

epoll_event 结构体的 events 字段是表示感兴趣的事件和被触发的事件，可能的取值为：

{% highlight text %}
EPOLLIN ：对应的文件描述符可以读；
EPOLLOUT：对应的文件描述符可以写；
EPOLLPRI：对应的文件描述符有紧急的数据可读；
EPOLLERR：对应的文件描述符发生错误；
EPOLLHUP：对应的文件描述符被挂断；
EPOLLET ：对应的文件描述符有事件发生。
{% endhighlight %}

<!--
LT(level triggered)，默认的工作方式，同时支持 block 和 no-block socket。对于该模式，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的 fd 进行 IO 操作。如果你不作任何操作，或者还没有处理完，内核还会继续通知你的，所以，这种模式编程出错误可能性要小一点。传统的 select/poll 都是这种模型的代表。

ET(edge-triggered)，高速工作方式，只支持 no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过 epoll 告诉你，然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了，比如，你在发送、接收或者接收请求，或者发送接收的数据少于一定量时导致了一个 EWOULDBLOCK 错误。但是请注意，如果一直不对这个 fd 作 IO 操作（从而导致它再次变成未就绪），内核不会发送更多的通知(only once)，不过在TCP协议中，ET模式的加速效用仍需要更多的benchmark确认。

ET和LT的区别在于LT事件不会丢弃，而是只要读buffer里面有数据可以让用户读，则不断的通知你。而ET则只在事件发生之时通知。可以简单理解为LT是水平触发，而ET则为边缘触发。

ET模式仅当状态发生变化的时候才获得通知,这里所谓的状态的变化并不包括缓冲区中还有未处理的数据,也就是说,如果要采用ET模式,需要一直read/write直到出错为止,很多人反映为什么采用ET模式只接收了一部分数据就再也得不到通知了,大多因为这样;而LT模式是只要有数据没有处理就会一直通知下去的.
-->

### 内核实现

epoll 的几个接口实际都对应于 kernel 的 API ，主要位于 `fs/eventpoll.c` 文件中。在分析 epoll 时发现有 `fs_initcall()` 这样的调用，以此为例分析一下 Linux 的初始化。


<!--
<pre style="font-size:0.8em; face:arial;">
fs_initcall(eventpoll_init);                                      // fs/eventpoll.c
#define fs_initcall(fn) __define_initcall(fn, 5)                  // include/linux/init.h
#define __define_initcall(fn, id) \                               // 同上
      static initcall_t __initcall_##fn##id __used \
      __attribute__((__section__(".initcall" #id ".init"))) = fn

// 最后展开为
static initcall_t __initcall_eventpoll_init5 __used __attribute__((__section__(".initcall5.init"))) = eventpoll_init;
</pre>
也就是在 .initcall5.init 段中定义了一个变量 __initcall_eventpoll_init5 并将改变量赋值为 eventpoll_init 。内核中对初始化的调用过程如下。
<pre style="font-size:0.8em; face:arial;">
arch/x86/kernel/head_64.S
   x86_64_start_kernel();                    arch/x86/kernel/head[64|32].c
      start_kernel();                        init/main.c
         rest_init();                        init/main.c
            kernel_init();                   init/main.c，通过内核线程实现
               kernel_init_freeable();       init/main.c
                  do_basic_setup();          init/main.c
                      do_initcalls();        init/main.c
</pre>
对于 epoll 来说，实际是在初始化过程中对变量进行初始化。<br><br><br>





当某一进程调用 epoll_create() 方法时，内核会创建一个 eventpoll 结构体，这个结构体中有两个成员与 epoll 的使用方式密切相关。
<pre style="font-size:0.8em; face:arial;">
struct eventpoll{
    struct rb_root  rbr;        // 红黑树的根节点，这颗树中存储着所有添加到epoll中的需要监控的事件
    struct list_head rdlist;    // 双链表中则存放着将要通过epoll_wait返回给用户的满足条件的事件
};
</pre>
每一个 epoll 对象都有一个独立的 eventpoll 结构体，用于存放通过 epoll_ctl() 方法向 epoll 对象中添加进来的事件，这些事件都会挂载在红黑树中。<br><br>

而所有添加到 epoll 中的事件都会与设备+网卡驱动程序建立回调关系，也就是说，当相应的事件发生时会调用这个回调方法。这个回调方法在内核中就是 ep_poll_callback() 它会将发生的事件添加到 rdlist 双链表中。<br><br>

在 epoll 中，对于每一个事件，都会建立一个 epitem 结构体。
<pre style="font-size:0.8em; face:arial;">
struct epitem{
    struct rb_node rbn;         // 红黑树节点
    struct list_head rdllink;   // 双向链表节点
    struct epoll_filefd ffd;    // 事件句柄信息
    struct eventpoll *ep;       // 指向其所属的eventpoll对象
    struct epoll_event event;   // 期待发生的事件类型
}
</pre>
当调用 epoll_wait() 检查是否有事件发生时，只需要检查 eventpoll 对象中的 rdlist 双链表中是否有 epitem 元素即可。如果 rdlist 不为空，则把发生的事件复制到用户态，同时将事件数量返回给用户。<br><br>
<center><img src="pictures/polletc/epoll_data_structure.jpg"></center><br><br>

其中 epoll_create()、epoll_ctl() 的函数调用如下所示。
<pre style="font-size:0.8em; face:arial;">
epoll_create();
    ep_alloc();                 // 分配struct eventpoll变量，由epoll_create返回的fd在内核中对应该变量
    get_unused_fd_flags();      // 返回一个有效的fd
    anon_inode_getfile();       // 返回一个file，同时注册eventpoll_fops
    fd_install();               // 将上述的两者关联起来并返回fd

epoll_ctl()
    switch(op)
        EPOLL_CTL_ADD:                         // 处理不同的事件
            epds.events |= POLLERR | POLLHUP;  // 默认会包含两个事件
            ep_insert()
        ... ...
</pre>


ep_ptable_queue_proc 当有事件发生时会调用该函数。



select需要驱动程序实现fops内的poll函数，通过每个设备文件对应的poll函数提供的信息判断当前是否有资源可用(如可读或写)，如果有的话则返回可用资源的文件描述符个数，没有的话则睡眠，等待有资源变为可用时再被唤醒继续执行。

select就是巧妙的利用等待队列机制让用户进程适当在没有资源可读/写时睡眠，有资源可读/写时唤醒。

select会循环遍历它所监测的fd_set内的所有文件描述符对应的驱动程序的poll函数。驱动程序提供的poll函数首先会将调用select的用户进程插入到该设备驱动对应资源的等待队列(如读/写等待队列)，然后返回一个bitmask告诉select当前资源哪些可用。当select循环遍历完所有fd_set内指定的文件描述符对应的poll函数后，如果没有一个资源可用(即没有一个文件可供操作)，则select让该进程睡眠，一直等到有资源可用为止，进程被唤醒(或者timeout)继续往下执行。











每次操作时poll会将所有的fd复制到内核，而epoll在epoll_ctl时添加，epoll_wait时不需要重复拷贝，



epoll的实现主要依赖于一个迷你文件系统：eventpollfs。此文件系统通过eventpoll_init初始化。在初始化的过程中，eventpollfs create两个slub分别是：epitem和eppoll_entry。

epoll使用过程中有几个基本的函数分别是epoll_create，epoll_ctl，epoll_wait。涉及到四个重要的数据结构： struct eventpoll ， struct epitem， struct epoll_event ，struct eppoll_entry。(作者：黄江伟，will.huang@aliyun-inc.com)

1、epoll_create和epoll_ctl

其中eventpoll是通过epoll_create生成，epoll_create传入一个size参数，size参数只要>0即可，没有任何意义。epoll_create调用函数sys_epoll_create1实现eventpoll的初始化。sys_epoll_create1通过ep_alloc生成一个eventpoll对象，并初始化eventpoll的三个等待队列，wait，poll_wait以及rdlist （ready的fd list）。同时还会初始化被监视fs的rbtree 根节点。

epollcreate在调用ep_alloc通过anon_inode_getfd创建一个名字为“[eventpoll]”的eventpollfs文件描述符号并将file->private_data指定为指向前面生成的eventpoll。这样就将eventpoll和文件id关联。最后返回文件描述符id。

通过epoll_create生成一个eventpoll后，可以通过epoll_ctl提供的相关操作对eventpoll进行ADD，MOD，DEL操作。epoll_ctl有四个参数，分别是：int epfd（需要操作的eventpoll）, int op（操作类型）, int fd（需要被监视的文件）, struct epoll_event *event（被监视文件的相关event）。epoll_ctl首先通过epfd的private_data域获取需要操作的eventpoll，然后通过ep_find确认需要操作的fd是否已经在被监视的红黑树中（eventpoll->rbr）。然后根据op的类型分别作ADD（ep_insert），MOD（ep_modify），DEL（ep_remove）操作。

首先分析ep_insert，ep_insert有四个参数分别为： struct eventpoll *ep（需要操作的eventpoll）, struct epoll_event *event（epoll_create传入的event参数，当然得从user空间拷贝过来）, struct file *tfile（被监视的文件描述符）, int fd（被监视的文件id）。ep_insert首先从slub中分配一个epitem的对象epi。并初始化epitem的三个list头指针，rdllink（指向eventpoll的rdlist），fllist指向（struct file的f_ep_links），pwqlist（指向包含此epitem的所有poll wait queue）。并将epitem的ep指针，指向传入的eventpoll，并通过传入参数event对ep内部变量event赋值。然后通过ep_set_ffd将目标文件和epitem关联。这样epitem本身就完成了和eventpoll以及被监视文件的关联。下面还需要做两个动作：将epitem插入目标文件的polllist并注册回调函数；将epitem插入eventpoll的rbtree。

为了完成第一个动作，还需要一个数据结构ep_pqueue帮忙，ep_pqueue主要包含两个变量一个是epitem还有一个是callback函数（ep_ptable_queue_proc）相关的一个数据结构poll_table，ep_pqueue主要完成epitem和callback函数的关联。然后通过目标文件的poll函数调用callback函数ep_ptable_queue_proc。Poll函数一般由设备驱动提供，以网络设备为例，他的poll函数为sock_poll然后根据sock类型调用不同的poll函数如：packet_poll。packet_poll在通过datagram_poll调用sock_poll_wait，最后在poll_wait实际调用callback函数（ep_ptable_queue_proc）。

ep_ptable_queue_proc函数完成epitem加入到特定文件的wait队列任务。ep_ptable_queue_proc有三个参数：struct file *file（目标文件）, wait_queue_head_t *whead（目标文件的waitlist）, poll_table *pt（前面生成的poll_table）。在函数中，引入了另外一个非常重要的数据结构eppoll_entry。eppoll_entry主要完成epitem和epitem事件发生时的callback（ep_poll_callback）函数之间的关联，并将上述两个数据结构包装成一个链表节点，挂载到目标文件file的waithead中。这两还得完成两个动作，首先将eppoll_entry的whead指向目标文件的waitlist（传入的参数2），然后初始化base变量指向epitem，最后通过add_wait_queue将epoll_entry挂载到目标文件的waitlist。完成这个动作后，epoll_entry已经被挂载到waitlist，然后还有一个动作必须完成，就是将eppoll_entry挂载到epitem的pwqlist上面。现在还剩下一个动作，就是将epitem的fllink链接到目标文件的f_ep_links上，这部分工作将在poll函数返回后在ep_insert中完成。当然ep_insert除了完成这个动作外，还会完成前面提到的第二步，epitem插入eventpoll的rbtree。完成以上动作后，将还会判断当前插入的event是否刚好发生，如果是，那么做一个ready动作，将epitem加入到rdlist中，并对epoll上的wait队列调用wakeup。

到此为止基本完成了epoll_create以及epoll_ctl最重要的ADD函数的工作介绍。下面进入epoll_wait函数介绍。(作者：黄江伟，will.huang@aliyun-inc.com)

2、epoll_wait

epoll_wait有四个参数int epfd（被wait的epoll所关联的epollfs的fd）, struct epoll_event __user * events（返回监视到的事件）, int maxevents（每次return的events最大值）, int timeout（最大wait时间）。epoll_wait首先会检测传入参数的合法性，包括maxevents有没有超过范围（0<=maxevents<EP_MAX_EVENTS（(INT_MAX / sizeof(struct epoll_event))））；events指向的空间是否可写；epfd是否合法等。参数合法性检测都通过后，将通过epfd获取锁依赖的struct file，然后通过file->private_data获取eventpoll。获取epoll后调用ep_poll函数完成真正的epoll_wait工作。ep_poll函数也是四个参数和epoll_wait唯一的差别就是第一参数是前面获取的eventpoll指针。ep_poll首先根据timeout的值判断是否是无限等待，如果不是将timeout（ms）转换为jiffs。然后判断eventpoll的rdlist是否为空，如果为空，那么将current进程通过一个waitquene entry加入eventpoll的waitlist（wq）。并将task的状态改为TASK_INTERRUPTIBLE；并通过schedule_timeout让出处理器。如果rdlist非空，那么通过ep_send_events将event转发到userspace。

ep_send_events通过ep_scan_ready_list对ready_list进行扫描，由于现在在对ready_list进行操作，这个时候必须保证rdlist数据的一致性，如果此时又有新的event ready，那么我们必须提供临时的存储空间，eventpoll提供了一个ovflist用来存储这种event。ep_send_events获取了rdlist后通过ep_send_events_proc完成真正的转发工作。完成转发后，ep_send_events还需要去判断ovflist，如果ovflist中有events，那么还需要将这些events转移到rdlist中。

ep_send_events_proc扫描rdlist从头上面拿出epitem，然后调用epollfs的poll函数（ep_eventpoll_poll），判断拿出来的那个events是否真的已经ready（这部分比较难理解，没怎么看懂）。如果ready，那么将数据封装到uevent里面，同事这里还需要判断epitem的类型是否是Level Triggered如果是，那么还需要把event再次插入队列尾部。(作者：黄江伟，will.huang@aliyun-inc.com)

3、ep_poll_callback

以上描述中还缺少关键的一环，就是如何在被监视文件发生event的时候，如何将epitem加入rdlist并唤醒调用epoll_wait进程。这个工作由ep_poll_callback函数完成。前面提到eppoll_entry完成一个epitem和ep_poll_callback的关联，同时eppoll_entry会被插入目标文件file的（private_data）waithead中。以scoket为例，当socket数据ready，终端会调用相应的接口函数比如rawv6_rcv_skb，此函数会调用sock_def_readable然后，通过sk_has_sleeper判断sk_sleep上是否有等待的进程，如果有那么通过wake_up_interruptible_sync_poll函数调用ep_poll_callback。

ep_poll_callback函数首先会判断是否rdlist正在被使用（通过ovflist是否等于EP_UNACTIVE_PTR），如果是那么将epitem插入ovflist。如果不是那么将epitem插入rdlist。然后调用wake_up函数唤醒epitem上wq的进程。这样就可以返回到epoll_wait的调用者，将他唤醒。(作者：黄江伟，will.huang@aliyun-inc.com)

http://www.embeddedlinux.org.cn/html/yingjianqudong/201405/11-2860.html poll&&epoll实现分析（一）—poll实现
http://blog.csdn.net/xiajun07061225/article/details/9250579
http://www.cnblogs.com/apprentice89/p/3234677.html
http://www.cnblogs.com/debian/archive/2012/02/16/2354454.html
http://blog.csdn.net/justlinux2010/article/details/8506890
http://blog.chinaunix.net/uid-20687780-id-2105154.html
http://www.cnblogs.com/apprentice89/archive/2013/05/09/3068274.html
http://blog.chinaunix.net/uid-20687780-id-2105157.html
http://blog.chinaunix.net/uid-29339876-id-4070572.html
http://bbs.chinaunix.net/thread-2021810-1-1.html
http://blog.csdn.net/sunpyliu/article/details/6761614
http://blog.chinaunix.net/uid-26339466-id-3292595.html
http://blog.csdn.net/russell_tao/article/details/7160071

ptmalloc,tcmalloc和jemalloc内存分配策略研究 -->


{% highlight text %}
{% endhighlight %}
