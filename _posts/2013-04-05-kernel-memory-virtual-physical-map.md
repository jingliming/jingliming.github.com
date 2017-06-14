---
title: Kernel 内存映射
layout: post
comments: true
language: chinese
category: [linux]
keywords: linux,内存,memory,映射
description:
---


<!-- more -->

内存管理的最小单元是 Page ，这也就意味着在一页内的线性地址和物理地址是连续的。另外，需要注意 page 与 page frame 的区别，后者是物理内存的分割单元；前者包含数据，可能保存在 page frame 中，也可能保存在磁盘上。

线性地址到物理地址的映射是通过 page table 完成的，内核会在启动分页机制前完成初始化；而且内核会将 A) 不可用物理地址空间，B) 内核代码以及内核初始数据结构对应的地址空间保留。

## 用户虚拟地址到物理地址转换

查看当前进程的虚拟地址映射 ```cat /proc/self/maps```，

通过命令你给 ```ps aux | less``` 查看内存相关参数时，会发现两个比较有疑问的指标 RSS、VSZ (单位是KB)，通过 ```man 1 ps``` 查看对应的解释如下：

 * RSS resident set size, the non-swapped physical memory that a task has used (in kiloBytes). (alias rssize, rsz).
 * VSZ virtual memory size of the process in KiB (1024-byte units). Device mappings are currently excluded; this is subject to change. (alias vsize).

简言之，RSS 就是这个进程实际占用的物理内存；VSZ 就是进程的虚拟内存，包括了还未发生缺页异常加载映射到内存中。如果通过上述的两个指标来评估实际内存使用量，那么是错误的 ！！！

ps 显示的统计结果实际上有个前提假设：如果只有这一个进程在运行。在现在的内核中，显然是不可能的，很多进程会共享一些内存，例如 libc 。




## /proc/meminfo

该文件的输出实现在 ```fs/proc/meminfo.c``` 文件中。

{% highlight text %}
MemTotal:        8070604 kB
MemFree:          842800 kB   当前未使用的内存，注意，这里不包括内核中可回收内存，
                              包括了HighFree+LowFree，需要内核配置CONFIG_HIGHMEM选项
MemAvailable:    3720728 kB   在不发生swap时的最大可用内存，真正可用物理内存


Shmem:            962032 kB   一般是tmpfs使用，如/dev/shm,/run等，另外还有内核中的SysV


Buffers:          104944 kB
Cached:          3734768 kB
SwapCached:         1028 kB
Active:          4575172 kB
Inactive:        2156288 kB
Active(anon):    2779724 kB
Inactive(anon):  1074056 kB
Active(file):    1795448 kB
Inactive(file):  1082232 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       8127484 kB
SwapFree:        8101540 kB
Dirty:              2156 kB
Writeback:             0 kB
AnonPages:       2891148 kB
Mapped:           554884 kB
Shmem:            962032 kB
Slab:             354444 kB
SReclaimable:     304396 kB
SUnreclaim:        50048 kB
KernelStack:        8144 kB
PageTables:        41536 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    12162784 kB
Committed_AS:    6948452 kB
VmallocTotal:   34359738367 kB
VmallocUsed:      370112 kB
VmallocChunk:   34358947836 kB
HardwareCorrupted:     0 kB
AnonHugePages:    839680 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      278372 kB
DirectMap2M:     5908480 kB
DirectMap1G:     2097152 kB
{% endhighlight %}

在计算可用内存时，最早一般使用 free+cached，而实际上 cached 中包含了


