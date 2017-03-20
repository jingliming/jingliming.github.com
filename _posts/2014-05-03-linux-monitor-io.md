---
title: Linux 监控之 IO
layout: post
language: chinese
comments: true
category: [linux]
keywords: linux,monitor,监控,IO,fio,iostat,iotop,pidstat,iodump,ioprofile,blktrace
description: 简单介绍下 Linux 中与 IO 相关的内容。
---

简单介绍下 Linux 中与 IO 相关的内容。

<!-- more -->

## 简介

可以通过如下命令查看与 IO 相关的系统信息。

{% highlight text %}
# tune2fs -l /dev/sda7                       ← 读取superblock信息
# blockdev --getbsz /dev/sda7                ← 获取block大小
# tune2fs -l /dev/sda7 | grep "Block size"   ← 同上
# dumpe2fs /dev/sda7 | grep "Block size"     ← 同上
# stat /boot/ | grep "IO Block"              ← 同上
# fdisk -l                                   ← 硬盘的扇区大小(Sector Size)
{% endhighlight %}

> 在 [WiKi](https://en.wikipedia.org/wiki/Disk_formatting) 中的定义：A "block", a contiguous number of bytes, is the minimum unit of memory that is read from and written to a disk by a disk driver。

块是文件系统的抽象，而非磁盘的属性，一般是 Sector Size 的倍数；扇区大小则是磁盘的物理属性，它是磁盘设备寻址的最小单元。另外，内核中要求 Block_Size = Sector_Size * (2的n次方)，且 Block_Size <= 内存的 Page_Size (页大小)。



## iostat 系统级

可以直接通过 ```iostat -xdm 1``` 命令显示磁盘队列的长度等信息。该命令会读取 ```/proc/diskstats``` 文件中的数据，其中各个段的含义如下。

{% highlight text %}
filed1 成功读的数目
filed2 合并读的数目
filed3 成功读的sector数目
filed4 所有读花的时间（毫秒）
filed5 成功写的数目
filed6 合并写的数目
filed7 成功写的sector数目
filed8 所有写花的时间（毫秒）
filed9 现在正在进行的I/O数目
filed10 现在正在进行的I/O花的时间（毫秒），这个字段是filed9的一个累加值
filed11 是一个权重值，当有上面的I/O操作是，这个值就增加
{% endhighlight %}


<!--
其中参数如下：<ul><li>

rrqm/s wrqm/s<br>
读写请求每秒合并后发送给磁盘的请求。</li><br><li>

r/s w/s<br>
应用发送给系统的请求数目。</li><br><li>

rrqm/s    : The number of read requests merged per second that were queued to the hard disk
wrqm/s    : The number of write requests merged per second that were queued to the hard disk
rsec/s    : The number of sectors read from the hard disk per second
wsec/s    : The number of sectors written to the hard disk per second
svctm     : The average service time (in milliseconds) for I/O requests that were issued to the device
%util     : Percentage of CPU time during which I/O requests were issued to the device (bandwidth utilization for the device). Device saturation occurs when this value is close to 100%.


argrq-sz<br>
提交给驱动层 IO 请求的平均大小 (sectors)，一般不小于 4K，不大于 max(readahead_kb, max_sectors_kb)。可用于判断当前的 IO 模式，越大代表顺序，越小代表随机。
<pre style="font-size:0.8em; face:arial;">
argrq-sz = (rsec + wsec) / (r + w)
</pre></li><br><li>

argqu-sz<br>
提供给驱动层的队列平均长度。</li><br><li>

svctm (ms)<br>
一次磁盘 IO 请求的服务时间，对于单块盘，完全随机读时，基本在 7ms 左右，既寻道+旋转延迟时间。不过据说往后会删掉该监控选项。</li><br><li>

await<br>
平均的等待事件，包括了在队列中的等待时间以及实际磁盘的处理时间。</li><br><li>

%util<br>
一秒内 IO 操作所占的比例，计算公式是 (r/s+w/s)*(svctm/1000)，对于一块磁盘，因为没有并发IO的概念，所以这个公式是正确的，但是对于RAID磁盘组或者SSD来说，这个计算公式就有问题了，就算这个值超过100%，也不代表存储有瓶颈，容易产生误导。

Service time (storage wait) is the time it takes to actually send the I/O request to the storage and get an answer back – this is the time the storage system needs to handle the I/O. It varies between 3.8 and 7 ms on average.

Average Wait (host wait) is the time the I/O’s wait in the host I/O queue.

Average Queue Size is the average amount of I/O’s waiting in the queue.

A very simple example: Assume a storage system handles each I/O in exactly 10 milliseconds and there are always 10 I/O’s in the host queue. Then the Average Wait will be 10 x 10 = 100 milliseconds.

In this specific test you can verify that storage wait * queue size ≈ host wait

注: 各统计量之间关系
=======================================
%util = ( r/s  +  w/s) * svctm / 1000                        # 队列长度 =  到达率     *  平均服务时间
avgrq-sz = ( rMB/s + wMB/s) * 2048 / (r/s  + w/s)    # 2048 为 1M / 512
=======================================
iostat 统计的是通用块层经过合并(rrqm/s, wrqm/s)后,直接向设备提交的IO数据,可以反映系统整体的IO状况,但是有以下2个缺点:
距离业务层比较遥远,跟代码中的write,read不对应(由于系统预读 + pagecache + IO调度算法等因素, 也很难对应)
是系统级,没办法精确到进程,比如只能告诉你现在磁盘很忙,但是没办法告诉你是谁在忙,在忙什么？
-->

## iotop pidstat iodump 进程级

一个 Python 脚本，可以查看官网 [guichaz.free.fr/iotop](http://guichaz.free.fr/iotop/)，另一个通过 C 实现的监控可参考 [IOPP](https://github.com/markwkm/iopp)。

pidstat 用于统计进程的状态，包括 IO 状况，可以查看当前系统哪些进程在占用 IO 。

{% highlight text %}
----- 只显示IO
# pidstat -d 1
{% endhighlight %}

上述两者均是统计的 ```/proc/pid/io``` 中的信息；另可参考 io/iotop.stp，这是 iotop 的复制版。

iodump 是一个统计每一个进程(线程)所消耗的磁盘 IO 工具，是一个 perl 脚本，其原理是打开有关 IO 的内核记录消息开关，而后读取消息然后分析输出。

{% highlight text %}
# echo 1 >/proc/sys/vm/block_dump                        # 打开有关IO内核消息的开关
# while true; do sleep 1; dmesg -c ; done | perl iodump  # 然后分析
{% endhighlight %}

上述输出的单位为块 (block)，每块的大小取决于创建文件系统时指定的块大小。


## ioprofile 业务级

ioprofile 命令本质上等价于 lsof + strace，可以查看当前进程。

<!--
具体下载可见 http://code.google.com/p/maatkit/

ioprofile 可以回答你以下三个问题:
1  当前进程某时间内,在业务层面读写了哪些文件(read, write)？
2  读写次数是多少?(read, write的调用次数)
3  读写数据量多少?(read, write的byte数)
假设某个行为会触发程序一次IO动作,例如: "一个页面点击,导致后台读取A,B,C文件"
-->


## blktrace

blktrace 是块层 IO 路径监控和分析工具，作者 Jens Axboe 是内核 IO 模块的维护者，目前就职于 FusionIO，同时他还是著名 IO 评测工具 fio 的作者，使用它可以深入了解 IO 通路。

{% highlight text %}
# yum install blktrace                    # 在CentOS中安装
$ make                                    # 解压源码后直接安装
$ man -l doc/blktrace.8                   # 查看帮助
{% endhighlight %}

其源码可以从 [brick.kernel.dk](http://brick.kernel.dk/snaps/) 下载，详细使用参考 [blktrace User Guide](http://www.cse.unsw.edu.au/~aaronc/iosched/doc/blktrace.html) 。

### 原理

该工具包括了内核空间和用户空间两部分实现，内核空间里主要是给块层 IO 路径上的关键点添加 tracepoint，然后借助于 relayfs 系统特性将收集到的数据写到 buffer 去，再从用户空间去收集。

目前，内核空间部分的代码已经集成到主线代码里面去了，可以看看内核代码 block/blktrace.c 文件是不是存在，编译的时候把对应的这个 trace 选项选择上就可以了。

{% highlight text %}
$ grep 'CONFIG_BLK_DEV_IO_TRACE' /boot/config-`uname -r`
{% endhighlight %}

大部分实现代码都在 blktrace.c，利用 tracepoint 的特性，注册了一些 trace 关键点，可以查看 Documentation/tracepoint.txt 文件；交互机制利用了 relayfs 特性，看看 Documentation/filesystems/relay.txt 。

此时捞取的信息还比较原始，可以通过用户空间的 blkparse、btt、seekwatcher 这样的工具来分析收集到的数据。

注意，使用之前要确保 debugfs 已经挂载，默认会挂载在 /sys/kernel/debug 。

### 使用

典型的使用如下，其中 /dev/sdaa、/dev/sdc 作为 LVM volume adb3/vol。

{% highlight text %}
# blktrace -d /dev/sda -o - | blkparse -i - -o blkparse.out       # 简单用法，Ctrl-C退出
# btrace /dev/sda                                                 # 同上

# blktrace /dev/sdaa /dev/sdc &                                   # 离线处理。1. 后台运行采集
% mkfs -t ext3 /dev/adb3/vol                                      # 2. 做些IO操作
% kill -15 9713                                                   # 3. 停止采集
% blkparse sdaa sdc sdo > events                                  # 4. 解析后查看
{% endhighlight %}

在 blktrace 中，-d 表示监控哪个设备，-o - 表示将监控输出到标准输出；在 blkparse 中，-i - 表示从标准输入获取信息，-o 表示将解析的内容记录在 blkparse.out 。

如下是输出的详细信息。

![monitor io blktrace]({{ site.url }}/images/linux/monitor-io-blktrace.png "monitor io blktrace"){: .pull-center width="80%"}

其中 event 对应了事件表；后面一列代表了操作类型，包括了 R(read)、W(write)、B(barrier operation)、S(synchronous operation)，其中 event 有如下类型：

|事件|说明|源码(block目录下) SetPosition|
|:---:|:---|:---|
|A |IO was remapped to a different device | blk-core.c/trace_block_remap |
|B |IO bounced                            | bounce.c/trace_block_bio_bounce |
|C |IO completion                         | blk-core.c/trace_block_rq_complete |
|D |IO issued to driver                   | elevator.c/trace_block_rq_issue |
|F |IO front merged with request on queue | blk-core.c/trace_block_bio_frontmerge |
|G |Get request                           | blk-core.c/trace_block_getrq |
|I |IO inserted onto request queue        | elevator.c/trace_block_rq_insert |
|M |IO back merged with request on queue  | blk-core.c/trace_block_bio_backmerge |
|P |Plug request                          | blk-core.c/trace_block_plug |
|Q |IO handled by request queue code      | blk-core.c/trace_block_bio_queue |
|S |Sleep request                         | blk-core.c/trace_block_sleeprq |
|T |Unplug due to timeout                 | blk-core.c/trace_block_unplug_timer |
|U |Unplug request                        | blk-core.c/trace_block_unplug_io |
|X |Split                                 | bio.c/trace_block_split |


### 详解

仍以如下简单命令为例。

{% highlight text %}
$ blktrace -d /dev/sda -o sda                 # 输出 sda.blktrace.N 文件，N 为物理 CPU 个数。
$ ls /sys/kernel/debug/block/sda              # 查看debugfs中的文件
dropped  msg  trace0  trace1  trace2  trace3
$ blkparse -i sda.blktrace.0                  # 解析成可读内容
$ blkrawverify sda                            # 校验，其中sda为blktrace的-o选项
{% endhighlight %}

其中 blktrace 通过 ioctl() 执行 BLKTRACESETUP、BLKTRACESTART、BLKTRACESTOP、BLKTRACETEARDOWN 操作，此时会在 debugfs 目录的 block/DEV 目录下写入数据。

<!--
<pre style="font-size:0.8em; face:arial;">
main()
   run_tracers()
      setup_buts()
      use_tracer_devpaths()
      start_tracers()
         start_tracer()          通过pthread_create()创建
            thread_main()
               open_ios()
               handle_pfds()     对于写入本地文件实际采用handle_pfds_entries()
</pre>
  struct io_info 输入输出结构
-f "%D %2c %8s %5T.%9t %5p %2a %3d "
-->


## FIO

FIO 是个非常强大的 IO 性能测试工具，其作者 Jens Axboe 是 Linux 内核 IO 部分的 maintainer，可以毫不夸张的说，如果你把所有的 FIO 参数都搞明白了，基本上就把 Linux IO 协议栈的问题搞的差不多明白了。

一个 IO 压测工具，源码可以参考 [github-axboe](https://github.com/axboe/fio/)，或者直接从 [freecode.com](http://freecode.com/projects/fio/) 上下载。

另外，该工具同时提供了一个图形界面 gfio 。

### 源码编译

可以直接从 github 上下载源码，然后通过如下方式进行编译。

{% highlight text %}
----- 编译，注意依赖libaio
$ make

----- 查看帮助
$ man -l fio.1

----- 通过命令行指定参数，进行简单测试
$ fio --name=global --rw=randread --size=128m --name=job1 --name=job2

----- 也可以通过配置文件进行测试
$ cat foobar.fio
[global]
rw=randread
size=128m
[job1]
[job2]
$ fio foobar.fio
{% endhighlight %}

可以通过命令行启动，不过此时参数较多，可以使用配置文件。

<!--
{% highlight text %}
# 顺序读
fio --filename=/dev/sda --direct=1 --rw=read --bs=4k --size=10G --numjobs=30 --runtime=20 --group_reporting --name=test-read

顺序写

fio --filename=/dev/sda --direct=1 --rw=write --bs=4k --size=10G --numjobs=30 --runtime=20 --group_reporting --name=test-write

随机读测试

fio --filename=/dev/sda --direct=1 --rw=randread --bs=4k --size=10G --numjobs=64 --runtime=20 --group_reporting --name=test-rand-read

随机写测试

fio --filename=/dev/sda --direct=1 --rw=randwrite --bs=4k --size=10G --numjobs=64 --runtime=20 --group_reporting --name=test-rand-write


ioengine=psync            # 设置io引擎，通过-i查看当前支持的
rw=randwrite              # 读写模型，包括了顺序读写、随机读写等
bs=16k                    # 单次io的块文件大小为16k
direct=1                  # 测试过程绕过机器自带的buffer，使测试结果更真实
size=5g                   # 指定测试文件的大小




filename=/dev/sdb1       测试文件名称，通常选择需要测试的盘的data目录。
bsrange=512-2048         同上，提定数据块的大小范围
numjobs=30               本次的测试线程为30.
runtime=1000             测试时间为1000秒，如果不写则一直将5g文件分4k每次写完为止。
rwmixwrite=30            在混合读写的模式下，写占30%
group_reporting          关于显示结果的，汇总每个进程的信息。

此外
lockmem=1g               只使用1g内存进行测试。
zero_buffers             用0初始化系统buffer。
nrfiles=8                每个进程生成文件的数量
{% endhighlight %}

应用使用IO通常有二种方式：同步和异步。 同步的IO一次只能发出一个IO请求，等待内核完成才返回，这样对于单个线程iodepth总是小于1，但是可以透过多个线程并发执行来解决，通常我们会用16-32根线程同时工作把iodepth塞满。 异步的话就是用类似libaio这样的linux native aio一次提交一批，然后等待一批的完成，减少交互的次数，会更有效率。
-->


### 源码解析

其版本通过 FIO_VERSION 宏定义，并通过 fio_version_string 变量定义。

{% highlight text %}
main()
  |-parse_options()
  |  |-parse_cmd_line()                    解析命令行，如-i显示所有的ioengines
  |  |  |-add_job()                        file1: xxxxxx 打印job信息
  |  |-log_info()                          fio-2.10.0
  |-fio_backend()
  |  |-create_disk_util_thread()           用于实时显示状态
  |  |  |-setup_disk_util()
  |  |  |-disk_thread_main()               通过pthread创建线程
  |  |     |-print_thread_status()
  |  |
  |  |-run_threads()                       Starting N processes
  |  |  |-setup_files()                    Laying out IO file(s)
  |  |  |-pthread_create()                 如果配置使用线程，调用thread_main
  |  |  |-fork()                           或者调用创建进程，同样为thread_main
  |  |
  |  |-show_run_stats()
  |     |-show_thread_status_normal()      用于显示最终的状态
  |        |-show_latencies()              显示lat信息
  |        |-... ...                       CPU、IO depth
{% endhighlight %}

ioengines 通过 fio_libaio_register() 类似的函数初始化。

## 参考

关于 FIO 可以查看源码中的 [HOWTO](/reference/linux/monitor/HOWTO.txt)，其它的压测工具可以参考 [Benchmarking](https://wiki.mikejung.biz/Benchmarking) ，或者参考 [本地文档](/reference/linux/monitor/Benchmarking.mht)，该网站还包括了很多有用文章。

[Block IO Layer Tracing: blktrace](/reference/linux/monitor/gelato_ICE06apr_blktrace_brunelle_hp.pdf) 介绍 blktrace 命令的使用；关于内核的 trace 功能参考 [Kernel Trace Systems](http://elinux.org/Kernel_Trace_Systems) 。

[Linux Block IO: Introducing Multi-queue SSD Access on Multi-core Systems](/reference/linux/monitor/linux-block-io-multi-queue-ssd.pdf) 。

<!--
http://www.cnblogs.com/quixotic/p/3258730.html
-->
{% highlight text %}
{% endhighlight %}
