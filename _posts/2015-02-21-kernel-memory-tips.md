---
title: Kernel 内存杂项
layout: post
comments: true
language: chinese
category: [linux]
keywords: linux,内存,kernel,内存空间
description: 简单介绍下内核中与内存相关的内容。
---

简单介绍下内核中与内存相关的内容。

<!-- more -->

## OverCommit

内核中内存的 OverCommit 就是操作系统承诺给所有进程的内存大小超过了实际可用的内存，主要是由于物理内存页的分配发生在使用时，而非申请时，也就是所谓的 COW 机制。

例如，程序通过 ```malloc()``` 申请了 200MB 内存，不过实际只使用了 100MB 的内存，也就是说，只要你来申请内存我就给你，寄希望于进程实际上用不到那么多内存。

如果申请了内存之后，确实会有大量进程使用内存，那么就会发生类似银行的 "挤兑"，此时就会使用 Linux 的 OOM killer 机制。

<!--
(OOM = out-of-memory)来处理这种危机：挑选一个进程出来杀死，以腾出部分内存，如果还不够就继续杀…也可通过设置内核参数 vm.panic_on_oom 使得发生OOM时自动重启系统。这都是有风险的机制，重启有可能造成业务中断，杀死进程也有可能导致业务中断，我自己的这个小网站就碰到过这种问题，参见前文。所以Linux 2.6之后允许通过内核参数 vm.overcommit_memory 禁止memory overcommit。
-->

其行为可以通过 ```vm.overcommit_memory``` 参数修改，包括了三种取值：