> 简单介绍下 CONFIG_HIGHMEM 。
>
> 部分 CPU (如ARM) 只能映射 4G 的内存管理空间，这 4G 空间包括了用户空间、内核空间、IO 空间，如果物理内存大于 4G ，那么必定有部分内存在这种情况下是无法管理的，这部分内存也就被称为 "high memory" 。
>
> 简单来说，之所以有 high memory 是因为物理内存超过了虚拟内存，导致内核无法一次映射所有物理内存，为此就需要有临时的映射。注意，创建临时映射的成本很高，需要修改内核的 page table 以及 TLB/MMU 。
>
>  详细可以查看 [Kenel-doc HIGH MEMORY HANDLING](https://www.kernel.org/doc/Documentation/vm/highmem.txt) 文档。

### Active VS. Inactive

除了通过 ```/proc/meminfo``` 查看外，还可以通过 ```vmstat -a``` 命令查看，与之相关的代码如下：

{% highlight c %}
static int meminfo_proc_show(struct seq_file *m, void *v)
{
    ... ...
    for (lru = LRU_BASE; lru < NR_LRU_LISTS; lru++)
        pages[lru] = global_page_state(NR_LRU_BASE + lru);

    ... ...
    seq_printf(m,
        "Active:         %8lu kB\n"
        "Inactive:       %8lu kB\n"
        "Active(anon):   %8lu kB\n"
        "Inactive(anon): %8lu kB\n"
        "Active(file):   %8lu kB\n"
        "Inactive(file): %8lu kB\n"
        ... ...
        K(pages[LRU_ACTIVE_ANON]   + pages[LRU_ACTIVE_FILE]),
        K(pages[LRU_INACTIVE_ANON] + pages[LRU_INACTIVE_FILE]),
        K(pages[LRU_ACTIVE_ANON]),
        K(pages[LRU_INACTIVE_ANON]),
        K(pages[LRU_ACTIVE_FILE]),
        K(pages[LRU_INACTIVE_FILE]),
        ... ...
}
{% endhighlight %}

为了实现 LRU 功能，正常应该有字段记录最近访问时间，可惜 x86 CPU 硬件并不支持这个特性，只能做到在访问页面时设置一个 Access Bit 标志位，无法记录时间。

所以 Linux 使用了一个折衷的方法，采用 LRU list 列表，把刚访问过的页面放在列首，越接近列尾的就是越长时间未访问过的页面，这样，虽然不能记录访问时间，但利用页面在 LRU list 中的相对位置也可以轻松找到年龄最长的页面。

内核设计了两种 LRU list: active list 和 inactive list，刚访问过的页面放进 active list，长时间未访问过的页面放进 inactive list，这样从 inactive list 回收页面就变得简单了。

内核线程 kswapd 会周期性地把 active list 中符合条件的页面移到 inactive list 中。 <!-- ，这项转移工作是由 refill_inactive_zone() 完成的。 -->

![active inactive list]({{ site.url }}/images/linux/kernel/memory-active-inactive-list.png "active inactive list"){: .pull-center width="80%" }

如上，如果 inactive list 很大，表明在必要时可以回收的页面很多，反之，则说明可以回收的页面不多。另外，这里的内存是用户进程所占用的内存而言的，内核占用的内存 (包括slab) 不在其中。

<!--
至于在源代码中看到的ACTIVE_ANON和ACTIVE_FILE，分别表示anonymous pages和file-backed pages。用户进程的内存页分为两种：与文件关联的内存（比如程序文件、数据文件所对应的内存页）和与文件无关的内存（比如进程的堆栈，用malloc申请的内存），前者称为file-backed pages，后者称为anonymous pages。File-backed pages在发生换页(page-in或page-out)时，是从它对应的文件读入或写出；anonymous pages在发生换页时，是对交换区进行读/写操作。
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

<!--
可用内存计算方法
http://www.cnblogs.com/feisky/archive/2012/04/14/2447503.html

一次高内存使用率的告警处理

http://farll.com/2016/10/high-memory-usage-alarm/

http://marek.vavrusa.com/c/memory/2015/02/20/memory/

http://careers.directi.com/display/tu/Understanding+and+optimizing+Memory+utilization



Documentation/filesystems/proc.txt

http://www.linuxatemyram.com/

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





dentry cache 和 inode cache 区别。



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


kpagecount
kpageflags
pagetypeinfo ?
slabinfo
swaps
vmallocinfo
vmstat
zoneinfo

-->

## 常见故障处理

### dentry cache

free 命令主要显示的用户的内存使用 (新版available可用)，包括使用 top 命令 (Shift+M 内存排序)，对于 slab 实际上没有统计 (可以使用 slabtop 查看)，真实环境中，很多出现与 dentry(slab) 的内存消耗过大的情况。

dentry cache 是目录项高速缓存，记录了目录项到 inode 的映射关系，用于提高目录项对象的处理效率，当通过 stat() 查看文件信息、open() 打开文件时，都会创建 dentry cache 。

{% highlight text %}
----- 查看当前进程的系统调用
# strace -f -e trace=open,stat,close,unlink -p $(ps aux | grep 'cmd' | awk '{print $2}')
{% endhighlight %}

不过这里有个比较奇怪的现象，从 ```/proc/meminfo``` 可以看出，slab 内存分为 SReclaimable 和 SUnreclaim 两部分；而占比比较大的是 SReclaimable 的内存，也就是可以回收的内存，而通过 slabtop 命令可以看到 dentry 对应的基本 100% 处于使用状态。

那么，到底是什么意思？这部分内存能不能回收？

首先，```/proc/meminfo``` 对应到源码中为 ```fs/proc/meminfo.c```，在内核中与这两个状态相关的代码在 ```mm/slub.c``` 中 (如果是新版本的内核)；简单来说就是对应到了 ```allocate_slab()``` 和 ```__free_slab()``` 函数中，实际上源码中直接搜索 ```NR_SLAB_RECLAIMABLE``` 即可。

对应到源码码中，实际只要申请 cache 时使用了 ```SLAB_RECLAIM_ACCOUNT``` 标示，那么在进行统计时，就会被标记为 SReclaimable 。

<!-- 而从 /proc/slabinfo 中读取对应源码 mm/slab_common.c 中的 cache_show() 函数。 -->

也就是说 meminfo 中的 SReclaimable 标识的是可以回收的 cache，但是真正的回收操作是由各个申请模块控制的，对于 dentry 来说，真正的回收操作还是 dentry 模块自己负责。

### 解决方法

对于上述的 slab 内存占用过多的场景，在 Linux 2.6.16 之后提供了 [drop caches](https://linux-mm.org/Drop_Caches) 机制，用于释放 page cache、inode/dentry caches ，方法如下。

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

另外，除了上述的手动处理方式之外，还可以通过修改 ```/proc/sys/vm/vfs_cache_pressure``` 的参数，来调整自动清理 inode/dentry caches 的优先级，默认为 100 此时需要调大；关于该参数可以参考内核文档 [vm.txt]( {{ site.kernel_docs_url }}/Documentation/sysctl/vm.txt )，简单摘抄如下：

{% highlight text %}
This percentage value controls the tendency of the kernel to reclaim
the memory which is used for caching of directory and inode objects.

At the default value of vfs_cache_pressure=100 the kernel will attempt to
reclaim dentries and inodes at a "fair" rate with respect to pagecache and
swapcache reclaim.  Decreasing vfs_cache_pressure causes the kernel to prefer
to retain dentry and inode caches. When vfs_cache_pressure=0, the kernel will
never reclaim dentries and inodes due to memory pressure and this can easily
lead to out-of-memory conditions. Increasing vfs_cache_pressure beyond 100
causes the kernel to prefer to reclaim dentries and inodes.

Increasing vfs_cache_pressure significantly beyond 100 may have negative
performance impact. Reclaim code needs to take various locks to find freeable
directory and inode objects. With vfs_cache_pressure=1000, it will look for
ten times more freeable objects than there are.
{% endhighlight %}

也就是说，当该文件对应的值越大时，系统的回收操作的优先级也就越高，不过内核对该值好像不敏感，一般会设置为 10000 。

<!-- https://major.io/2008/12/03/reducing-inode-and-dentry-caches-to-keep-oom-killer-at-bay/ -->

<!--
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
extra_free_kbytes
-->



接下来，看看如何保留部分内存。

### min_free_kbytes

该参数在内核文档 [vm.txt]( {{ site.kernel_docs_url }}/Documentation/sysctl/vm.txt )，简单摘抄如下：

{% highlight text %}
This is used to force the Linux VM to keep a minimum number
of kilobytes free.  The VM uses this number to compute a
watermark[WMARK_MIN] value for each lowmem zone in the system.
Each lowmem zone gets a number of reserved free pages based
proportionally on its size.

Some minimal amount of memory is needed to satisfy PF_MEMALLOC
allocations; if you set this to lower than 1024KB, your system will
become subtly broken, and prone to deadlock under high loads.

Setting this too high will OOM your machine instantly.
{% endhighlight %}


该参数值在 init_per_zone_wmark_min() 函数中设置，


<!--
1. 代表系统所保留空闲内存的最低限。
在系统初始化时会根据内存大小计算一个默认值，计算规则是：

  min_free_kbytes = sqrt(lowmem_kbytes * 16) = 4 * sqrt(lowmem_kbytes)(注：lowmem_kbytes即可认为是系统内存大小）

另外，计算出来的值有最小最大限制，最小为128K，最大为64M。
可以看出，min_free_kbytes随着内存的增大不是线性增长，comments里提到了原因“because network bandwidth does not increase linearly with machine size”。随着内存的增大，没有必要也线性的预留出过多的内存，能保证紧急时刻的使用量便足矣。
-->

该参数的主要用途是计算影响内存回收的三个参数 ```watermark[min/low/high]```，每个 Zone 都保存了各自的参数值，不同的范围操作如下。

{% highlight text %}
< watermark[low]
  触发内核线程kswapd回收内存，直到该Zone的空闲内存数达到watermark[high]；
< watermark[min]
  进行direct reclaim (直接回收)，即直接在应用程序的进程上下文中进行回收，再用
  回收上来的空闲页满足内存申请，因此实际会阻塞应用程序，带来一定的响应延迟，
  而且可能会触发系统OOM。
{% endhighlight %}

实际上，可以认为 ```watermark[min]``` 以下的内存属于系统的自留内存，用以满足特殊使用，所以不会给用户态的普通申请来用。


<!--
3）三个watermark的计算方法：

 watermark[min] = min_free_kbytes换算为page单位即可，假设为min_free_pages。（因为是每个zone各有一套watermark参数，实际计算效果是根据各个zone大小所占内存总大小的比例，而算出来的per zone min_free_pages）

 watermark[low] = watermark[min] * 5 / 4
 watermark[high] = watermark[min] * 3 / 2

所以中间的buffer量为 high - low = low - min = per_zone_min_free_pages * 1/4。因为min_free_kbytes = 4* sqrt(lowmem_kbytes），也可以看出中间的buffer量也是跟内存的增长速度成开方关系。
4）可以通过/proc/zoneinfo查看每个zone的watermark
例如：

Node 0, zone      DMA
pages free     3960
       min      65
       low      81
       high     97


3.min_free_kbytes大小的影响
min_free_kbytes设的越大，watermark的线越高，同时三个线之间的buffer量也相应会增加。这意味着会较早的启动kswapd进行回收，且会回收上来较多的内存（直至watermark[high]才会停止），这会使得系统预留过多的空闲内存，从而在一定程度上降低了应用程序可使用的内存量。极端情况下设置min_free_kbytes接近内存大小时，留给应用程序的内存就会太少而可能会频繁地导致OOM的发生。
min_free_kbytes设的过小，则会导致系统预留内存过小。kswapd回收的过程中也会有少量的内存分配行为（会设上PF_MEMALLOC）标志，这个标志会允许kswapd使用预留内存；另外一种情况是被OOM选中杀死的进程在退出过程中，如果需要申请内存也可以使用预留部分。这两种情况下让他们使用预留内存可以避免系统进入deadlock状态。
http://kernel.taobao.org/index.php?title=Kernel_Documents/mm_sysctl
-->


https://huoding.com/2015/06/10/444

http://www.cnblogs.com/panfeng412/p/drop-caches-under-linux-system-2.html

https://www.halobates.de/memory.pdf

https://bhsc881114.github.io/2015/04/19/%E4%B8%80%E6%AC%A1linux%E5%86%85%E5%AD%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5-slab/

http://linuxperf.com/?p=148



系统启动时，BIOS 会自动发现当前物理内存的地址，而 Linux 内核会在启动时 (还在实地址模式) 通过 e820 BIOS 系统调用获取当前系统的物理内存地址，当然还有 IO 的映射地址等；开始内核会初始化部分内存供内核使用，然后调用 Bootmem 系统。

Linux 将内存分为不同的 Zone 管理，每个 Zone 中通过一个 buddy 系统分配内存，每次分配内存都是以页的 2 指数倍分配，例如内核页大小为 4K(getconf PAGE_SIZE)，那么分配的页为 4K, 8K, 16K, ..., 128K，一般系统最大为 128K 。

Buddy Allocator 最大的问题就是碎片，所以一般系统不会直接使用，通常其上层还包括了 Page Cache、Slab Allocator 。



## kswapd

一个内核守护进程，通过 ```module_init(kswapd_init)@mm/vmscan.c``` 初始化，然后通过 ```kthread_run()``` 启动 ```kswapd()``` 。该线程会维护一个 LRU 队列，会将最近没有被访问过的 (PG_referenced) 的页放入到 inactive 队列中；否则放置到 active 队列中。




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









在 Linux 内核启动时，会打印类似如下的内存初始化信息，由于此时内核的引导过程未完成，initrd 以及初始化占用的内存未释放，所以最终内核可用的内存要更大一些。

{% highlight text %}
----- 查看内核打印的内存信息，包括释放的内存
$ dmesg | grep -E "(Memory:|Freeing)"
[    0.000000] Memory: 5428672k/8970240k available (6765k kernel code,
                       686640k absent, 309176k reserved, 4432k data, 1680k init)
[    0.008667] Freeing SMP alternatives: 28k freed
[    0.728176] Freeing initrd memory: 27652k freed
[    2.757835] Freeing unused kernel memory: 1680k freed

----- 计算上述打印的可用内存
$ echo "5428672 + 28 + 27652 + 1680" | bc
5458032

----- 通过free命令查看当前内存
$ free -k
              total        used        free      shared  buff/cache   available
Mem:        8070604     2425256      714864      778268     4930484     4515144

5428672k/8970240k available
    分母表示可用物理内存的大小；
    分子表示可供kernel分配的free memory的大小；

absent：
  不可用的物理内存大小，包括了 BIOS 保留、Kernel 不可用的物理内存；
{% endhighlight %}

<!--

    330548k reserved：
    包括【initrd】和【内核代码及数据】等，详见上面的解释。其中内核代码和部分数据包含在下列统计值中：

        kernel code
          表示kernel的代码，属于reserved memory；
        data
          表示kernel的数据，属于reserved memory；
        init
          表示init code和init data，属于reserved memory，但引导完成之后会释放给free memory；

    available = 物理内存 - absent - reserved
    reserved 包括 kernel code, data, init，还有 initrd 和更多其它的内容；
-->

物理内存可以通过 ```dmidecode --type 17``` 查看，包括了很多详细的物理信息，不过对于虚机来说不支持。







## jjj

在如下的程序中打印出 tmp 变量的地址，变量保存在栈中，假设地址为 ```address:0x7FFF797B9698``` ，该结果是逻辑地址。现在的 Interl CPU 寄存器采用 64-bit，而其内存的寻址空间为 48-bits 。

{% highlight c %}
#include <stdio.h>
int main()
{
    unsigned long tmp = 0x12345678;
    printf("tmp variable address:0x%08lX\n", &tmp);
    return 0;
}
{% endhighlight %}

现在的 CPU 都会有内存管理单元，对于 IA-32e 类似架构的 CPU 规定，地址映射过程是 ```逻辑地址 -> 线性地址 -> 物理地址```。当然运行在 Intel 上的 Linux 内核也采用相同的方法。



## x86-64 示例

以如下的值为例。

<pre style="font-size:0.8em; face:arial;">
$ ./mem_map
tmp address:0x7FFE07DB9A70
CR4=1427E0 PSE(0) PAE(1)
CR3=12A6E0000 CR0=80050033, pgd:0xFFFF88012A6E0000
gdtr address:FFFF88021FB09000, limit:7F
</pre>

这里的 GDTR 地址是 0xFFFF88021FB09000，对应的物理地址为 0x21FB09000，实际上内核地址对应的物理地址是减去一个偏移 __PAGE_OFFSET ，也就是 0xFFFF880000000000。<br><br>

应用程序中，tmp 变量的逻辑地址或者说线性地址为 0x7FFE07DB9A70，我们逐步映射到实际物理地址，看这个地址的数据是否真的为 "0x12345678beaf5dde" 。
</p>

### 分段机制

分段的存在更多就是为了兼容性，在 x86-64 下的 64-bit 程序该功能近似于 bypass，对于 CS 会根据 GDT 中的信息判断是 64-bit 还是 32-bit compatibility 模式，而对于数据段的选择器如 DS、ES、FS、GS、SS 可以为 00H。而对于 GS、SS 可以不为 00H，手册说仍然有效，不过没有仔细研究。

也就是说对于 intel x86-64 我们可以直接忽略分段功能，而直接查看分页的处理。


### 分页机制

分页过程会将 48-bit 的线性地址转换为 52-bit 的物理地址, 可以看出虽然是 64bit 的操作系统但在处理器层面并没有提供 2^64 大小的内存访问范围。

另外需要注意的是，通过 CR3 映射的内存地址实际是 "物理地址"。

对于分页机制 IA-32e 有 4K、2M、1G 的分页方式，Linux 中采用的通常为 4K ，在此仅以此为例。


![memory segmentation paging]({{ site.url }}/images/linux/kernel/memory-segmentation-paging.png "memory segmentation paging"){: .pull-center width="80%" }

### 实践

这里的示例代码可以参考 [github memory]({{ site.example_repository }}/linux/memory/to_physical/) ，主要包括了如下的文件：

* registers.c 内核模块，用于读取 cr0、gdtr 等信息，可以通过 ```/proc/registers``` 读取；
* address.c 返回临时对象的地址，也就是栈地址，目前是 48bits ；





在分页机制中，有效的虚拟地址是 48-bits，会映射到 52-bits 的物理地址；将 tmp 的地址 0x7FFE07DB9A70 映射为二进制格式为。

<pre style="font-size:0.8em; face:arial;">
0111 1111 1111 1110 0000 0111 1101 1011 1001 1010 0111 0000
011111111 111111000 000111110 110111001 101001110000
       FF       1F8        3E       1B9          A70
</pre>

</p>
<br><h3>第一级映射</h3><p>
CR3 寄存器中的值是 0x12A6E0000，这是第一级映射表 (PML4) 的起始物理地址，这张表中保存着第二级映射表的物理地址。而线性地址中的 bits[47:39] 对应 PML4E 的序号。
<pre style="font-size:0.8em; face:arial;">
0x12A6E0000 + 011111111b * 8 = 0x12A6E0000 + 0x7F8 = 0x12A6E07F8

000012A6E07F0    00000001DB9A6067      00000001F9ACC067
000012A6E0800    0000000000000000      0000000000000000
</pre>
每个单元是 64-bits 因此需要乘以 8。最后，该地址对应的二级映射表的起始地址为：0x1F9ACC000(067 后面的 12-bits 是页面属性)。
<!--Bit17:PCIDE-->
</p>

<br><h3>第二级映射</h3><p>
二级映射的任务是找到第三级映射表的起始地址。
<pre style="font-size:0.8em; face:arial;">
0x1F9ACC000 + 111111000b * 8 = 0x1F9ACC000 + 0xFC0 = 0x1F9ACCFC0

00001F9ACCFC0    000000018B96D067      0000000000000000
00001F9ACCFD0    0000000000000000      0000000000000000
</pre>
与上述类似，第三级映射表的起始地址为 0x18B96D000(067 后面的 12-bits 是页面属性)。
</p>


<br><h3>第三级映射</h3><p>
三级映射的任务是找到第四级映射表的起始地址。
<pre style="font-size:0.8em; face:arial;">
0x18B96D000 + 000111110b * 8 = 0x18B96D000 + 0x1F0 = 0x18B96D1F0

000018B96D1F0    0000000154F81067      0000000000000000
000018B96D200    0000000000000000      0000000000000000
</pre>
第四级映射表的起始地址为 0x154F81000(067 后面 12-bits 是页面属性)。
</p>


<br><h3>第四级映射</h3><p>
第四级映射的任务是找到临时变量 tmp 所在的物理页面起始地址。
<pre style="font-size:0.8em; face:arial;">
0x154F81000 + 110111001b * 8 = 0x154F81000 + 0xDC8 = 0x154F81DC8

0000154F81DC0    800000011FDD7067      800000014DD61067
0000154F81DD0    0000000000000000      800000017C7A6067
</pre>
0x154F81DC8 地址单元中的数据，就是物理页面起始地址，也就是我们最后所得到的 tmp 对应的物理页为 0x14DD61000 。
</p>

<br><h4>最终物理地址计算</h4><p>
tmp 变量所在内存页面物理地址为 0x14DD61000，这个地址仅是物理页面地址，tmp 变量所在的物理地址为。
<pre style="font-size:0.8em; face:arial;">
0x14DD61000 + 101001110000b = 0x14DD61000 + 0xA70 = 0x14DD61A70

000014DD61A70    12345678BEAF5DDE      0000000000400830
000014DD61A80    0000000000000000      00007F1FF67EFAF5
</pre>
经过 4 级页面映射，终于找到了 tmp 对应的实际物理地址为 0x14DD61A70 ，对应的数据也为 0x12345678BEAF5DDE 。
</p>


## 其它

### sysrq

Linux 中，有一个非常强大的 sysrq 功能，通过该功能可以查看一些内核的当前运行信息，该信息会打印信息到内核的环形缓冲并输出到系统控制台，一般也会通过 syslog 输出到 ```/var/log/messages``` 。

有些发行版本该功能是关闭的，可以通过如下方式打开。

{% highlight text %}
# echo 1 > /proc/sys/kernel/sysrq           // 打开
# echo 0 > /proc/sys/kernel/sysrq           // 关闭
# vi /etc/sysctl.conf                       // 设置永久生效
kernel.sysrq = 1
{% endhighlight %}

一些常见的功能可以参考如下内容。

{% highlight text %}
# echo "b" > /proc/sysrq-trigger            // 立即重新启动计算机
# echo "o" > /proc/sysrq-trigger            // 立即关闭计算机
# echo "m" > /proc/sysrq-trigger            // 导出内存分配的信息到demsg
# echo "p" > /proc/sysrq-trigger            // 导出当前CPU寄存器信息和标志位的信息
# echo "t" > /proc/sysrq-trigger            // 导出线程状态信息
# echo "c" > /proc/sysrq-trigger            // 故意让系统崩溃
# echo "s" > /proc/sysrq-trigger            // 立即重新挂载所有的文件系统
# echo "u" > /proc/sysrq-trigger            // 立即重新挂载所有的文件系统为只读
{% endhighlight %}

## 参考

[CS 635: Advanced Systems Programming](http://cs.usfca.edu/~cruse/cs635/) 网上一个牛掰的课程网站，包括了一些不错的源码，包括上面的 dram 和 fileview 。

[关于Linux内存地址映射的介绍](http://ilinuxkernel.com/?p=1276)，可以参考 [本地文档](/reference/linux/kernel/linux_memap.tar.bz2) ，含有x86_32/64详解，该博客对于Linux内存有比较详细的介绍；同时可以参考 [Linux 从虚拟地址到物理地址](http://blog.chinaunix.net/uid-24774106-id-3427836.html) 。

<a href="http://www.mouseos.com/arch/segmentation.html">segmentation 情景分析</a> 关于段选择寄存器比较详细的介绍。

<!--

https://software.intel.com/en-us/articles/intel-sdm
-->

## 物理地址转换

现在几乎所有的操作系统都支持虚拟地址 (Virtual Address) ，每个用户空间的进程都有自己的虚拟地址空间，内核配合 CPU 硬件 MMU 完成到物理地址的转换。

Linux 内核保存了地址转换相关的数据结构，正常来说无法在用户空间访问，在 2.6.25 之后，提供了 ```/proc/$(pid}/pagemap``` ```/proc/kpagecount``` ```/proc/kpageflags``` 完成虚拟地址到物理地址的转换。


### /proc/${pid}/pagemap

该文件包含了该进程的虚拟地址到物理地址转换相关的映射关系，每个转换包含了 64-bits，详细的内容可以参考内核文档 [pagemap.txt]({{ site.kernel_docs_url }}/Documentation/vm/pagemap.txt) 。

![memory proc pagemap]({{ site.url }}/images/linux/kernel/memory-proc-pagemap.png "memory proc pagemap"){: .pull-center width="80%" }

{% highlight text %}
$ cat /proc/$(pidof mysqld)/maps | head -3
00400000-0283f000 r-xp 00000000 08:09 1308936      /opt/mysql-5.7/bin/mysqld
02a3e000-02b18000 r--p 0243e000 08:09 1308936      /opt/mysql-5.7/bin/mysqld
02b18000-02bc7000 rw-p 02518000 08:09 1308936      /opt/mysql-5.7/bin/mysqld

$ ./pagemap `pidof mysqld` 0x00400000
{% endhighlight %}

可以查看源码 [pagemap.c]( {{ site.example_repository }}/linux/memory/pagemap/pagemap.c ) 以及 [pagemap.py]( {{ site.example_repository }}/linux/memory/pagemap/pagemap.py ) 。

<!--
https://github.com/dwks/pagemap
http://blog.jeffli.me/blog/2014/11/08/pagemap-interface-of-linux-explained/
http://www.cnblogs.com/lanrenxinxin/p/6216925.html
-->





## 内存分配

通常在用户空间中会通过 malloc() 动态申请内存，而实际上，在用户空间中对应了不同的实现方式，包括了 ptmalloc (glibc)、 tcmalloc (Google) 以及 jemalloc，接下来简单介绍这三种内存分配方式。

ptmalloc 的早期版本是由 Doug Lea 实现的，它有一个重要问题就是无法保证线程安全，Wolfram Gloger 改进了其实现从而支持多线程，

TCMalloc (Thread-Caching Malloc) 是 google 开发的开源工具 "google-perftools" 之一。

<!--
http://www.360doc.com/content/13/0915/09/8363527_314549128.shtml
http://blog.jobbole.com/91887/
http://blog.chinaunix.net/uid-26772535-id-3197173.html
http://blog.csdn.net/jltxgcy/article/details/44150429
http://blog.csdn.net/jltxgcy/article/details/44133307
http://www.cnblogs.com/vinozly/p/5489138.html
-->

### brk() sbrk()

{% highlight c %}
#include <unistd.h>
int brk(void *addr);
void *sbrk(intptr_t increment);
{% endhighlight %}

这两个函数都用来改变 "program break" 的位置，如下图所示：

![memory userspace layout]({{ site.url }}/images/linux/kernel/memory-userspace-layout.jpg "memory userspace layout"){: .pull-center width="70%" }

sbrk/brk 是从堆中分配空间，实际上就是移动一个位置，也就是 ```Program Break```，这个位置定义了进程数据段的终止处，增大就是分配空间，减小就是释放空间。

sbrk 用相对的整数值确定位置，如果这个整数是正数，会从当前位置向后移若干字节，如果为负数就向前若干字节，为 0 时获取当前位置；而 brk 则使用绝对地址。

{% highlight c %}
#include <stdio.h>
#include <unistd.h>

int main(int argc, char **argv)
{
    void *p0 = sbrk(0), *p1, *p2;
    printf("P0 %p\n", p0);
    brk(p0 + 4);     // 分配4字节
    p1 = sbrk(0);
    printf("P1 %p\n", p1);
    p2 = sbrk(4);
    printf("P2=%p\n", p2);

    return 0;
}
{% endhighlight %}



## mmap

对应到内核中的接口是 ```sys_mmap()``` 实际上最终调用的是 ```sys_mmap_pgoff()``` 函数，当然实际上这也是一个系统调用函数。


{% highlight text %}
sys_mmap_pgoff()
 |-vm_mmap_pgoff()
   |-security_mmap_file()       权限检查
   |-down_write()
   |-do_mmap_pgoff()
     |-PAGE_ALIGN()             会做若干检查判断是否超过限制，包括sysctl_max_map_count
     |-get_unmapped_area()      从用户地址空间寻找个合适的地址
     |-calc_vm_prot_bits()      根据prot和flags计算获取vm_flags
     |-calc_vm_flag_bits()
     |-can_do_mlock()           判断是否需要锁定内存
     |-mlock_future_check()
     |-mmap_region()            真正的映射函数
   |-up_write()
   |-mm_populate()
{% endhighlight %}

内存锁定的含义是，分配的内存始终位于真实内存之中，从不被置换 (swap) 出去，在应用层可以通过 ```mlock()``` 函数实现。

```mlock()``` 会锁定开始于地址 addr 并延续长度为 len 个地址范围的内存，调用成功返回后所有包含该地址范围的分页都保证在 RAM 内，这些分页保证一直在 RAM 内直到后来被解锁。



            mmap_region(file, addr, len, flags, vm_flags, pgoff);
                vma_merge
                kmem_cache_zalloc
                    file->f_op->mmap(file, vma); ==> generic_file_mmap
                    shmem_zero_setup(vma);


### mtrace

该工具可以用来协助定位内存泄露，默认没有安装，在 CentOS 中可以通过如下方式安装。

{% highlight text %}
# yum install glibc-utils
{% endhighlight %}

假设有如下的测试代码。


{% highlight text %}
$ cat main.c
#include <stdio.h>
#include <stdlib.h>
#include <mcheck.h>
int main(int argc, char **argv)
{
    setenv("MALLOC_TRACE", "taoge.log", 1);
    mtrace();

    malloc(2 * sizeof(int));

    return 0;
}

$ gcc -Wall -g -o foobar main.c
$ ./foobar
$ mtrace foobar trace.log
{% endhighlight %}

可以看到，有内存泄露，且正确定位到了代码的行数。实际上，mtrace 原理很简单，就是记录每一对 malloc/free 的调用情况，然后检查是否有内存没有释放。




<!--
// int mallopt(int param, int value);
// mtrace muntrace mcheck mcheck_pedantic mcheck_check_all mprobe
// malloc_stats mallinfo malloc_trim malloc_info

答：brk是系统调用，主要工作是实现虚拟内存到内存的映射，可以让进程的堆指针增长一定的大小，逻辑上消耗掉一块虚拟地址空间，malloc向OS获取的内存大小比较小时，将直接通过brk调用获取虚拟地址。

mmap是系统调用，也是实现虚拟内存到内存的映射，可以让进程的虚拟地址区间切分出一块指定大小的虚拟地址空间vma_struct，一个进程的所有动态库文件.so的加载，都需要通过mmap系统调用映射指定大小的虚拟地址区间，被mmap映射返回的虚拟地址，逻辑上被消耗了，直到用户进程调用unmap，会回收回来。malloc向系统获取比较大的内存时，会通过mmap直接映射一块虚拟地址区间。

malloc是C语言标准库中的函数，主要用于申请动态内存的分配，其原理是当堆内存不够时，通过brk/mmap等系统调用向内核申请进程的虚拟地址区间，如果堆内部的内存能满足malloc调用，则直接从堆里获取地址块返回。

new是C++内置操作符，用于申请动态内存的分配，并同时进行初始化操作。其实现会调用malloc，对于基本类型变量，它只是增加了一个cookie结构, 比如需要new的对象大小是 object_size, 则事实上调用 malloc 的参数是 object_size + cookie， 这个cookie 结构存放的信息包括对象大小，对象前后会包含两个用于检测内存溢出的变量，所有new申请的cookie块会链接成双向链表。
对于自定义类型，new会先申请上述的大小空间，然后调用自定义类型的构造函数，对object所在空间进行构造。
-->







ptmalloc







15600615832

## 物理内存探测

Linux 被 bootloader 加载到内存后，首先执行的是 ```_start()@arch/x86/boot/header.S/header.S```，该函数在做了一些准备工作后会跳转到 boot 目录下的 ```main()@main.c``` 函数执行，第一次内存相关的调用是在实模式时，通过调用 ```detect_memory()``` 实现。

如下是该函数的实现。

{% highlight c %}
int detect_memory(void)
{
    int err = -1;

    if (detect_memory_e820() > 0)
        err = 0;

    if (!detect_memory_e801())
        err = 0;

    if (!detect_memory_88())
        err = 0;

    return err;
}
{% endhighlight %}

该函数会依次尝试调用 ```detect_memory_e820()```、```detect_memory_e801()```、```detect_memory_88()``` 获得系统物理内存布局，这三个函数都在 memory.c 中实现。

其内部都会以内联汇编的形式调用 bios 中断以取得内存信息，该中断调用形式为 ```int 0x15```，同时调用前分别把 ```AX``` 寄存器设置为 ```E820h```、```E801h```、```88h```，该功能分别用于获取系统内存布局、获取内存大小、获取扩展内存大小，关于 ```0x15``` 号中断详细信息可以去查询相关手册。

在 x86 中，IO 设备也会映射到内存空间，也就是说系统使用的物理内存空间是不连续的，被分成了很多段，而且每段的属性也不一样。通过 ```int 0x15``` 查询物理内存时，每次返回一个内存段的信息，因此要想返回系统中所有的物理内存，必须以迭代的方式去查询。

### 内存查询

目前使用较多的是 ```e820``` ，可以通过 ```dmesg``` 查看内核启动输出，一般有类似 ```e820: BIOS-provided physical RAM map``` 的输出，下面以 ```e820``` 为例。

> e820 是和 BIOS 的 int 0x15 中断相关的，之所以叫 e820 是因为在用这个中断时 AX 必须是 0xe820。

其中，与次相关的结构体如下。

{% highlight c %}
struct e820entry {
    __u64 addr;             /* start of memory segment */
    __u64 size;             /* size of memory segment */
    __u32 type;             /* type of memory segment */
} __attribute__((packed));

struct e820map {
    __u32 nr_map;
    struct e820entry map[E820_X_MAX];
};
{% endhighlight %}

在 ```detect_memory_e820()``` 函数中，把 ```int 0x15``` 放到一个 do-while 循环里，将每次得到的内存段放到 ```struct e820entry``` 里。像其它启动时获得的结果一样，最终都会被放到 ```boot_params``` 里，探测到的各个内存段情况被放到了 ```boot_params.e820_map``` 。

{% highlight text %}
main()@arch/x86/boot/main.c
 |-detect_memory()                 ← 探测物理内存
   |-detect_memory_e820()
   |-detect_memory_e801()
   |-detect_memory_88()

start_kernel()
 |-setup_arch()                    ← 完成与体系结构相关的初始化工作
   |-setup_memory_map()
     |-e820_print_map()
{% endhighlight %}

Linux 物理内存管理区会在 ```start_kernel()``` 函数中进行初始化，此时启动分配器已经建立，所以可以从bootmem中分配需要的内存。

在 ```e820_print_map()``` 函数中，会打印如下内容。

{% highlight text %}
[    0.000000] e820: BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x0000000000057fff] usable
[    0.000000] BIOS-e820: [mem 0x0000000000058000-0x0000000000058fff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000059000-0x000000000009cfff] usable
[    0.000000] BIOS-e820: [mem 0x000000000009d000-0x00000000000fffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x00000000cc8e2fff] usable
[    0.000000] BIOS-e820: [mem 0x00000000cc8e3000-0x00000000cc8e3fff] ACPI NVS
[    0.000000] BIOS-e820: [mem 0x00000000cc8e4000-0x00000000cc90dfff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000cc90e000-0x00000000d6202fff] usable
[    0.000000] BIOS-e820: [mem 0x00000000d6203000-0x00000000d7f52fff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000d7f53000-0x00000000d7fa2fff] ACPI NVS
[    0.000000] BIOS-e820: [mem 0x00000000d7fa3000-0x00000000d7ffefff] ACPI data
[    0.000000] BIOS-e820: [mem 0x00000000d7fff000-0x00000000d7ffffff] usable
[    0.000000] BIOS-e820: [mem 0x00000000d8000000-0x00000000d80fffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000d8200000-0x00000000db7fffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000f80fa000-0x00000000f80fafff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000f80fd000-0x00000000f80fdfff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000fe000000-0x00000000fe010fff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000100000000-0x00000002237fffff] usable
{% endhighlight %}

<!--
http://tomhibolu.iteye.com/blog/1214876
-->


## 内存初始化

通过 BIOS 取得所有的内存布局之后，Linux 会对所获取的内存块做相关的检查并保存对其处理后的区域，也就是内存活动区域，会保存在 ```struct node_active_region``` 中。

另外，在 ```arch/x86/kernel/x86_init.c``` 中，维护了一个 ```x86_init``` 变量，用于保存常见的初始化函数，如页表的初始化函数 ```pagetable_init()```，实际指向的是 ```paging_init()``` 函数。

接着会对 zones 进行初始化，设置最终的分页机制等；系统的内存信息保存在 ```/proc/meminfo``` 中，内核实现可以参考 ```fs/proc/meminfo.c``` 。

> MTRR (Memory Type Range Register) 用来确定系统内存中一段物理内存的类型，进而可以控制处理器对内存区域的访问，也就是告诉 CPU 在解析或者说操作内存的时候应该用什么手段，常见的有 Write Through(WT)、Write Back(WB)、Write Protected(WP)等。
>
> 系统当前的 mtrr 信息保存在 /proc/mtrr 中。

如下，详细介绍 Linux 中内存的初始化过程，首先需要注意的是，64-bits 没有 HIGH 。

{% highlight text %}
start_kernel()@init/main.c
 |-setup_arch()                           ← 完成与体系结构相关的初始化工作
 | |-setup_memory_map()                   ← 建立内存图
 | | |-x86_init.resources.memory_setup()
 | | | |-sanitize_e820_map()              ← 消除内存重叠部分
 | | | |-append_e820_map()                ← 将内存配置从boot_params.e820_map拷贝到e820中
 | | |   |-e820_add_region()              ← 将内存段的信息保存到e820的map数组中
 | | |-e820_print_map()                   ← 打印出物理内存的分布
 | |
 | |-e820_end_of_ram_pfn()                ← 找出最大的可用页帧号，后面会找出低端内存的最大页面号
 | | |-e820_end_pfn()                     ← 会打印last_pfn、max_arch_pfn
 | |
 | |-mtrr_bp_init()                       ← 设置MTRR
 | |-init_mem_mapping()                   ← 设置最终的内存映射机制
 | | |-probe_page_size_mask()
 | | |  |-init_gbpages()
 | | |-init_memory_mapping()              ← 入参是(0,0x100000)，该函数中会打印一系列的mapping信息
 | | | |-split_mem_range()
 | | | |-kernel_physical_mapping_init()   ← 完成虚拟地址到物理地址的映射
 | | | | |-pgd_populate()
 | | | | |-__flush_tlb_all()
 | | | |-add_pfn_range_mapped()
 | | |-load_cr3()
 | | |-__flush_tlb_all()
 | | |-early_memtest()
 | |
 | |-early_trap_pf_init()
 | |-setup_real_mode()
 | |-memblock_set_current_limit()
 | |-reserve_initrd()                     ← 设置RADDISK
 | |-acpi_boot_table_init()               ← ACPI设置
 | |
 | |-initmem_init()                       ← 初始化内存分配器
 | |-x86_init.paging.pagetable_init()     ← 建立完整的页表，实际调用paging_init()
 |   |-sparse_init()
 |   |-node_clear_state()
 |   |-zone_sizes_init()                  ← 在此设置各个区，可以查看/proc/meminfo
 |     |-free_area_init_nodes()           ← 打印Zone ranges信息
 |
 |-build_all_zonelists()                  ← 区域链表设置
 |-page_alloc_init()

{% endhighlight %}

<!--
page_table_range_init()
kmem_cache_init()   # 打印SLUB相关信息
-->

在设置内存时，首先会打印出系统的物理内存分布，然后找出最大物理页面帧号 ```max_pfn```，低端内存的最大页面号 ```max_low_pfn``` 。

### 页表初始化

在 2.6.11 后，Linux 采用四级分页模型：1) 页全局目录 (Page Global Directory, PGD)；2) 页上级目录 (Page Upper Directory, PUD)；3) 页中间目录 (Page Middle Directory, PMD)；4) 页表 (Page Table, PT)。

对于没有启动 PAE 的 32 位系统，实际上只用到了两级分页，将 PUD、PMD 设置为 0 达到使用两级分页的目的，但为了保证程序能 32 位和 64 系统上都能运行，内核保留了页上级目录和页中间目录在指针序列中的位置。


## 内存分配



## Page Allocator



### alloc_page()

通过 ```alloc_page()``` 函数完成页的分配，在使用时需要通过 ```page_address()``` 完成线性地址转换，详细可以参考 [alloc.c]({{ site.example_repository }}/linux/LKM/memory/alloc.c) 示例程序。

内核通过 struct page 描述一个页，所有的页描述符存放在全局 mem_map 数组中，每个 page 代表一个物理页面，整个数组代表着系统中的全部物理页面，数组的下标为页框号 (pfn)，代表了 page 结构对应第几个物理页面。

页面表项的高 20 位对于软件和 MMU 硬件有着不同的意义，对于软件，这是一个物理页面的序号，将这个序号用作下标就可以从 mem_map 找到代表这个物理页面的 page 数据结构，对于硬件，则 (在低位补上12个0后) 就是物理页面的起始地址。

根据是否配置了 ```CONFIG_NUMA``` 会调用不同的函数，一般来说很多发行版本都会配置该选项。

{% highlight c %}
#ifdef CONFIG_NUMA
static inline struct page *
alloc_pages(gfp_t gfp_mask, unsigned int order)
{
    return alloc_pages_current(gfp_mask, order);
}
#else
#define alloc_pages(gfp_mask, order) \
        alloc_pages_node(numa_node_id(), gfp_mask, order)
#endif
{% endhighlight %}

接下来看看实现的详细细节。

{% highlight text %}
alloc_pages()
 |-alloc_pages_current()
   |-get_task_policy()                  获取内存的分配策略
   |-__alloc_pages_nodemask()           buddy内存分配系统的核心部分
     |-first_zones_zonelist()
     |-get_page_from_freelist()         快速分配内存
     | |-zone_watermark_ok()            判断设置的内存水位
     | |-buffered_rmqueue()             从zone中分配order阶的页帧
     |   |-rmqueue_bulk()               对于单页分配
     |     |-__rmqueue()                从伙伴系统中分配指定的页
     |     | |-__rmqueue_smallest()
     |     | | |-expand()               处理请求页小于当前页的情况
     |     | |   |-list_add()           将页一分为二，并添加到链表中
     |     | |-__rmqueue_fallback()
     |     |-__mod_zone_page_state()    更新页的统计状态
     |
     |-__alloc_pages_slowpath()         慢速分配
{% endhighlight %}




<!--

Linux进程的线性地址空间（进程虚拟地址空间分布），0~3G是User地址空间，3~4G是Kernel地址空间。(适用于ARM、X86等，mips按0~2G,2~3G划分)

关于线性地址布局，此图未说明部分：

1.紧接着内核数据区向上是mem_map全局page数组。

2. kernel启动地址并非是0xC0000000，而是PAGE_OFFSET+TEXT_OFFSET（0x8000），而在这32k大小的空间存放着内核一级页表数组swapper_pg_dir（每一项一级页表4个字节，映射1M内存，4G空间共4K项一级页表，占用16k内存），页表在启动阶段setup_arch->paging_init中调用prepare_page_table()和map_lowmem()进行初始化和映射，将在另一篇详细叙述。

3.内核动态加载驱动模块so将被load到紧接着MODULES_VADDR~0xC0000000的16M空间。


小于896M的物理内存直接映射到内核3G~3G+896M线性地址空间内，VA=PA-PHYS_OFFSET+PAGE_OFFSET

大约896M的物理内存通过建立各级页表的方式映射到高端映射区。

每个task在起task_struct内都有一个指针mm指向mm_struct，其控制着该task的所有内存信息，其成员mmap指向vma区链表，pgd指向页全局目录位置。

切换任务时要为下一个task装载其pgd到C2（X86为cr3）。

在cpu寻址时，mmu将虚拟地址按照2级映射方式，映射到对应物理叶匡的地址偏移上去。

而在内存管理上，系统启动阶段使用的是Bootmem，它是以简单的bitmap方式（0：未使用，1：已用）。

系统启动后buddy接手内存管理（以页为单位），分为两种情况，per_cpu_pageset和free_area，前者用来分配单个页（在Linux物理内存描述三个层级中有详细描述），后者用来分配多页的情况，11个链表管理页块的大小从2^1~2^11，每个链表中又分为不同迁移类型的子链表。

#define MIGRATE_UNMOVABLE     0
#define MIGRATE_RECLAIMABLE   1
#define MIGRATE_MOVABLE       2
#define MIGRATE_PCPTYPES      3 /* the number of types on the pcp lists */
#define MIGRATE_RESERVE       3
#define MIGRATE_ISOLATE       4 /* can not allocate from here */

MIGRATE_TYPES是为了有效解决内存碎片问题而引入的，将可移动（用户空间申请的内存可以重新映射）、不可移动（内核空间通常申请的）、可回收（文件映射）等组织在不同的链表中，申请内存时按照不同情况在各自类型的链表中释放，如果所在类型内存不足，会fallback到其他类型链表继续分配。在系统初始化完成时全部内存划分给MOVABLE类型链表，当有其他类型需求时再从MOVABLE中释放生成。

slab分配器是面向对象的分配机制，其建立在buddy基础之上，细节待另一篇详述。
-->


## Buddy System

## Slub

## 用户空间内存申请

## 进程加载

## mmap

内存映射有几种应用场景，可以将文件或者设备 IO 映射到内存，从而提高 IO 效率，可以将用户内存空间映射到内核的内存空间。

内核中，通过 ```ioremap()``` 映射设备 IO 地址空间以供内核访问，kmap 映射申请的高端内存，DMA 使用比较多的是网卡驱动里 ring buffer 机制。

{% highlight text %}
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);

prot: 设置页面的保护级别，注意不要与文件打开模式冲突
  PROT_EXEC  页可执行
  PROT_READ  页可读
  PROT_WRITE 页可写
  PROT_NONE  页不可以访问
flags:
  MAP_SHARED  共享内存映射，更新此内存对其它进程可见，而且会同步到文件，调用msync后写入文件；
  MAP_PRIVATE 创建COW映射，更新时对其它进程不可见，而且不会更新到文件；
{% endhighlight %}

### 文件映射

可以通过 mmap 方式修改一个文件：A) 打开文件，获取对应的文件描述符；B) 通过 ```mmap()``` 将对应的文件映射到内存；C) 现在可以关闭文件，然后直接通过内存修改；D) 通过 ```msync()``` 将内存的内容同步到文件，然后 ```munmap()``` 。

