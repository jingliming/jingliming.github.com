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
{% endhighlight %}


<!--
各个段的含义为：
	<ol type='A'><li>
		procs 进程<br>
		<ul><li>
			r: 等待运行的进程数，如果运行队列过大，表示CPU很忙，一般会造成CPU使用率很高。</li><li>
			b: 不可中断睡眠(uninterruptible sleep)的进程数，也就是被阻塞的进程，通常为IO。
		</li></ul></li><br><li>

		Memory 内存<br>
		<ul><li>
			swpd: 虚拟内存(Virtual Memory)的使用量，应该是交换区。</li><li>
			free: 空闲内存数，同free命令中的free(Mem)。</li><li>
			buff: 同free命令，已分配未使用的内存大小。</li><li>
			cache: 同free命令，已分配未使用的内存大小。</li><li>
			inact: 非活跃(inactive)内存的大小，使用 -a 选项????。</li><li>
			active: 活跃(active)内存的大小，使用 -a 选项????。
		</li></ul></li><br><li>

		Swap 交换区<br>
		<ul><li>
			si: 每秒从交换区(磁盘)写到内存的大小。</li><li>
            so: 每秒写入交换区(磁盘)的大小。</li></ul>
        如果这两个值长期大于0，系统性能会受到影响，即我们平常所说的Thrashing(颠簸)；如果虽然free很少，但是si和so也很少（大多时候是0），也不用担心，系统性能这时不会受到影响的。
        </li><br><li>

		IO<br>
		<ul><li>
			bi: 每秒读取的块数(blocks/s)。</li><li>
			bo: 每秒写入的块数(blocks/s)。
		</li></ul></li><br><li>

		System 系统<br>
		<ul><li>
			in: 每秒的中断数，包括时钟中断。</li><li>
			cs: 每秒上线文的交换数。
		</li></ul></li><br><li>

		CPU<br>
		<ul><li>
			us: 用户进程执行时间。</li><li>
			sy: 系统进程执行时间。</li><li>
			id: 空闲进程执行时间，2.5.41 之后含有 IO-wait 的时间。</li><li>
			wa: 等待 IO 的时间，2.5.41 之后含有空闲任务。
		</li></ul>
	</li></ol>

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
</P>



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

可以根据进程查看进程相关信息占用的内存情况，实际是整个内存的映射关系。

{% highlight text %}
$ pmap -d 14596
{% endhighlight %}

## 小结

可以通过 free 查看整个系统内存的使用情况，free(Mem) 过小不会对系统的性能产生影响，实际剩余的内存数为 free(-/+ buffers/cache)。

通过 vmstat 1 3 输出的 swap(si/so) 可以查看交换区的切入/出情况，如果长期不为 0，则可能是因为内存不足，此时应该查看那个进程占用内存较多。


## 其它

### Cache 和 Buffer 的区别

简单说，Buffer 的核心作用是用来缓冲，缓和冲击。比如你每秒要写 100 次硬盘，对系统冲击很大，浪费了大量时间在忙着处理开始写和结束写这两件事。用个 buffer 暂存起来，变成每 10 秒写一次硬盘，对系统的冲击就很小，写入效率高了，极大缓和了冲击。

Cache 的核心作用是加快取用的速度。比如你一个很复杂的计算做完了，下次还要用结果，就把结果放手边一个好拿的地方存着，下次不用再算了，加快了数据取用的速度。

所以，如果你注意关心过存储系统的话，你会发现硬盘的读写缓冲/缓存名称是不一样的，叫 write-buffer 和 read-cache；很明显地说出了两者的区别。


<!--
当然很多时候宏观上说两者可能是混用的。比如实际上memcached很多人就是拿来读写都用的。不少时候Non-SQL数据库也是。严格来说，CPU里的L2和L3 Cache也都是读写兼用——因为你没法简单地定义CPU用它们的方法是读还是写。硬盘里也是个典型例子，buffer和cache都在一块空间上，到底是buffer还是cache？

不过仔细想一下，你说拿cache做buffer用行不行？当然行，只要能控制cache淘汰逻辑就没有任何问题。那么拿buffer做cache用呢？貌似在很特殊的情况下，能确定访问顺序的时候，也是可以的。简单想一下就明白——buffer根据定义，需要随机存储吗？一般是不需要的。但cache一定要。所以大多数时候用cache代替buffer可以，反之就比较局限。这也是技术上说cache和buffer的关键区别。

https://www.zhihu.com/question/26190832/answer/146259979
-->


## 参考

查看那个进程在使用交换空间可以参考 [Find Out What Process Are Using Swap Space](http://www.cyberciti.biz/faq/linux-which-process-is-using-swap/)，或者 [本地文档](/reference/linux/monitor/Find Out What Process Are Using Swap Space.maff) 。

通过 [vmtouch offical site](http://hoytech.com/vmtouch/) 可以查看文件在内存中的占比，可以下载 [本地源码](/reference/linux/monitor/vmtouch.c) 。

<!--memtester 内存测试工具。-->

<!--
可用内存计算方法
http://www.cnblogs.com/feisky/archive/2012/04/14/2447503.html
内存监控相关
https://linux-audit.com/understanding-memory-information-on-linux-systems/
一次高内存使用率的告警处理
http://farll.com/2016/10/high-memory-usage-alarm/
-->


{% highlight text %}
{% endhighlight %}
