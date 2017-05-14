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

除了可以通过该命令查看磁盘信息之外，还可以用来查看 CPU 信息，分别通过 ```-d``` 和 ```-c``` 参数控制；可直接通过 ```iostat -xdm 1``` 命令显示磁盘队列的长度等信息。

{% highlight text %}
Device: rrqm/s wrqm/s   r/s   w/s  rMB/s  wMB/s avgrq-sz avgqu-sz await r_await w_await svctm %util
sda       0.02   1.00  0.99  1.84   0.03   0.04    46.98     0.01  2.44    0.47    3.49  0.25  0.07
{% endhighlight %}

其中参数如下：

{% highlight text %}
rrqm/s wrqm/s
  读写请求每秒合并后发送给磁盘的请求。

r/s w/s
  应用发送给系统的请求数目。

argrq-sz
  提交给驱动层IO请求的平均大小(sectors)，一般不小于4K，不大于max(readahead_kb, max_sectors_kb)；
  可用于判断当前的 IO 模式，越大代表顺序，越小代表随机；计算公式如下：
  argrq-sz = (rsec + wsec) / (r + w)

argqu-sz Average Queue Size
  在驱动层的队列排队的平均长度。

await Average Wait
  平均的等待时间，主要包括了在队列中的等待时间，暂不确认是否包含了磁盘的处理时间。

svctm(ms) Service Time
  一次磁盘IO请求的服务时间，也就是请求发送后到接收到响应的时间差，对于单块SATA盘，完全随机读时，
  基本在7ms左右，既寻道+旋转延迟时间；不过据说以后会删掉该监控选项。

%util
  一秒内IO操作所占的比例，计算公式是(r/s+w/s)*(svctm/1000)；对于一块磁盘，因为没有并发IO的概念，
  所以这个公式是正确的，但是对于RAID磁盘组或者SSD来说，这个计算公式就有问题了，就算这个值超过100%，
  也不代表存储有瓶颈，容易产生误导。
{% endhighlight %}

iostat 统计的是通用块层经过合并 (rrqm/s, wrqm/s) 后，直接向设备提交的 IO 数据，可以反映系统整体的 IO 状况，但是距离应用层比较远，由于系统预读、Page Cache、IO调度算法等因素，很难跟代码中的 write()、read() 对应。

简言之，这是系统级，没办法精确到进程，比如只能告诉你现在磁盘很忙，但是没办法告诉你是那个进程在忙，在忙什么？

<!--
rsec/s    : The number of sectors read from the hard disk per second
wsec/s    : The number of sectors written to the hard disk per second
-->

### 其它

该命令会读取 ```/proc/diskstats``` 文件中的数据，其中各个段的含义如下。

{% highlight text %}
filed1  成功完成读的总次数；
filed2  合并写完成次数，通过合并提高效率，例如两次4K合并为8K，这样只有一次IO操作；
filed3  成功读过的扇区总次数；
filed4  所有读操作所花费的毫秒数；
filed5  成功完成写的总次数；
filed6  合并写的次数；
filed7  成功写过的扇区总次数；
filed8  所有写操作所花费的毫秒数；
filed9  现在正在进行的IO数目；
filed10 输入/输出操作花费的毫秒数；
filed11 是一个权重值，当有上面的IO操作时，这个值就增加。
{% endhighlight %}







http://veithen.github.io/2013/11/18/iowait-linux.html




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