可以参考示例程序 [filemap.c]( {{ site.example_repository }}/linux/memory/mmap/0-filemap.c)，该程序会读取 file.txt 文件并修改其中的一个字符。

### 共享内存

通过 mmap 可以实现 A) 文件映射，通常用于不相关的进程通讯；B) 匿名映射 (通过fork关联) 。

#### 文件映射

可以参考示例程序 [sharemem1.c]( {{ site.example_repository }}/linux/memory/mmap/1-sharemem1.c) 以及 [sharemem2.c]( {{ site.example_repository }}/linux/memory/mmap/1-sharemem2.c) 。

两个进程之间通过 mmap 共享内存，其中 ```sharemem1``` 会先初始化内存，然后睡眠 11 秒，此时可以通过 ```sharemem2``` 读取其中的内容；11 秒过后，将文件内容重新设置为 0 。

当 ```sharemem1``` 输出 ```initialize over``` 后，查询的信息不再是 0 ，而随后会再次设置为 0。

#### 匿名映射

使用匿名映射，常用在父子进程中通讯，可以参考示例程序 [anonymem.c]( {{ site.example_repository }}/linux/memory/mmap/2-anonymem.c) 。

### 实现原理

简单来说，就是内核是如何保证各个进程寻址到同一个共享内存区域的内存页面。

