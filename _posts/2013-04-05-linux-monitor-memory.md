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


# 参考

memtester 内存测试工具。



-->



{% highlight text %}
{% endhighlight %}
