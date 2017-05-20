---
title: Linux 监控之 Memory
layout: post
comments: true
language: chinese
category: [linux]
keywords: linux,monitor,监控,内存,memory
description:
---


<!-- more -->

<!--

# 简介

在 Linux 的内存分配机制中，优先使用物理内存，当物理内存还有空闲时（还够用），不会释放其占用内存，就算占用内存的程序已经被关闭了，该程序所占用的内存用来做缓存使用，这样对于开启过的程序、或是读取刚存取过得数据会比较快，可以提高整体 I/O 效率。

为了提高磁盘存取效率，Linux 做了一些精心的设计，除了对 dentry 进行缓存（用于 VFS，加速文件路径名到 inode 的转换），还采取了两种主要 Cache 方式：Buffer Cache 和 Page Cache。前者针对磁盘块的读写，后者针对文件 inode 的读写。这些 Cache 有效缩短了 I/O 系统调用（比如 read，write，getdents）的时间。

在 Linux 缓存机制中存在两种，分别为 buffer 和 cache，主要区别为：A buffer is something that has yet to be "written" to disk. A cache is something that has been "read" from the disk and stored for later use。
具体的操作如下：<ul><li>
    当从磁盘读取到内存的数据在被相关应用程序读取后，如果有剩余内存，则这部分数据会存入cache，以备第2次读取时，避免重新读取磁盘。</li><br><li>
    当一个应用程序在内存中修改过数据后，因为写入磁盘速度相对较低，在有空闲内存的情况下，这些数据先存入buffer，在以后某个时间再写入磁盘，从而应用程序可以继续后面的操作，而不必等待这些数据写入磁盘的操作完成。</li><br><li>
    如果在某个时刻，系统需要更多的内存，则会把cache部分擦除，并把buffer中的内容写入磁盘，从而把这两部分内存释放给系统使用，这样再次读取cache中的内容时，就需要重新从磁盘读取了。</li></ul>
</p>
-->

## free 命令

内存使用情况可以通过 free 命令查看，在 CentOS 中，是 procps-ng 包中的一个程序，该命令实际是读取 /proc/meminfo 文件，对应内核源码中的 fs/proc/meminfo.c。

{% highlight text %}
$ free
         total       used       free     shared     buffers      cached
Mem:    386024     377116       8908          0       21280      155468
-/+ buffers/cache: 200368     185656
Swap:    393552        0      393552
{% endhighlight %}

最新版本的 free 会将 buffer/cache 展示在一起，可以通过 free -w 分开查看 buffers 以及 cache；默认的单位是 KB 。

{% highlight text %}
Mem 表示物理内存统计
    total  : 物理内存总大小；
    used   : 已经分配的内存总计（含buffers 与cache ），但其中部分可能尚未使用；
    free   : 未被分配的内存；
    shared : 多个进程共享的内存总额；
    buffers: 系统分配但未被使用的 buffers 数量，磁盘buffers数量；
    cached: 系统分配但未被使用的 cache 数量，磁盘cache数量；

-/+ buffers/cached 表示物理内存的缓存统计
    used: 实际使用的 buffers 与 cache 总量(used-buffers-cached)，也是实际使用的内存总量；
    free: 未被使用的 buffers 与 cache 和未被分配的内存之和(buffers+cached+free)，这就是系统当前实际可用内存；

Swap 表示硬盘上交换分区的使用情况
    只有 mem 被当前进程实际占用完，即没有了 buffers 和 cache 时，才会使用到 swap 。
{% endhighlight %}

其实第一行可以从操作系统的角度去看，即 OS 认为自己总共有 total 的内存，分配了 used 的内存，还有 free 的内存；其中分配的内存中分别有 buffers 和 cached 的内存还没有使用，也就是说 OS 认为还有空闲的内存，又懒得收回之前分配，但已经分配的不能再使用了。

第二行可以认为从进程的角度去看，可用的内存包括了空闲的+buffers+cached，total-空闲的就是时用的内存数。

可以通过如下方式计算：

{% highlight text %}
----- 实际可用内存大小
Free(-/+ buffers/cache) = Free(Mem) + buffers(Mem) + Cached(Mem)
                 185656 =      8908 +        21280 +      155468

----- 物理内存总大小
total(Mem) = used(-/+ buffers/cache) + free(-/+ buffers/cache)
    386024 =                  200368 +                  185656
           = used(Mem) + free(Mem)
           =    377116 +      8908

----- 已经分配的内存大小
Used(Mem) = Used(-/+ buffers/cache) + buffers(Mem) + Cached(Mem)
   377116 =                  200368 +        21280 +     155468
{% endhighlight %}

关于内存的其它信息可以参考如下文件。

{% highlight text %}
/proc/meminfo                  // 机器的内存使用信息
/proc/pid/maps                 // 显示进程 PID 所占用的虚拟地址
/proc/pid/statm                // 进程所占用的内存
/proc/self/statm               // 当前进程的信息
{% endhighlight %}

#### 新版

如上所述，新版的命令行输入如下。

{% highlight text %}
$ free -w
              total        used        free      shared     buffers       cache   available
Mem:        8070604     2928928      495012     1162676      104156     4542508     3624840
Swap:       8127484      232388     7895096

total     (MemTotal     @ /proc/meminfo)
free      (MemFree      @ /proc/meminfo)
shared    (Shmem        @ /proc/meminfo)  主要是tmpfs使用
buffers   (Buffers      @ /proc/meminfo)
cache     (Cached+Slab  @ /proc/meminfo)
available (MemAvailable @ /proc/meminfo)
used = total - free - buffers - cache     (注意，没有去除shared)
{% endhighlight %}

可以通过 ```free -k -w && cat /proc/meminfo``` 命令进行测试。


### Swap对性能的影响

当内存不足的时候把磁盘的部分空间当作虚拟内存来使用，而实际上并非是在内存不足的时候才使用，有个参数叫做 swappiness，是用来控制 swap 分区使用的，可以直接查看 ```/proc/sys/vm/swappiness``` 文件。

默认值是 60，范围是 [0, 100]；为 0 表示会最大限度使用物理内存，然后才是 swap；为 100 时，表示尽量使用 swap 分区，把内存上的数据及时的搬运到 swap 空间里面。

分配太多的 Swap 空间会浪费磁盘空间，而 Swap 空间太少，则系统有可能会发生错误。当系统的物理内存用光了，系统就会跑得很慢，但仍能运行；但是如果 Swap 空间也用光了，那么系统就会发生错误。

通常情况下，Swap 空间是物理内存的 2~2.5 倍，最小为 64M；这也根据不同的应用，有不同的配置：如果是桌面系统，则只需要较小的 Swap 空间；而大的服务器则视情况不同需要不同大小的 Swap 空间，一般来说对于 4G 以下的物理内存，配置 2 倍的 swap ，4G 以上配置 1 倍。

另外， Swap 分区的数量对性能也有很大的影响。因为 Swap 交换的操作是磁盘 IO 的操作，如果有多个 Swap 交换区， Swap 空间的分配会以轮流的方式操作于所有的 Swap ，这样会大大均衡 IO 的负载，加快 Swap 交换的速度。

实际上，当空闲物理内存不多时，不一定表示系统运行状态很差，因为内存的 cache 及 buffer 部分可以随时被重用。**只有 swap 被频繁调用，才是内存资源是否紧张的依据，可以通过 vmstat 查看**。

#### 状态查看

可以使用 [getswapused1.sh](/reference/linux/monitor/getswapused1.sh) 和 [getswapused2.sh](/reference/linux/monitor/getswapused2.sh) 查看 swap 的使用情况，如果不使用 root 用户则只会显示当前用户的 swap，其它用户显示为 0。

可以通过如下的命令排序，查看使用最多的进程，```./getswap.sh | egrep -v "Swap used: 0" | sort -n -k 5``` 。