在 Linux 中，内存页面的所有信息通过 ```struct page``` 表示，该结构体在 ```include/linux/mm_types.h``` 中定义，其中有一个域指针 mapping 。

{% highlight c %}
struct page {
    unsigned long flags;
    union {
        struct address_space *mapping;
        void *s_mem;
    };
}
{% endhighlight %}



1、 page cache及swap cache中页面的区分：一个被访问文件的物理页面都驻留在page cache或swap cache中，一个页面的所有信息由struct page来描述。struct page中有一个域为指针mapping ，它指向一个struct address_space类型结构。page cache或swap cache中的所有页面就是根据address_space结构以及一个偏移量来区分的。

2、文件与 address_space结构的对应：一个具体的文件在打开后，内核会在内存中为之建立一个struct inode结构，其中的i_mapping域指向一个address_space结构。这样，一个文件就对应一个address_space结构，一个 address_space与一个偏移量能够确定一个page cache 或swap cache中的一个页面。因此，当要寻址某个数据时，很容易根据给定的文件及数据在文件内的偏移量而找到相应的页面。

3、进程调用mmap()时，只是在进程空间内新增了一块相应大小的缓冲区，并设置了相应的访问标识，但并没有建立进程空间到物理页面的映射。因此，第一次访问该空间时，会引发一个缺页异常。