* 0 (Heuristic Overcommit Handling)，默认值<br>允许 OC ，不过如果申请的内存过大，那么系统会拒绝请求，而判断的算法在后面介绍。
* 1 (Always Overcommit)<br>允许 OC，对内存申请来者不拒。
* 2 (Don't Overcommit)<br>禁止 OC 。

当上述的值为 2 时，会禁止 OC，那么怎么才算是 OC 呢？实际上，Kernel 设有一个阈值，申请的内存总数超过这个阈值就算 OC，阀值可以通过 ```/proc/meminfo``` 文件查看：

{% highlight text %}
# grep -i commit /proc/meminfo
CommitLimit:    12162784 kB
Committed_AS:    6810268 kB
{% endhighlight %}

其中 ```CommitLimit``` 就是对应的阈值，超过了该值就算 Overcommit ，该值通过 ```vm.overcommit_ratio``` 或 ```vm.overcommit_kbytes``` 间接设置的，公式如下：

{% highlight text %}
CommitLimit = (Physical RAM * vm.overcommit_ratio / 100) + Swap
{% endhighlight %}

其中 ```vm.overcommit_ratio``` 的默认值是 50，也就是表示物理内存的 50% ，当然，也可以直接通过 ```vm.overcommit_kbytes``` 即可。注意，如果 huge pages 那么就需要从物理内存中减去，公式变成：

{% highlight text %}
CommitLimit = ([total RAM] – [total huge TLB RAM]) * vm.overcommit_ratio / 100 + swap
{% endhighlight %}

另外，```/proc/meminfo``` 中的 ```Committed_AS``` 表示所有进程已经申请的内存总大小；注意，是已经申请的，不是已经分配的。如果 ```Committed_AS``` 超过 ```CommitLimit``` 就表示发生了 OC 。


<!--
“sar -r”是查看内存使用状况的常用工具，它的输出结果中有两个与overcommit有关，kbcommit 和 %commit：
kbcommit对应/proc/meminfo中的 Committed_AS；
%commit的计算公式并没有采用 CommitLimit作分母，而是Committed_AS/(MemTotal+SwapTotal)，意思是_内存申请_占_物理内存与交换区之和_的百分比。
$ sar -r
05:00:01 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
05:10:01 PM    160576   3648460     95.78         0   1846212   4939368     62.74   1390292   1854880         4

附：对Heuristic overcommit算法的解释

内核参数 vm.overcommit_memory 的值0，1，2对应的源代码如下，其中heuristic overcommit对应的是OVERCOMMIT_GUESS：

源文件：source/include/linux/mman.h

#define OVERCOMMIT_GUESS                0
#define OVERCOMMIT_ALWAYS               1
#define OVERCOMMIT_NEVER                2

Heuristic overcommit算法在以下函数中实现，基本上可以这么理解：
单次申请的内存大小不能超过 【free memory + free swap + pagecache的大小 + SLAB中可回收的部分】，否则本次申请就会失败。
源文件：source/mm/mmap.c 以RHEL内核2.6.32-642为例

0120 /*
0121  * Check that a process has enough memory to allocate a new virtual
0122  * mapping. 0 means there is enough memory for the allocation to
0123  * succeed and -ENOMEM implies there is not.
0124  *
0125  * We currently support three overcommit policies, which are set via the
0126  * vm.overcommit_memory sysctl.  See Documentation/vm/overcommit-accounting
0127  *
0128  * Strict overcommit modes added 2002 Feb 26 by Alan Cox.
0129  * Additional code 2002 Jul 20 by Robert Love.
0130  *
0131  * cap_sys_admin is 1 if the process has admin privileges, 0 otherwise.
0132  *
0133  * Note this is a helper function intended to be used by LSMs which
0134  * wish to use this logic.
0135  */
0136 int __vm_enough_memory(struct mm_struct *mm, long pages, int cap_sys_admin)
0137 {
0138         unsigned long free, allowed;
0139
0140         vm_acct_memory(pages);
0141
0142         /*
0143          * Sometimes we want to use more memory than we have
0144          */
0145         if (sysctl_overcommit_memory == OVERCOMMIT_ALWAYS)
0146                 return 0;
0147
0148         if (sysctl_overcommit_memory == OVERCOMMIT_GUESS) { //Heuristic overcommit算法开始
0149                 unsigned long n;
0150
0151                 free = global_page_state(NR_FILE_PAGES); //pagecache汇总的页面数量
0152                 free += get_nr_swap_pages(); //free swap的页面数
0153
0154                 /*
0155                  * Any slabs which are created with the
0156                  * SLAB_RECLAIM_ACCOUNT flag claim to have contents
0157                  * which are reclaimable, under pressure.  The dentry
0158                  * cache and most inode caches should fall into this
0159                  */
0160                 free += global_page_state(NR_SLAB_RECLAIMABLE); //SLAB可回收的页面数
0161
0162                 /*
0163                  * Reserve some for root
0164                  */
0165                 if (!cap_sys_admin)
0166                         free -= sysctl_admin_reserve_kbytes >> (PAGE_SHIFT - 10); //给root用户保留的页面数
0167
0168                 if (free > pages)
0169                         return 0;
0170
0171                 /*
0172                  * nr_free_pages() is very expensive on large systems,
0173                  * only call if we're about to fail.
0174                  */
0175                 n = nr_free_pages(); //当前free memory页面数
0176
0177                 /*
0178                  * Leave reserved pages. The pages are not for anonymous pages.
0179                  */
0180                 if (n <= totalreserve_pages)
0181                         goto error;
0182                 else
0183                         n -= totalreserve_pages;
0184
0185                 /*
0186                  * Leave the last 3% for root
0187                  */
0188                 if (!cap_sys_admin)
0189                         n -= n / 32;
0190                 free += n;
0191
0192                 if (free > pages)
0193                         return 0;
0194
0195                 goto error;
0196         }
0197
0198         allowed = vm_commit_limit();
0199         /*
0200          * Reserve some for root
0201          */
0202         if (!cap_sys_admin)
0203                 allowed -= sysctl_admin_reserve_kbytes >> (PAGE_SHIFT - 10);
0204
0205         /* Don't let a single process grow too big:
0206            leave 3% of the size of this process for other processes */
0207         if (mm)
0208                 allowed -= mm->total_vm / 32;
0209
0210         if (percpu_counter_read_positive(&vm_committed_as) < allowed)
0211                 return 0;
0212 error:
0213         vm_unacct_memory(pages);
0214
0215         return -ENOMEM;
0216 }
-->

<!--
https://unix.stackexchange.com/questions/60474/atop-shows-red-line-vmcom-and-vmlim-what-does-it-mean
-->


## 参考

关于内存的 OverCommit 可以参考内核文档 [vm/overcommit-accounting]( {{ site.kernel_docs_url }}/Documentation/vm/overcommit-accounting ) 。

<!-- https://www.win.tue.nl/~aeb/linux/lk/lk-9.html -->



{% highlight text %}
{% endhighlight %}