一个 IO 压测工具，源码以及二进制文件可以参考 [github-axboe](https://github.com/axboe/fio/)，或者直接从 [freecode.com](http://freecode.com/projects/fio/) 上下载。另外，该工具同时提供了一个图形界面 gfio 。

在 CentOS 中可以通过如下方式安装。

{% highlight text %}
# yum --enablerepo=epel install fio
{% endhighlight %}

<!-- fio2gnuplot.py fio_generate_plots -->

<!--
fio --name=global --rw=randread --size=128m --name=job1
job1: (g=0): rw=randread, bs=4K-4K/4K-4K/4K-4K, ioengine=sync, iodepth=1
fio-2.1.2
Starting 1 process
job1: Laying out IO file(s) (1 file(s) / 128MB)
Jobs: 1 (f=1): [r] [100.0% done] [6228KB/0KB/0KB /s] [1557/0/0 iops] [eta 00m:00s]
job1: (groupid=0, jobs=1): err= 0: pid=122727: Fri Mar 31 10:52:45 2017
  read : io=131072KB, bw=4485.9KB/s, iops=1121, runt= 29224msec
    clat (usec): min=250, max=72421, avg=888.51, stdev=932.95
     lat (usec): min=250, max=72422, avg=888.73, stdev=932.96
    clat percentiles (usec):
     |  1.00th=[  406],  5.00th=[  434], 10.00th=[  450], 20.00th=[  474],
     | 30.00th=[  494], 40.00th=[  516], 50.00th=[  548], 60.00th=[  604],
     | 70.00th=[  756], 80.00th=[ 1144], 90.00th=[ 1960], 95.00th=[ 2544],
     | 99.00th=[ 3632], 99.50th=[ 4448], 99.90th=[ 6496], 99.95th=[ 8096],
     | 99.99th=[34560]
    bw (KB  /s): min= 2691, max= 6088, per=99.97%, avg=4483.69, stdev=724.50
    lat (usec) : 500=32.24%, 750=37.59%, 1000=7.10%
    lat (msec) : 2=13.41%, 4=8.95%, 10=0.68%, 20=0.01%, 50=0.01%
    lat (msec) : 100=0.01%
  cpu          : usr=0.42%, sys=2.05%, ctx=32789, majf=0, minf=22
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=32768/w=0/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
   READ: io=131072KB, aggrb=4485KB/s, minb=4485KB/s, maxb=4485KB/s, mint=29224msec, maxt=29224msec

Disk stats (read/write):
  xvde: ios=32737/21354, merge=0/113646, ticks=27852/53296, in_queue=81132, util=95.60%


首先，每个任务的第一行非常明确，如下也就是读取(131072KB=128MB)数据，速率是4.48MB/s，其中IOPS是1121(4K block size)，执行了2.94秒。
  read : io=131072KB, bw=4485.9KB/s, iops=1121, runt= 29224msec

接下来是latency的计量值，在老版本中，先是slat(submission latency)也就是将本次的IO提交到内核所消耗的时间，不过在新版本中该统计值已经取消。
    slat (usec): min=3, max=335, avg= 9.73, stdev= 5.76

下面是clat(completion latency)，也就是提交到内核后到IO完成的时间，不包括上述的slat时间，老版本中这个是从应用层比较接近的延迟时间，而新版本中可以参考lat。
    clat (usec): min=250, max=72421, avg=888.51, stdev=932.95
 
lat(latency)这个统计值是从fio创建结构体开始，到IO完成的时间值，所以，也就是最接近从应用层看到的延迟时间。
     lat (usec): min=250, max=72422, avg=888.73, stdev=932.96

接下来，是clat的分布情况，TODODO: 嘛意思啊
    clat percentiles (usec):
     |  1.00th=[  406],  5.00th=[  434], 10.00th=[  450], 20.00th=[  474],
     | 30.00th=[  494], 40.00th=[  516], 50.00th=[  548], 60.00th=[  604],
     | 70.00th=[  756], 80.00th=[ 1144], 90.00th=[ 1960], 95.00th=[ 2544],
     | 99.00th=[ 3632], 99.50th=[ 4448], 99.90th=[ 6496], 99.95th=[ 8096],
     | 99.99th=[34560]

Bandwidth就比较好看了，只是per有些疑问TODODO:
    bw (KB  /s): min= 2691, max= 6088, per=99.97%, avg=4483.69, stdev=724.50

接下来是延迟的分布，也就是32.24%的请求小于500us，37.59%的请求小于750us，以此类推。TODODO:(37.59%是不是大于500us且小于500us的)
    lat (usec) : 500=32.24%, 750=37.59%, 1000=7.10%
    lat (msec) : 2=13.41%, 4=8.95%, 10=0.68%, 20=0.01%, 50=0.01%
    lat (msec) : 100=0.01%

CPU使用率、上下文切换context switch，以及major/minor page fault，如果使用 direct IO 时，page fault 会比较少。
  cpu          : usr=0.42%, sys=2.05%, ctx=32789, majf=0, minf=22
 
这个IO depths实际上完全是应用的行为，也就是一次扔给系统的IO请求数，更过的请求实际内核可以利用电梯算法等进行优化处理。
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%

https://tobert.github.io/post/2014-04-17-fio-output-explained.html

https://segmentfault.com/a/1190000003880571

https://www.linux.com/learn/inspecting-disk-io-performance-fio

应用使用IO通常有二种方式：同步和异步。 同步的IO一次只能发出一个IO请求，等待内核完成才返回，这样对于单个线程iodepth总是小于1，但是可以透过多个线程并发执行来解决，通常我们会用16-32根线程同时工作把iodepth塞满。 异步的话就是用类似libaio这样的linux native aio一次提交一批，然后等待一批的完成，减少交互的次数，会更有效率。


http://blog.yufeng.info/archives/2104  包括了Cgroup的隔离、MySQL模拟等等

-->


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





## 其它

### ionice

获取或设置程序的 IO 调度与优先级。

{% highlight text %}
ionice [-c class] [-n level] [-t] -p PID...
ionice [-c class] [-n level] [-t] COMMAND [ARG]

----- 获取进程ID为89、91的IO优先级
$ ionice -p 89 91
{% endhighlight %}

<!--
<P><STRONG>ilde</STRONG><STRONG>：</STRONG>空闲磁盘调度，该调度策略是在当前系统没有其他进程需要进行磁盘IO时，才能进行磁盘；因此该策略对当前系统的影响基本为0；当然，该调度策略不能带有任何优先级参数；目前，普通用户是可以使用该调度策略（自从内核2.6.25开始）。</P>
<P><STRONG>Best effort</STRONG><STRONG>：</STRONG>是缺省的磁盘IO调度策略；(1)该调度策略可以指定优先级参数(范围是0~7，数值越小，优先级越高)；(2)针对处于同一优先级的程序将采round-robin方式；(3)对于best effort调度策略，8个优先级等级可以说明在给定的一个调度窗口中时间片的大小。(4)目前，普调用户(非root用户)是可以使用该调度策略。(5)在内核2.6.26之前，没有设置IO优先级的进程会使用“none”作为调度策略，但是这种策略使得进程看起来像是采用了best effort调度策略，因为其优先级是通过关于cpu nice有关的公式计算得到的：io_priority = (cpu_nice + 20) / 5。(6)在内核2.6.26之后，如果当前系统使用的是CFQ调度器，那么如果进程没有设置IO优先级级别，将采用与内核2.6.26之前版本同样的方式，推到出io优先级级别。</P>
<P><STRONG>Real time</STRONG><STRONG>：</STRONG>实时调度策略，如果设置了该磁盘IO调度策略，则立即访问磁盘，不管系统中其他进程是否有IO。因此使用实时调度策略，需要注意的是，该访问策略可能会使得其他进程处于等待状态。</P>

<P>&nbsp;</P>
<P><STRONG>参数说明：</STRONG></P>
<P>-c class ：class表示调度策略，其中0 for none, 1 for real time, 2 for best-effort, 3 for idle。</P>
<P>-n classdata：classdata表示IO优先级级别，对于best effort和real time，classdata可以设置为0~7。</P>
<P>-p pid：指定要查看或设置的进程号或者线程号，如果没有指定pid参数，ionice will run the listed program with the given parameters。</P>
<P>-t ：忽视设置优先级时产生的错误。</P>
<P>COMMAND：表示命令名</P>
<P>&nbsp;</P>
<P><STRONG>实例：</STRONG></P>
<P># ionice -c 3 -p 89</P>
<P>设置进程号为89的进程的调度策略是idle。</P>
<P>&nbsp;</P>
<P># ionice -c 2 -n 0 bash</P>
<P>运行bash，调度策略是best-effort，最高优先级。</P>
<P>&nbsp;</P>
<P># ionice -p 89 91</P>
<P>打印进程号为89和91进程的调度策略和IO优先级。</P>
<P>&nbsp;</P>
<P>#ionice -c3 -p$$</P>
<P>将当前的进程(就是shell)磁盘IO调度策略设置为idle类型.</P>
-->



## 参考

关于 FIO 可以查看源码中的 [HOWTO](/reference/linux/monitor/HOWTO.txt)，其它的压测工具可以参考 [Benchmarking](https://wiki.mikejung.biz/Benchmarking) ，或者参考 [本地文档](/reference/linux/monitor/Benchmarking.mht)，该网站还包括了很多有用文章。

[Block IO Layer Tracing: blktrace](/reference/linux/monitor/gelato_ICE06apr_blktrace_brunelle_hp.pdf) 介绍 blktrace 命令的使用；关于内核的 trace 功能参考 [Kernel Trace Systems](http://elinux.org/Kernel_Trace_Systems) 。

[Linux Block IO: Introducing Multi-queue SSD Access on Multi-core Systems](/reference/linux/monitor/linux-block-io-multi-queue-ssd.pdf) 。

<!--
http://www.cnblogs.com/quixotic/p/3258730.html
-->



 

{% highlight text %}
{% endhighlight %}