4、 对于共享内存映射情况，缺页异常处理程序首先在swap cache中寻找目标页（符合address_space以及偏移量的物理页），如果找到，则直接返回地址；如果没有找到，则判断该页是否在交换区 (swap area)，如果在，则执行一个换入操作；如果上述两种情况都不满足，处理程序将分配新的物理页面，并把它插入到page cache中。进程最终将更新进程页表。
注：对于映射普通文件情况（非共享映射），缺页异常处理程序首先会在page cache中根据address_space以及数据偏移量寻找相应的页面。如果没有找到，则说明文件数据还没有读入内存，处理程序会从磁盘读入相应的页 面，并返回相应地址，同时，进程页表也会更新。

5、所有进程在映射同一个共享内存区域时，情况都一样，在建立线性地址与物理地址之间的映射之后，不论进程各自的返回地址如何，实际访问的必然是同一个共享内存区域对应的物理页面。
注：一个共享内存区域可以看作是特殊文件系统shm中的一个文件，shm的安装点在交换区上。

上面涉及到了一些数据结构，围绕数据结构理解问题会容易一些。


## Cache

BufferCache

        块缓冲，通常1K，对应于一个磁盘块，用于减少磁盘IO
        由物理内存分配，通常空闲内存全是bufferCache
        应用层面，不直接与BufferCache交互，而是与PageCache交互（见下）
        读文件：