<!--
<br><br><h2>测试</h2><p>
通过执行 dd 查看缓存的变化。
<pre style="font-size:0.8em; face:arial;">
# free -w && dd if=/dev/zero of=foobar bs=4k count=40960 && free -w          // 写入168M
</pre>
可以看到 cache 增长的数目与文件大小基本相符，而且 buffers 同样由所增长；然后，通过 linux-ftools 中的工具查看。
<pre style="font-size:0.8em; face:arial;">
# linux-fincore foobar                    // cached_perc为100%
</pre>
该命令的执行过程大致如下。
<pre style="font-size:0.8em; face:arial;">
main()
  |-getpagesize()                         // 获取系统中页的大小
  |-open(path, O_RDONLY)                  // 只读方式打开
  |-fstat(fd, &file_stat)                 // 查看文件属性，主要是大小
    ....
    file_mmap = mmap((void *)0, file_stat.st_size, PROT_NONE, MAP_SHARED, fd, 0 );
    size_t calloc_size = (file_stat.st_size+page_size-1)/page_size;
    mincore_vec = calloc(1, calloc_size);
    if ( mincore(file_mmap, file_stat.st_size, mincore_vec) != 0 )
    ....
    if (mincore_vec[page_index]&1) {

            ++cached;
</pre>
页面的大小是4K ，有了长度，我们就知道了，我们需要多少个int来存放结果。mmap建立映射关系，mincore获取文件页面的驻留情况，从起始地址开始，长度是filesize，结果保存在mincore_vec数组里。如果mincore[page_index]&1 == 1,那么表示该页面驻留在内存中，否则没有对应缓存。
-->

## 常用命令

介绍下与内存相关的命令。

### vmstat

vmstat 不会将自己作为一个有效的进程。

{% highlight text %}
$ vmstat [options] [delay [count]]

常用选项：
  delay: 每多少秒显示一次。
  count: 显示多少次，默认为一次。
  -a: 显示活跃和非活跃内存。
  -f: 显示从系统启动之后 forks 的数量；包括了 fork, vfork, clone 的系统调用，等价于创建的全部任务数。
  -m,--slabs: 显示 slabinfo 。
  -n,--one-header: 只显示一次，而不是周期性的显示。
  -s,--stats: 显示内存统计以及各种活动信息。
  -d,--disk: 显示磁盘的相关统计。
  -D,--disk-sum: 显示硬盘统计信息的摘要。
  -p,--partition device: 显示某个磁盘的统计信息。
  -S,--unit char: 显示输出时的单位，1000(k),1024(K),1000*1000(m),1024*1024(M)，默认是 K，不会改变 swap 和 block 。

----- 每秒刷新一次
$ vmstat -S M 1
procs -----------memory---------- ---swap-- -----io---- --system-- ----cpu----
r  b   swpd   free   buff  cache   si   so    bi    bo   in    cs us sy id wa
3  0      0   1963    607   2359    0    0     0     0    0     1 32  0 68  0

各个段的含义为：
  procs 进程
    r: 等待运行的进程数，如果运行队列过大，表示CPU很忙，一般会造成CPU使用率很高。
    b: 不可中断睡眠(uninterruptible sleep)的进程数，也就是被阻塞的进程，通常为IO。
  memory 内存
    swpd: 虚拟内存(Virtual Memory)的使用量，应该是交换区。</li><li>
    free: 空闲内存数，同free命令中的free(Mem)。</li><li>
    buff: 同free命令，已分配未使用的内存大小。</li><li>
    cache: 同free命令，已分配未使用的内存大小。</li><li>
    inact: 非活跃(inactive)内存的大小，使用 -a 选项????。</li><li>
    active: 活跃(active)内存的大小，使用 -a 选项????。
  swap 交换区
    si: 每秒从交换区(磁盘)写到内存的大小。
    so: 每秒写入交换区(磁盘)的大小。
        如果这两个值长期大于0，系统性能会受到影响，即我们平常所说的Thrashing(颠簸)。
  IO
    bi: 每秒读取的块数(blocks/s)。
    bo: 每秒写入的块数(blocks/s)。
  system 系统
    in: 每秒的中断数，包括时钟中断。
    cs: 每秒上线文的交换数。
  CPU
    us: 用户进程执行时间。
    sy: 系统进程执行时间。
    id: 空闲进程执行时间，2.5.41 之后含有 IO-wait 的时间。
    wa: 等待 IO 的时间，2.5.41 之后含有空闲任务。
{% endhighlight %}


<!--
<br><br><h2>源码分析</h2><p>
通过 vmstat -a 命令能看到 active memory 和 inactive memory，这两个选项的解释也不太清楚，可以通过源码查看。vmstat 实际会从 /proc/meminfo 读取，其源码实现在 fs/proc/meminfo.c。
<pre style="font-size:0.8em; face:arial;">
$ grep -i active /proc/meminfo

$ cat fs/proc/meminfo.c
... ...
static int meminfo_proc_show(struct seq_file *m, void *v)
{
    ... ...
    for (lru = LRU_BASE; lru < NR_LRU_LISTS; lru++)
        pages[lru] = global_page_state(NR_LRU_BASE + lru);


        "Active:         %8lu kB\n"
        "Inactive:       %8lu kB\n"
        "Active(anon):   %8lu kB\n"
        "Inactive(anon): %8lu kB\n"
        "Active(file):   %8lu kB\n"
        "Inactive(file): %8lu kB\n"


        K(pages[LRU_ACTIVE_ANON]   + pages[LRU_ACTIVE_FILE]),
        K(pages[LRU_INACTIVE_ANON] + pages[LRU_INACTIVE_FILE]),
        K(pages[LRU_ACTIVE_ANON]),
        K(pages[LRU_INACTIVE_ANON]),
        K(pages[LRU_ACTIVE_FILE]),
        K(pages[LRU_INACTIVE_FILE]),

}
</pre>
这段代码是统计所有的 LRU list，为了可以达到“回收的页面应该是最近使用得最少的”效果，最理想的情况是每个页面都有一个年龄项，用于记录最近一次访问页面的时间，可惜 x86 CPU 硬件并不支持这个特性，x86 CPU 只能做到在访问页面时设置一个标志位 Access Bit，无法记录时间。<br><br>

所以 Linux 内核使用了一个折衷的方法：它采用了 LRU list 列表，把刚访问过的页面放在列首，越接近列尾的就是越长时间未访问过的页面，这样，虽然不能记录访问时间，但利用页面在 LRU list 中的相对位置也可以轻松找到年龄最长的页面。<br><br>

Linux 内核设计了两种 LRU list: active list 和 inactive list，刚访问过的页面放进 active list，长时间未访问过的页面放进 inactive list，这样从 inactive list 回收页面就变得简单了。<br><br>

内核线程 kswapd 会周期性地把 active list 中符合条件的页面移到 inactive list 中，这项转移工作是由 refill_inactive_zone() 完成的。
-->

### vmtouch

通过 [vmtouch offical site](http://hoytech.com/vmtouch/) 可以查看文件在内存中的占比，可以下载 [本地源码](/reference/linux/monitor/vmtouch.c) 。

使用参数可以直接通过 ```vmtouch -h``` 查看，如下是简单的示例：

{% highlight text %}
----- 查看/etc目录下有多少在缓存中
$ vmtouch /etc/
           Files: 2844
     Directories: 790
  Resident Pages: 274/9971  1M/38M  2.75%
         Elapsed: 0.15243 seconds

----- 查看一个大文件在缓存中的比例
$ vmtouch -v big-dataset.txt
big-dataset.txt
[                                                            ] 0/169321

           Files: 1
     Directories: 0
  Resident Pages: 0/169321  0/661M  0%
         Elapsed: 0.011822 seconds

----- 缓存部分内容到内存中，分别对应了文本和二进制文件
$ tail -n 10000 big-dataset.txt > /dev/null
$ hexdump -n 102400 big-dataset.txt > /dev/null

----- 再次查看缓存的文件，也就是62页在内存中
$ vmtouch -v big-dataset.txt
big-dataset.txt
[o                                                           ] 62/169321

           Files: 1
     Directories: 0
  Resident Pages: 62/169321  248K/661M  0.0366%
         Elapsed: 0.003463 seconds

----- 将文件加载到内存中
$ vmtouch -vt big-dataset.txt
big-dataset.txt
[OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO] 169321/169321
           Files: 1
     Directories: 0
   Touched Pages: 169321 (661M)
         Elapsed: 1.7372 seconds

----- 手动从内存中删除
$ vmtouch -ve big-dataset.txt
Evicting big-dataset.txt
           Files: 1
     Directories: 0
   Evicted Pages: 42116 (164M)
         Elapsed: 0.076824 seconds

----- 将目录下的所有文件锁定到内存中
$ vmtouch -dl /var/www/htdocs/critical/
{% endhighlight %}

在 Linux 中，主要是通过 posix_fadvise() 来清除缓存；通过 mincore() 判断页是否存在于缓存中。


<!---
Linux 内核模式使用 Buffer IO，以充分使用操作系统的 Page Cache；读取时先检查页缓存里面是否有需要的数据，没有就从设备读取，返回给用户的同时，加到缓存一份；写时，写到缓存后返回系统调用，再由后台的进程定期涮到磁盘去。

但是如果你的IO非常密集，就会出现问题。首先由于pagesize是4K，内存的利用效率比较低。其次缓存的淘汰算法很简单，由操作系统自主进行，用户不大好参与。当你的写很多，超过系统内存的某个上限的时候，后台的进程(swapd)要出来回收页面，而且一旦回收的速度小于写入的速度，就会出现不可预期的行为。

这里面最大的问题是：当你使用的内存包括缓存，没超过操作系统规定的上限的时候，操作系统选择不作为，让用户充分使用缓存，从它的角度来看这样效率最高。但是正是由于这种策略在实践中会导致问题。

比如说MYSQL服务器，我们可以把数据直接走direct IO,但是它的日志是走bufferio的。因为走directio需要对写入文件的偏移和大小都要扇区对全，这对日志系统来讲太麻烦了。由于MYSQL是基于事务的，会涉及到大量的日志动作，频繁的写入，然后fsync. 日志一旦写入磁盘，buffer page就没用了，但是一直会在内存呆着，直到达到内存上限，引起操作系统突然大量回收
页面，出现IO柱塞或者内存交换等负面问题。

那么我们知道了困境在哪里，我们可以主动避免这个现象的发生。有二种方法：
1. 日志也走direct io,需要规模的修改MYSQL代码，如percona就这么做了，提供相应的patch。
2. 日志还是走buffer io, 但是定期清除无用page cache.

第一张方法不是我们要讨论的，我们重点讨论第二种如何做：

我们在程序里知道文件的句柄，是不是就可以很轻松的用：

    int posix_fadvise(int fd, off_t offset, off_t len, int advice);
    POSIX_FADV_DONTNEED The specified data will not be accessed in the near future.

来解决问题呢？
比如写类似 posix_fadvise(fd, 0, len_of_file, POSIX_FADV_DONTNEED)；这样的代码来清掉文件所属的缓存。

前面介绍的vmtouch就有这样的功能，清某个文件的缓存。
vmtouch -ve logfile 就可以试验，但是你会发现内存根本就没下来，原因呢？
-->









## 其它

### 刷新内存

可以通过手动执行 sync 命令刷新内存，以确保文件系统的完整性，sync 命令将所有未写的系统缓冲区写到磁盘中，包含已修改的 i-node、已延迟的块 I/O 和读写映射文件。

{% highlight text %}
# sync; echo 1 > /proc/sys/vm/drop_caches              // 仅清除页面缓存PageCache
# sync; echo 2 > /proc/sys/vm/drop_caches              // 清除目录项和inode
# sync; echo 3 > /proc/sys/vm/drop_caches              // 清除页面缓存，目录项和inode
{% endhighlight %}

其中第一个是比较安全的，可以在生产环境中使用；最后一个操作比较危险，不建议在生产环境中使用，除非你自己清楚在做什么，源码实现在 fs/drop_caches.c 中。实际上，上述的操作通常针对 IO 的基准测试；否则不建议清除缓存。

可以通过如下命令清除掉交换区的空间。

{% highlight text %}
# swapoff -a && swapon -a
{% endhighlight %}

### 进程内存使用量

简单介绍下如何查看各个进程的内存使用量。

#### top

可以直接使用 top 命令后，查看 %MEM 的内容，表示该进程内容使用比例，可以通过 -u 指定用户，也就是只监控特定的用户。

常用的命令有。

{% highlight text %}
P: 按%CPU使用率排行
T: 按TIME+排行
M: 按%MEM排行
{% endhighlight %}

#### ps

如下例所示：

{% highlight text %}
$ ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid' | sort -nrk5
{% endhighlight %}

其中 rsz 为实际内存，上例实现按内存排序，由大到小。

#### pmap

可以根据进程查看进程相关信息占用的内存情况，可以通过 ```-x```、```-X```、```-XX``` 查看详细信息。

{% highlight text %}
$ pmap -d 14596
输出：
  address :  映像起始地址；
  kbytes  :  映像大小；
  RSS     :  (Resident Set Size)驻留集大小，单位是K；
  dirty   :  脏页大小，包括了共享和私有，单位是K；
  mode    :  映像权限(r=read, w=write, x=execute, s=shared, p=private)；
  mapping :  映像支持文件，[anon]为分配的内存，[stack]为程序堆栈；
  offset  :  文件偏移；
  device  :  设备名。
{% endhighlight %}

## 小结

可以通过 free 查看整个系统内存的使用情况，free(Mem) 过小不会对系统的性能产生影响，实际剩余的内存数为 free(-/+ buffers/cache)。

通过 vmstat 1 3 输出的 swap(si/so) 可以查看交换区的切入/出情况，如果长期不为 0，则可能是因为内存不足，此时应该查看那个进程占用内存较多。


## Cache VS. Buffer

简单说，Buffer 的核心作用是用来缓冲，缓和冲击。比如你每秒要写 100 次硬盘，对系统冲击很大，浪费了大量时间在忙着处理开始写和结束写这两件事。用个 buffer 暂存起来，变成每 10 秒写一次硬盘，对系统的冲击就很小，写入效率高了，极大缓和了冲击。

Cache 的核心作用是加快取用的速度。比如你一个很复杂的计算做完了，下次还要用结果，就把结果放手边一个好拿的地方存着，下次不用再算了，加快了数据取用的速度。

所以，如果你注意关心过存储系统的话，你会发现硬盘的读写缓冲/缓存名称是不一样的，叫 write-buffer 和 read-cache；很明显地说出了两者的区别。

<!--
当然很多时候宏观上说两者可能是混用的。比如实际上memcached很多人就是拿来读写都用的。不少时候Non-SQL数据库也是。严格来说，CPU里的L2和L3 Cache也都是读写兼用——因为你没法简单地定义CPU用它们的方法是读还是写。硬盘里也是个典型例子，buffer和cache都在一块空间上，到底是buffer还是cache？

不过仔细想一下，你说拿cache做buffer用行不行？当然行，只要能控制cache淘汰逻辑就没有任何问题。那么拿buffer做cache用呢？貌似在很特殊的情况下，能确定访问顺序的时候，也是可以的。简单想一下就明白——buffer根据定义，需要随机存储吗？一般是不需要的。但cache一定要。所以大多数时候用cache代替buffer可以，反之就比较局限。这也是技术上说cache和buffer的关键区别。

https://www.zhihu.com/question/26190832/answer/146259979
-->

那么 free 命令中的 buffers 和 cache 是什么意思？

### free

新版的 free 命令输出如下。

{% highlight text %}
$ free -wm
              total        used        free      shared     buffers       cache   available
Mem:           7881        3109         797        1113         309        3665        3310
Swap:          7936         260        7676
{% endhighlight %}

* buffers，表示块设备 (block device) 所占用的缓存页，包括了直接读写块设备以及文件系统元数据 (metadata) 比如 SuperBlock 所使用的缓存页；
* cached，表示普通文件数据所占用的缓存页。

该命令读取的是 ```/proc/meminfo``` 文件中的 Buffers 和 Cached 数据，而在内核中的实现实际上对应了 ```meminfo_proc_show()@fs/proc/meminfo.c``` 函数，内容如下。

{% highlight c %}
static int meminfo_proc_show(struct seq_file *m, void *v)
{
    si_meminfo(&i);  // 通过nr_blockdev_pages()函数填充bufferram
    si_swapinfo(&i);

    cached = global_page_state(NR_FILE_PAGES) -
            total_swapcache_pages() - i.bufferram;
    if (cached < 0)
        cached = 0;

    seq_printf(m,
        "Buffers:        %8lu kB\n"
        "Cached:         %8lu kB\n"
        // ... ..
        K(i.bufferram),
        K(cached),
        // ... ..
}
{% endhighlight %}

如上计算 cached 公式中，```global_page_state(NR_FILE_PAGES)``` 实际读取的 ```vmstat[NR_FILE_PAGES]```，也就是用于统计所有缓存页 (page cache) 的总和，它包括：

<!--
mm/filemap.c:200:       __dec_zone_page_state(page, NR_FILE_PAGES);
mm/filemap.c:488:               __inc_zone_page_state(new, NR_FILE_PAGES);
mm/filemap.c:580:       __inc_zone_page_state(page, NR_FILE_PAGES);
mm/swap_state.c:105:            __inc_zone_page_state(page, NR_FILE_PAGES);
mm/swap_state.c:157:    __dec_zone_page_state(page, NR_FILE_PAGES);
mm/shmem.c:318:         __inc_zone_page_state(page, NR_FILE_PAGES);
mm/shmem.c:341: __dec_zone_page_state(page, NR_FILE_PAGES);
mm/shmem.c:1003:                __inc_zone_page_state(newpage, NR_FILE_PAGES);
mm/shmem.c:1004:                __dec_zone_page_state(oldpage, NR_FILE_PAGES);
mm/migrate.c:414:       __dec_zone_page_state(page, NR_FILE_PAGES);
mm/migrate.c:415:       __inc_zone_page_state(newpage, NR_FILE_PAGES);

    Cached
    buffers
    交换区缓存(swap cache)
-->


> swap cache 主要是针对匿名内存页，例如用户进程通过malloc()申请的内存页，当要发生swapping换页时，如果一个匿名页要被换出时，会先计入到swap cache，但是不会立刻写入物理交换区，因为Linux的原则是除非绝对必要，尽量避免IO。
>
> 所以swap cache中包含的是被确定要swapping换页，但是尚未写入物理交换区的匿名内存页。

<!--
vmstat[NR_FILE_PAGES] 可以通过 /proc/vmstat 来查看，表示所有缓存页的总数量：
# cat /proc/vmstat
...
nr_file_pages 587334
...


注意以上nr_file_pages是以page为单位，而不像free命令是以KB为单位，一个page等于4KB。

    直接修改 nr_file_pages 的内核函数是：
    __inc_zone_page_state(page, NR_FILE_PAGES) 和
    __dec_zone_page_state(page, NR_FILE_PAGES)，
    一个用于增加，一个用于减少。


先看”cached”：

“Cached” 就是除去 “buffers” 和 “swap cache” 之外的缓存页的数量：
global_page_state(NR_FILE_PAGES) – total_swapcache_pages – i.bufferram
所以关键还是要理解 “buffers” 是什么含义。


#### buffers

从源码中可以看到 buffers 来自于 nr_blockdev_pages() 的返回值，该函数如下：

{% highlight c %}
long nr_blockdev_pages(void)
{
    struct block_device *bdev;
    long ret = 0;
    spin_lock(&bdev_lock);
    list_for_each_entry(bdev, &all_bdevs, bd_list) {
        ret += bdev->bd_inode->i_mapping->nrpages;
    }
    spin_unlock(&bdev_lock);
    return ret;
}
{% endhighlight %}

这段代码就是遍历所有的块设备 (block device)，累加每个块设备的 inode 的 i_mapping 的页数，统计得到的就是 buffers 。所以很明显，buffers 是与块设备直接相关的。

那么谁会更新块设备的缓存页数量(nrpages)呢？我们继续向下看。

搜索kernel源代码发现，最终更新mapping->nrpages字段的函数就是add_to_page_cache和__remove_from_page_cache：

static inline int add_to_page_cache(struct page *page,
                struct address_space *mapping, pgoff_t offset, gfp_t gfp_mask)
{
        int error;

        __set_page_locked(page);
        error = add_to_page_cache_locked(page, mapping, offset, gfp_mask);
        if (unlikely(error))
                __clear_page_locked(page);
        return error;
}

void remove_from_page_cache(struct page *page)
{
        struct address_space *mapping = page->mapping;
        void (*freepage)(struct page *) = NULL;
        struct inode *inode = mapping->host;

        BUG_ON(!PageLocked(page));

        if (IS_AOP_EXT(inode))
                freepage = EXT_AOPS(mapping->a_ops)->freepage;

        spin_lock_irq(&mapping->tree_lock);
        __remove_from_page_cache(page);
        spin_unlock_irq(&mapping->tree_lock);
        mem_cgroup_uncharge_cache_page(page);

        if (freepage)
                freepage(page);
}

page_cache_tree_insert()

__delete_from_page_cache()
 |-page_cache_tree_delete() --

这两个函数是通用的，block device 和 文件inode 都可以调用，至于更新的是块设备的(buffers)还是文件的(cached)，取决于调用参数变量mapping：如果mapping对应的是文件inode，自然就不会影响到 “buffers”；如果mapping对应的是块设备，那么相应的统计信息会反映在 “buffers” 中。我们下面看看kernel中哪些地方会把块设备的mapping传递进来。

搜索内核源代码发现，ext4_readdir 函数调用 page_cache_sync_readahead 时传递的参数是 sb->s_bdev->bd_inode->i_mapping，其中s_bdev就是块设备，也就是说在读目录(ext4_readdir)的时候可能会增加 “buffers” 的值：
static int ext4_readdir(struct file *filp,
                         void *dirent, filldir_t filldir)
{

...
        struct super_block *sb = inode->i_sb;
...
                        if (!ra_has_index(&filp->f_ra, index))
                                page_cache_sync_readahead(
                                        sb->s_bdev->bd_inode->i_mapping,
                                        &filp->f_ra, filp,
                                        index, 1);
...
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14

static int ext4_readdir(struct file *filp,
                         void *dirent, filldir_t filldir)
{

...
        struct super_block *sb = inode->i_sb;
...
                        if (!ra_has_index(&filp->f_ra, index))
                                page_cache_sync_readahead(
                                        sb->s_bdev->bd_inode->i_mapping,
                                        &filp->f_ra, filp,
                                        index, 1);
...
}

继续琢磨上面的代码，sb表示SuperBlock，属于文件系统的metadata（元数据），突然间一切恍然大悟：因为metadata不属于文件，没有对应的inode，所以，对metadata操作所涉及的缓存页都只能利用块设备mapping，算入 buffers 的统计值内。

    打个岔：ext4_readdir() 中调用 page_cache_sync_readahead() 显然是在进行预读(read-ahead)，为什么read-ahead没有使用普通文件inode的mapping，而是使用了底层的块设备呢？从记载在补丁中的说明来看，这是一个权宜之计，看这里，所以不必深究了。

举一反三，如果文件含有间接块(indirect blocks)，因为间接块属于metadata，所以走的也是块设备的mapping。查看源代码，果然如此：
ext4_get_blocks
->  ext4_ind_get_blocks
    ->  ext4_get_branch
        ->  sb_getblk

static inline struct buffer_head *
sb_getblk(struct super_block *sb, sector_t block)
{
        return __getblk(sb->s_bdev, block, sb->s_blocksize);
}
1
2
3
4
5
6
7
8
9
10

ext4_get_blocks
->  ext4_ind_get_blocks
    ->  ext4_get_branch
        ->  sb_getblk

static inline struct buffer_head *
sb_getblk(struct super_block *sb, sector_t block)
{
        return __getblk(sb->s_bdev, block, sb->s_blocksize);
}

这样，我们就知道了，”buffers” 是块设备(block device)占用的缓存页，分为两种情况：

    直接对块设备进行读写操作；
    文件系统的metadata（元数据），比如 SuperBlock。
-->

#### 验证

如上，读取 EXT4 文件系统的目录会使用到 "buffers"，这里使用 find 命令扫描文件系统，观察 "buffers" 增加的情况：

{% highlight text %}
# sync
# echo 3 > /proc/sys/vm/drop_caches
$ free -wk; find ~ -name "not exits file" >/dev/null 2>&1; free -wk
              total        used        free      shared     buffers       cache   available
Mem:        8070604     3260408     3445852     1102588        5236     1359108     3418844
Swap:       8127484      300172     7827312
              total        used        free      shared     buffers       cache   available
Mem:        8070604     3249764     3207336     1087716      250484     1363020     3417764
Swap:       8127484      300172     7827312
{% endhighlight %}

再测试下直接读取 block device 并观察 "buffers" 增加的现象：

{% highlight text %}
# sync
# echo 3 > /proc/sys/vm/drop_caches
# free -wk; dd if=/dev/sda1 of=/dev/null count=200M; free -wk
              total        used        free      shared     buffers       cache   available
Mem:        8070604     3244516     3486124     1094648         932     1339032     3451048
Swap:       8127484      300172     7827312
532480+0 records in
532480+0 records out
272629760 bytes (273 MB) copied, 0.612241 s, 445 MB/s
              total        used        free      shared     buffers       cache   available
Mem:        8070604     3245032     3218528     1094868      267196     1339848     3427012
Swap:       8127484      300172     7827312
{% endhighlight %}

<!--
 结论：

free 命令所显示的 “buffers” 表示块设备(block device)所占用的缓存页，包括直接读写块设备、以及文件系统元数据(metadata)如SuperBlock所使用的缓存页；
而 “cached” 表示普通文件所占用的缓存页。
-->


## 参考

查看那个进程在使用交换空间可以参考 [Find Out What Process Are Using Swap Space](http://www.cyberciti.biz/faq/linux-which-process-is-using-swap/)，或者 [本地文档](/reference/linux/monitor/Find Out What Process Are Using Swap Space.maff) 。

关于内存的介绍可以参考 [What every programmer should know about memory](https://www.akkadia.org/drepper/cpumemory.pdf)，包括了从硬件到软件的详细介绍，内容非常详细。


<!-- memtester 内存测试工具。-->

<!--
可用内存计算方法
http://www.cnblogs.com/feisky/archive/2012/04/14/2447503.html

内存监控相关
https://linux-audit.com/understanding-memory-information-on-linux-systems/

一次高内存使用率的告警处理
http://farll.com/2016/10/high-memory-usage-alarm/

http://marek.vavrusa.com/c/memory/2015/02/20/memory/

http://careers.directi.com/display/tu/Understanding+and+optimizing+Memory+utilization


MemTotal:        3844344 kB
MemFree:          128712 kB   当前未使用的内存，注意，这里不包括内核中可回收内存，包括了HighFree+LowFree，需要内核配置CONFIG_HIGHMEM选项
MemAvailable:    1373376 kB   在不发生swap时的最大可用内存

Shmem:            197092 kB   一般是tmpfs使用，如/dev/shm,/run等，另外还有内核中的SysV


在计算可用内存时，最早一般使用 free+cached，而实际上 cached 中包含了


CONFIG_HIGHMEM
  部分 CPU (如ARM) 只能映射 4G 的内存管理空间，这 4G 空间包括了用户空间、内核空间、IO 空间，如果物理内存大于 4G ，那么必定有部分内存在这种情况下是无法管理的，这部分内存也就被称为 "high memory" 。

  简单来说，之所以有 high memory 是因为物理内存超过了虚拟内存，导致内核无法一次映射所有物理内存，为此就需要有临时的映射。注意，创建临时映射的成本很高，需要修改内核的 page table 以及 TLB/MMU 。

  [Kenel-doc HIGH MEMORY HANDLING](https://www.kernel.org/doc/Documentation/vm/highmem.txt)

[浅析Linux的共享内存与tmpfs文件系统](http://hustcat.github.io/shared-memory-tmpfs/)

Documentation/filesystems/proc.txt
fs/proc/meminfo.c


共享内存主要用于进程间通信，Linux 有两种方式的共享内存 (Shared Memory) 机制：A) System V shared memory(shmget/shmat/shmdt)，B)POSIX shared memory(shm_open/shm_unlink)；两者提供功能基本类似 (semaphores, shared memory, message queues)，前者由于历史原因更广泛：


### SYSV 共享内存

----- 查看以及调整支持的最大内存，shmmax shmall shmmni
$ cat /proc/sys/kernel/shmmax
33554432
----- 修改为32M
# echo 33554432 > /proc/sys/kernel/shmmax
----- 尝试创建65M的system V共享内存时失败
$ ipcmk -M 68157440
ipcmk: create share memory failed: Invalid argument


http://www.linuxatemyram.com/

http://www.361way.com/ipcs-shared-memory/5144.html

http://rubenlaguna.com/wp/2015/02/22/posix-slash-system-v-shared-memory-vs-threads-shared-memory/
https://www.ibm.com/developerworks/cn/aix/library/au-ipc/
http://blog.jqian.net/post/linux-shm.html
https://www.ibm.com/developerworks/cn/linux/l-cn-slub/
http://blog.csdn.net/bullbat/article/details/7194794
http://blog.scoutapp.com/articles/2009/07/31/understanding-load-averages
https://engineering.linkedin.com/performance/optimizing-linux-memory-management-low-latency-high-throughput-databases
http://kernel.taobao.org/index.php?title=Kernel_Documents/mm_sysctl
https://lwn.net/Articles/422291/
http://linuxperf.com/?p=142
http://blog.csdn.net/ctthuangcheng/article/details/8916065
https://www.kernel.org/doc/gorman/html/understand/understand015.html
http://blog.csdn.net/kickxxx/article/details/8618451
https://www.kernel.org/doc/gorman/html/understand/understand005.html





/* strtol example */
#include <stdio.h>      /* printf */
#include <stdlib.h>     /* strtol */

int main ()
{
  char szNumbers[] = "2001 60c0c0 -1101110100110100100000 0x6fffff";
  char * pEnd;
  long int li1, li2, li3, li4;
  li1 = strtol (szNumbers,&pEnd,10);
  li2 = strtol (pEnd,&pEnd,16);
  li3 = strtol (pEnd,&pEnd,2);
  li4 = strtol (pEnd,NULL,0);
  printf ("The decimal equivalents are: %ld, %ld, %ld and %ld.\n", li1, li2, li3, li4);
  return 0;
}

将字符串转换为长整型数，会扫描参数 nptr 字符串，跳过前面的空白字符 (如空格、tab等)，直到遇上数字或正负符号才开始做转换，然后再遇到非数字或字符串结束符('\0')结束转换，并将结果返回。

#include <stdlib.h>
long int strtol (const char *nptr, char **endptr, int base);
long long int strtoll(const char *nptr, char **endptr, int base);

参数：
  str    : 要转换的字符串；
  endstr : 第一个不能转换的字符的指针，非NULL传回不符合条件而终止的字符串指针；
  base   : 字符串str所采用的进制，范围从2~36，也可以为0(默认为10进制；遇到0x/0X前置采用16进制；遇到0前置采用8进制) 。
返回值：
  转换后的长整型数；如果不能转换或者nptr为空字符串，返回 0(0L)；
  如果转换得到的值超出long int所能表示的范围，函数将返回LONG_MAX或LONG_MIN (在limits.h中定义)，并将errno设置为ERANGE。


ANSI C 规范定义了 stof()、atoi()、atol()、strtod()、strtol()、strtoul() 共6个可以将字符串转换为数字的函数，大家可以对比学习。另外在 C99 / C++11 规范中又新增了5个函数，分别是 atoll()、strtof()、strtold()、strtoll()、strtoull()，在此不做介绍，请大家自行学习。



【示例】将字符串转换成10进制。

    #include <stdio.h>
    #include <stdlib.h>
    int main ()
    {
        char szNumbers[] = "2001 60c0c0 -1101110100110100100000 0x6fffff";
        char * pEnd;
        long int li1, li2, li3, li4;
        li1 = strtol (szNumbers,&pEnd,10);
        li2 = strtol (pEnd,&pEnd,16);
        li3 = strtol (pEnd,&pEnd,2);
        li4 = strtol (pEnd,NULL,0);
        printf ("转换成10进制: %ld、%ld、%ld、%ld\n", li1, li2, li3, li4);
        system("pause");
        return 0;
    }

执行结果：
转换成10进制: 2001、6340800、-3624224、7340031



dentry cache 和 inode cache 区别。



### 用户虚拟地址到物理地址转换

查看当前进程的虚拟地址映射 ```cat /proc/self/maps```，

通过命令你给 ```ps aux | less``` 查看内存相关参数时，会发现两个比较有疑问的指标 RSS、VSZ (单位是KB)，通过 ```man 1 ps``` 查看对应的解释如下：

 * RSS resident set size, the non-swapped physical memory that a task has used (in kiloBytes). (alias rssize, rsz).
 * VSZ virtual memory size of the process in KiB (1024-byte units). Device mappings are currently excluded; this is subject to change. (alias vsize).

简言之，RSS 就是这个进程实际占用的物理内存；VSZ 就是进程的虚拟内存，包括了还未发生缺页异常加载映射到内存中。如果通过上述的两个指标来评估实际内存使用量，那么是错误的 ！！！

ps 显示的统计结果实际上有个前提假设：如果只有这一个进程在运行。在现在的内核中，显然是不可能的，很多进程会共享一些内存，例如 libc 。




ldd


内存泄露检测。

可以通过 pmap 进行检查，原则如下：

1. Virtual Memory(VIRT) 或者 writeable/private (pmap –d) 一直在增长；



2, The RSS is just for your reference, it can’t used as the factor to detect memory leak.

3, although you call ‘free’ or ‘delete’, the virtual memory may not shrink immediately.

4, the same new operation may mapped to different [anon] virtual memory space. Here 1M was alloated to 000000001b56a000    and 00002ac25a77c000.

    000000001b56a000    1156      12      12 rw---    [ anon ]

    00002ac25a77c000    1040      16      16 rw---    [ anon ]

nm and objdump -x readelf

0000000000400000      4K r-x-- memtest               可执行文件
0000000000600000      4K r---- memtest               只读数据.rodata
0000000000601000      4K rw--- memtest               全局数据
00007f6f29e62000   1752K r-x-- libc-2.17.so
00007f6f2a018000   2048K ----- libc-2.17.so
00007f6f2a218000     16K r---- libc-2.17.so
00007f6f2a21c000      8K rw--- libc-2.17.so
00007f6f2a21e000     20K rw---   [ anon ]
00007f6f2a223000    128K r-x-- ld-2.17.so
00007f6f2a431000     12K rw---   [ anon ]
00007f6f2a43f000     12K rw---   [ anon ]
00007f6f2a442000      4K r---- ld-2.17.so
00007f6f2a443000      4K rw--- ld-2.17.so
00007f6f2a444000      4K rw---   [ anon ]
00007ffc38e32000    132K rw---   [ stack ]
  0x7ffc38e51504
00007ffc38f74000      8K r-x--   [ anon ]
ffffffffff600000      4K r-x--   [ anon ]
 total             4164K

内存申请、释放会立即在writeable/private中显示。

pmap -x $(pidof memtest)
pmap -x $(pidof uagent)
top -p $(pidof uagent)

两次申请的内存是否相同？




/proc/PID/{maps,smaps,statm} 中的内容，与 pmap 打印的信息比较相似，接下来看看 Mode 中的信息是什么意思？哪些标示了真正的内存？

首先与 PID 相关的 proc 系统在 fs/proc/base.c 文件中实现，详见 tid_base_stuff 中的入口函数定义。

#### statm

对应了 fs/proc/array.c 中的 proc_pid_statm() 函数。
SHR shared
RES resident
VIRT mm->total_vm
CODE code
DATA data

#### maps




#### smaps

01cbe000-01da9000 r--p 016be000 ca:01 68320258   /usr/sbin/mysqld
Size:                940 kB
Rss:                 236 kB
Pss:                 236 kB
Shared_Clean:          0 kB
Shared_Dirty:          0 kB
Private_Clean:         0 kB
Private_Dirty:       236 kB
Referenced:           76 kB
Anonymous:           236 kB
AnonHugePages:         0 kB
Swap:                  0 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Locked:                0 kB
VmFlags: rd mr mw me dw ac

上述的输出对应 fs/proc/task_mmu.c 文件中的 show_smap() 函数，该函数

show_smap()
 |-walk_page_range()

Linux 内核中，struct vm_area_struct (include/linux/mm_types.h) 是关于虚存管理的最基本单元，描述了一段连续的、具有相同访问属性的虚存空间，该虚存空间的大小为物理内存页面的整数倍，范围是 [vm_start, vm_end) 。


struct vm_area_struct {
  unsigned long vm_start;
  unsigned long vm_end;
  struct vm_area_struct *vm_next, *vm_prev;
  struct rb_node vm_rb;

由于进程所使用到的虚存空间不连续，且访问属性也不同，所以一般一个进程的虚存空间需要多个 vm_area_struct 描述，数目少时通过双向链表组织，否则通过红黑树。


　　假如该vm_area_struct描述的是一个文件映射的虚存空间，成员vm_file便指向被映射的文件的file结构，vm_pgoff是该虚存空间起始地址在vm_file文件里面的文件偏移，单位为物理页面。

　　一个程序可以选择MAP_SHARED或MAP_PRIVATE共享模式将一个文件的某部分数据映射到自己的虚存空间里面。这两种映射方式的区别在于：MAP_SHARED映射后在内存中对该虚存空间的数据进行修改会影响到其他以同样方式映射该部分数据的进程，并且该修改还会被写回文件里面去，也就是这些进程实际上是在共用这些数据。而MAP_PRIVATE映射后对该虚存空间的数据进行修改不会影响到其他进程，也不会被写入文件中。

　　来自不同进程，所有映射同一个文件的vm_area_struct结构都会根据其共享模式分别组织成两个链表。链表的链头分别是：vm_file->f_dentry->d_inode->i_mapping->i_mmap_shared,vm_file->f_dentry->d_inode->i_mapping->i_mmap。而vm_area_struct结构中的vm_next_share指向链表中的下一个节点；vm_pprev_share是一个指针的指针，它的值是链表中上一个节点（头节点）结构的vm_next_share（i_mmap_shared或i_mmap）的地址。

　　进程建立vm_area_struct结构后，只是说明进程可以访问这个虚存空间，但有可能还没有分配相应的物理页面并建立好页面映射。在这种情况下，若是进程执行中有指令需要访问该虚存空间中的内存，便会产生一次缺页异常。这时候，就需要通过vm_area_struct结构里面的vm_ops->nopage所指向的函数来将产生缺页异常的地址对应的文件数据读取出来。

　　vm_flags主要保存了进程对该虚存空间的访问权限，然后还有一些其他的属性。vm_page_prot是新映射的物理页面的页表项pgprot的默认值。



flag 对应了 struct vm_area_struct (include/linux/mm_types.h) 中的 vm_flags ，例如 VM_READ、VM_WRITE、VM_EXEC 等。

关于mm_struct和vm_area_struct的关系可以参考
http://csapp.cs.cmu.edu/3e/ics3/vm/linuxvm.pdf

### 内核中常见操作

1. 给定一个属于某个进程的虚拟地址，要求找到其所属的区间以及 vma_area_struct 结构，该功能是由 find_vma()@mm/mmap.c 中实现。

进程加载过程
http://www.cnblogs.com/web21/p/6222547.html


Linux 中每个页通过 page frame number, PFN 唯一定义。

Linux内存的通解
http://www.tldp.org/LDP/tlk/mm/memory.html



#### statm

$ pmap -x $(pidof mysqld) && ps aux | grep mysqld

-x 输出值: 显示扩展格式
  Address : 虚拟内存开始地址；
  Kbytes  : 占用内存的字节数，单位是KB；
  RSS     : 保留内存的字节数（KB）
  Dirty: 脏页的字节数（包括共享和私有的）（KB）
  Mode: 内存的权限，包括了read、write、execute、shared、private；
  Mapping : 占用内存的文件、[anon] (分配的内存)、[stack] (堆栈)；

$ pmap -d $(pidof mysqld)
Address           Kbytes Mode  Offset           Device    Mapping
0000000000400000   23288 r-x-- 0000000000000000 0ca:00001 mysqld
0000000001cbe000     940 r---- 00000000016be000 0ca:00001 mysqld
0000000001da9000     680 rw--- 00000000017a9000 0ca:00001 mysqld
0000000001e53000     888 rw--- 0000000000000000 000:00000   [ anon ]
0000000001f31000   11680 rw--- 0000000000000000 000:00000   [ anon ]
00007fd784000000    4248 rw--- 0000000000000000 000:00000   [ anon ]
00007fd784426000   61288 ----- 0000000000000000 000:00000   [ anon ]
00007fd78c000000     132 rw--- 0000000000000000 000:00000   [ anon ]
00007fd78c021000   65404 ----- 0000000000000000 000:00000   [ anon ]
00007fd7d2c67000      12 rw-s- 0000000000000000 000:0000a [aio] (deleted)
00007fd7d2c6a000      12 rw-s- 0000000000000000 000:0000a [aio] (deleted)
00007fd7d2c6d000       4 rw-s- 0000000000000000 000:0000a [aio] (deleted)
00007fd7d2c6e000       4 rw--- 0000000000000000 000:00000   [ anon ]
00007fd7d2c6f000       4 r---- 000000000001f000 0ca:00001 ld-2.17.so
00007fd7d2c70000       4 rw--- 0000000000020000 0ca:00001 ld-2.17.so
00007fd7d2c71000       4 rw--- 0000000000000000 000:00000   [ anon ]
00007ffc8fef3000     132 rw--- 0000000000000000 000:00000   [ stack ]
mapped: 1248492K    writeable/private: 545308K    shared: 128K

-d 输出值: 显示设备格式
  Offset: 文件偏移
  Device: 设备名 (major:minor)
最后一行输出:
  mapped    : 该进程映射的虚拟地址空间大小，也就是该进程预先分配的虚拟内存大小，即ps出的vsz；
  writeable/private : 进程所占用的私有地址空间大小，也就是该进程实际使用的内存大小，不含shared libraries；
  shared : 进程和其他进程共享的内存大小；
可以通过如下命令计算，假设导出到/tmp/1文件中
  mapped: 所有行Kbytes字段的累加
    awk 'BEGIN{sum=0};{sum+=$2};END{print sum}' /tmp/1
  writeable/private: 模式为w+p，且不是s的内存
 awk 'BEGIN{sum=0};{if($3~/w/ && $3!~/s/) {sum+=$2}};END{print sum}' /tmp/1
  shared: 共享内存数
    awk 'BEGIN{sum=0};{if($3~/s/) {sum+=$2}};END{print sum}' /tmp/1

共享内存的库实际包括了两部分，分别是代码 ```r----``` 以及数据 ```rw---``` 。

如果加载的插件比较多，那么writeable/private非常大126M，而 top %MEM 只有 0.1M ？？？？

man 1 top
%MEM -- Memory usage (RES)
   A task's currently used share of available physical memory.

RES  --  Resident Memory Size (KiB)
   The non-swapped physical memory a task is using.



    awk 'BEGIN{sum=0};{if($3="-----") {sum+=$2}};END{print sum}' /tmp/1

writeable/private 真正含义是什么，为什么大于 RSS ，而 -x 统计的 RSS 基本相同



查看那个进程在使用SWAP
https://www.cyberciti.biz/faq/linux-which-process-is-using-swap/

虚拟内存到物理内存的映射
https://github.com/dwks/pagemap


## 按列求和

可以使用 awk 命令计算文件中某一列的总和。

----- 对第二列求和
$ awk 'BEGIN{sum=0}{sum+=$2}END{print sum}' data.txt
----- 对满足开始为/foobar/的行求和
$ awk 'BEGIN{sum=0};/foobar/{sum+=$2};END{print sum}' data.txt
----- 另外比较完整的一个例子
$ awk -F ',' 'BEGIN{sum=0;count=0}{if ($(NF-11)==2 && $NF==0 && $3=="11" && $6~/TIME|ESTABLISHED/) \
 {sum +=$5; count++;} } END {print "sum="sum" count="count " avg="sum/count}'

$N 表示第 N 列，从 0 开始计算；$0 表示所有列；NF 表示 Number of Fields，也就是字段数，$NF 表示最后一个字段，$(NF-1) 表示倒数第二个字段。


awk 'BEGIN{sum=0};/foobar/{sum+=$1};END{print sum}' data.txt


awk '{print $2}' /tmp/1

关于内存极其推荐的两篇文章 [What every programmer should know about memory](https://www.akkadia.org/drepper/cpumemory.pdf) 以及 [Memory Part1-Part6](https://techtalk.intersec.com/2013/07/memory-part-1-memory-types/)


top + g3 查看内存使用情况，包括了 ```%MEM, VIRT, RES, CODE, DATA, SHR, nMaj, nDRT``` 列，这些信息都是从 /proc/PID/statm 文件中读取的。

size (mapped to VIRT), resident (mapped to RES), share (mapped to SHR), text (mapped to CODE), lib (always 0 on Linux 2.6+), data (mapped to DATA) and dt (always 0 on Linux 2.6+, mapped to nDrt).

http://jzhihui.iteye.com/blog/1447570
http://spockwangs.github.io/2011/08/20/loading-running-and-termination-of-linux-program.html
http://www.cnblogs.com/vampirem/archive/2013/05/30/3108973.html
http://time-track.cn/get-libs-the-process-use.html
http://www.penglixun.com/tech/system/the_diffrents_of_page_cache_and_buffer_cache.html
http://www.cnblogs.com/emperor_zark/archive/2013/03/15/linux_page_1.html
http://blog.csdn.net/mihouge/article/details/6936099
https://techtalk.intersec.com/2013/07/memory-part-1-memory-types/
https://www.thomas-krenn.com/en/wiki/Linux_Page_Cache_Basics
http://www.cnblogs.com/bravery/archive/2012/06/27/2560611.html
http://www.360doc.com/content/12/0925/21/1072296_238157352.shtml
http://lzz5235.github.io/2014/12/10/toolsvmpage-typesc.html
http://www.greenend.org.uk/rjk/tech/dataseg.html
http://fivelinesofcode.blogspot.com/2014/03/how-to-translate-virtual-to-physical.html
http://blog.jeffli.me/blog/2014/11/08/pagemap-interface-of-linux-explained/


### buddyinfo

是 Linux Buddy 系统管理物理内存的 debug 信息，为了解决物理内存的碎片问题，其把所有空闲的内存，以 2 的幂次方的形式，分成 11 个块链表，分别对应为 1、2、4、8、16、32、... ... 1024 个页块。

Linux 支持 NUMA 技术，NUMA 系统的结点通常是由一组 CPU 和本地内存组成，每个节点都有相应的本地内存，z在 buddyinfo 中通过 Node N 表示一个 NUMA 节点，如果硬件不支持 NUMA，则只有 Node 0。

而每个节点下的内存设备，又可以划分为多个内存区域 (zone)，因此下面的显示中，对于 Node 0 的内存，又划分类 DMA、DMA32、Normal，部分还有 HighMem 区域。

而每行的数字表示上述 11 个链表连续空闲的区域的大小，例如 Normal 中的第二列表示连续两个内存的大小为 ```3506*2*PAGE_SIZE```。

{% highlight text %}
# cat /proc/buddyinfo
Node 0, zone      DMA      2      2      2      1      3      2      0      0      1      1      3
Node 0, zone    DMA32    118    156    996    790    254     88     47     13      0      0      0
Node 0, zone   Normal   2447   3506    613     31      0      0      0      0      0      0      0
{% endhighlight %}


kpagecount
kpageflags
pagetypeinfo ?
slabinfo
swaps
vmallocinfo
vmstat
zoneinfo


## 常见故障处理


### dentry cache

free 命令主要显示的用户的内存使用 (新版available可用)，包括使用 top 命令 (Shift+M内存排序)，对于 slab 实际上没有统计 (可以使用 slabtop 查看)，真实环境中，很多出现与 dentry(slab) 的内存消耗过大的情况。

dentry cache 是目录项高速缓存，记录了目录项到 inode 的映射关系，用于提高目录项对象的处理效率，当通过 stat() 查看文件信息、open() 打开文件时，都会创建 dentry cache 。

{% highlight text %}
----- 查看当前进程的系统调用
# strace -f -e trace=open,stat,close,unlink -p $(ps aux | grep 'cmd' | awk '{print $2}')
{% endhighlight %}

不过这里有个比较奇怪的现象，从 /proc/meminfo 可以看出，slab 内存分为两部分：A) SReclaimable；B) SUnreclaim；而占比比较大的是 SReclaimable 的内存，也就是可以回收的内存，而通过 slabtop 命令可以看到 dentry 对应的基本 100% 处于使用状态。

首先，/proc/meminfo 对应到源码中为 fs/proc/meminfo.c，在内核中与这两个状态相关的代码在 mm/slub.c 中，简单来说就是对应到了 allocate_slab() 和 __free_slab() 函数中，直接搜索 NR_SLAB_RECLAIMABLE 即可。对应到代码中，只要申请的 cache 使用了 SLAB_RECLAIM_ACCOUNT 标示，那么就统计为 SReclaimable 。

而从 /proc/slabinfo 中读取对应源码 mm/slab_common.c 中的 cache_show() 函数。

也就是说 meminfo 中的 SReclaimable 标识的是可以回收的 cache，但是真正的回收操作是由各个申请模块控制的，对于 dentry 来说，真正的回收操作还是 dentry 模块自己负责。

### 解决方法

Linux 2.6.16 之后提供了 [drop caches](https://linux-mm.org/Drop_Caches) 机制，用于释放 page cache、inode/dentry caches ，方法如下。

{% highlight text %}
----- 刷新文件系统缓存
# sync
----- 释放Page Cache
# echo 1 > /proc/sys/vm/drop_caches
----- 释放dentries+inodes
# echo 2 > /proc/sys/vm/drop_caches
----- 释放pagecache, dentries, inodes
# echo 3 > /proc/sys/vm/drop_caches
----- 如果没有root权限，但是有sudo权限
$ sudo sysctl -w vm.drop_caches=3
{% endhighlight %}

注意，修改 drop_caches 的操作是无损的，不会释放脏页，所以需要在执行前执行 sync 以确保确实有可以释放的内存。

另外，需要通过修改 /proc/sys/vm/vfs_cache_pressure 调整清理 inode/dentry caches 的优先级，默认为 100 此时需要调大；关于该文件，在 LinuxInsight 中的解释如下：

At the default value of vfs_cache_pressure = 100 the kernel will attempt to reclaim dentries and inodes at a “fair” rate with respect to pagecache and swapcache reclaim. Decreasing vfs_cache_pressure causes the kernel to prefer to retain dentry and inode caches. Increasing vfs_cache_pressure beyond 100 causes the kernel to prefer to reclaim dentries and inodes.

https://major.io/2008/12/03/reducing-inode-and-dentry-caches-to-keep-oom-killer-at-bay/



系统启动时，BIOS 会自动发现当前物理内存的地址，而 Linux 内核会在启动时 (还在实地址模式) 通过 e820 BIOS 系统调用获取当前系统的物理内存地址，当然还有 IO 的映射地址等；开始内核会初始化部分内存供内核使用，然后调用 Bootmem 系统。

Linux 将内存分为不同的 Zone 管理，每个 Zone 中通过一个 buddy 系统分配内存，每次分配内存都是以页的 2 指数倍分配，例如内核页大小为 4K(getconf PAGE_SIZE)，那么分配的页为 4K, 8K, 16K, ..., 128K，一般系统最大为 128K 。

Buddy Allocator 最大的问题就是碎片，所以一般系统不会直接使用，通常其上层还包括了 Page Cache、Slab Allocator 。


### 自动回收 dcache



查了一些关于Linux dcache的相关资料，发现操作系统会在到了内存临界阈值后，触发kswapd内核进程工作才进行释放，这个阈值的计算方法如下：

1. 首先，grep low /proc/zoneinfo，得到如下结果：

        low      1
        low      380
        low      12067

2. 将以上3列加起来，乘以4KB，就是这个阈值，通过这个方法计算后发现当前服务器的回收阈值只有48MB，因此很难看到这一现象，实际中可能等不到回收，操作系统就会hang住没响应了。

3. 可以通过以下方法调大这个阈值：将vm.extra_free_kbytes设置为vm.min_free_kbytes和一样大，则/proc/zoneinfo中对应的low阈值就会增大一倍，同时high阈值也会随之增长，以此类推。

$ sudo sysctl -a | grep free_kbytes
vm.min_free_kbytes = 39847
vm.extra_free_kbytes = 0
$ sudo sysctl -w vm.extra_free_kbytes=836787  ######1GB


 4. 举个例子，当low阈值被设置为1GB的时候，当系统free的内存小于1GB时，观察到kswapd进程开始工作（进程状态从Sleeping变为Running），同时dcache开始被系统回收，直到系统free的内存介于low阈值和high阈值之间，停止回收。

vm.overcommit_ratio=2
vm.dirty_background_ratio=5
vm.dirty_ratio=20
min_free_kbytes
extra_free_kbytes

https://huoding.com/2015/06/10/444
http://www.cnblogs.com/panfeng412/p/drop-caches-under-linux-system-2.html

https://www.halobates.de/memory.pdf

https://bhsc881114.github.io/2015/04/19/%E4%B8%80%E6%AC%A1linux%E5%86%85%E5%AD%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5-slab/
http://linuxperf.com/?p=148
-->

{% highlight text %}
{% endhighlight %}