直接从bufferCache中读取

    写文件：

       方法一，写bufferCache，后写磁盘

       方法二，写bufferCache，后台程序合并写磁盘

 

PageCache

    页缓冲/文件缓冲，通常4K，由若干个磁盘块组成（物理上不一定连续），也即由若干个bufferCache组成
    读文件：

      可能不连续的几个磁盘块》》bufferCache》》pageCache》》应用程序进程空间

    写文件：

       pageCache, bufferCache》》磁盘

 

SwapCache

    交换空间（虚拟内存的表现形式）

 

如何使用PageCache

    以下日志，属摘抄部分，我自己还没理解。
    【1】通过VFS直接在不同文件的Cache之间或者Cache与应用程序所提供的用户空间buffer之间拷贝数据，其实现原理如下图

    【2】是通过VMM（虚拟内存管理）将Cache项映射到用户空间，使得应用程序可以像使用内存指针一样访问文件，Memory map访问Cache的方式在内核中是采用请求页面机制实现的，其工作过程如图

    【3】总结1与2，总的访问图：


#### Swap Cache

它的主要作用不是为了提高磁盘的 IO 效率，而是为了解决页面在 swap in 和 swap out 时的同步问题，也就是在进行 swap out (将页面内容写入磁盘分区时) 进程如果发起了对换出页面的访问，系统如何对其处理。

由于存在 swap cache ，如果页面的数据还没有完全写入磁盘，这个 page frame 是在 swap cache 中的；等数据完全写入磁盘后，且没有进程对 page frame 访问时，才会真正释放 page frame，然后将其交给 buddy system 。


 page cache是一种策略，就是使用完的page并不是立即放到内核的
free page list中，而是暂时缓存着已被再次使用。这样系统中的
缓冲的page就越来越多，系统中的free page就越来越少，怎么办？
系统通过内核线程来完成缓存中页面的回收工作，这些内核线程是
定期被调用的，平时则处在睡眠的状态，如果进程需要page而free
page严重短缺的时候，进程可以唤醒这些内核线程来回收缓存的页面，
这样，一方面缓存，一方面回收达到一种平衡，同时改善了系统的性能。
 内核中在多处使用了page cache策略，最典型的有：页面交换，和磁盘
文件的读取。不过实现的方法无非是让不同状态的page，处在不同的
list中，而回收的内核线程从可以回收的队列中回收page。

swap cache主要是存放那些无根（就是说没有文件系统中的某个文件和其
对应）的page，例如你用malloc分配出来的。
它对应的file device就是swapfile。

它和page cache的区别在于，当文件从file system上读取出来的时候，
它的内容就会同时读入page cache中。但是当你用malloc分配内存的
时候，并不马上放到swap cache中，而是在进程中不再使用该内存的
时候它才被读入swap cache中。

buffer cache和page cache在2.4 内核里面几乎么啥大区别，我们完全
可以通过对page cache的操作来实现把数据写入到disk中，而不一定要
通过buffer cache。如果要说区别的话，主要是page cache的大小是固定
的，如果你是要从软盘设备（它的block大小是512字节），就可以用一页
来保存好几个block。这样buffer cache可以同时拥有连续的好几页page cache
中的page。



{% highlight text %}
{% endhighlight %}
