---
title: C 语言发布流程
layout: post
comments: true
language: chinese
category: [linux,program]
keywords: linux,c,cpp,c++,program
description: 整理下 C 语言中调试、发布的流程。
---

整理下 C 语言中调试、发布的流程。

<!-- more -->



## 编码规范

这里直接使用的是 Linux 的编码规范。

{% highlight text %}
$ scripts/checkpatch.pl --no-tree -f log.c
{% endhighlight %}

<!---
谈谈Linux内核驱动的coding style
http://www.cnblogs.com/wwang/archive/2011/02/24/1960283.html

采用的Linux内核的编码规范
https://www.kernel.org/doc/html/v4.15/translations/zh_CN/coding-style.html
-->

#### FAQ

{% highlight text %}
ERROR: need consistent spacing around '*' (ctx:WxV)
#88: FILE: include/linux/pmem.h:167:
+static inline void flush_cache_pmem(void __pmem *addr, size_t size)
{% endhighlight %}

实际上是无法识别 `__pmem` 引起的，此时需要在脚本的 `our $Sparse` 中添加 `__pmem` 。

<!--
## Linux 代码正式发布以及问题排查

## 0. 开发测试

## 1. RPM包安装

## 2. DebugInfo

http://www.unknownroad.com/rtfm/gdbtut/gdbstack.html
https://sourceware.org/gdb/onlinedocs/gdb/Backtrace.html
https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf
http://www.brendangregg.com/blog/2016-08-09/gdb-example-ncurses.html
-->

### BackTrace

{% highlight c %}
#include <stdio.h>
#include <signal.h>
#include <errno.h>
#include <string.h>
#include <stdlib.h>
#include <execinfo.h>

void log_backtrace(int signal)
{
        (void) signal;
        int i, frames;
        void *callstack[128];
        char **strs;

        frames = backtrace(callstack, 128);
        strs = backtrace_symbols(callstack, frames);
        if (strs == NULL) {
                printf("ERROR: backtrace_symbols failed, %s", strerror(errno));
                exit(1);
        }

        for (i = 0; i < frames; ++i)
                printf(" [%d]: %s\n", i, strs[i]);
        free(strs);
        exit(1);
}

int bar(void)
{
        int *addr = NULL;

        *addr = 3;
        return 0;
}

int foo(void)
{
        bar();
        return 0;
}


int main(void)
{
        signal(SIGSEGV, log_backtrace);
        foo();
        return 0;
}
{% endhighlight %}

<!--
https://www.linuxjournal.com/article/6391
https://www.mawenbao.com/research/glibc-backtrace-parsing.html
http://silencewt.github.io/2015/05/11/Segmentation-Fault%E9%94%99%E8%AF%AF%E5%8E%9F%E5%9B%A0%E6%80%BB%E7%BB%93/
-->

### 符号表

直接通过 `gcc -o test main.c` 进行编译时，打印的信息不含具体是那个函数，都是 16 进制的地址，此时可以通过反汇编获取，不过比较麻烦。

可以通过 `objdump -d test > test.s` 命令直接反汇编。

此时可以在编译时添加 `-rdynamic` 参数，其作用是：

{% highlight text %}
-rdynamic
    Pass the flag -export-dynamic to the ELF linker, on targets that support it. This
instructs the linker to add all symbols, not only used ones, to the dynamic symbol
table. This option is needed for some uses of "dlopen" or to allow obtaining backtraces
from within a program.
{% endhighlight %}

另外，对于 C++ 其符号名是通过 mangle 之后的，为了获取具体的函数信息，需要 demangle 处理，例如：

{% highlight text %}
c++filt  "_Z16print_stacktracev"
{% endhighlight %}

当然，此时是无法查看函数是具体的哪一行的，可以利用 `addr2line` 查看，此时在编译时需要添加 `-g` 选项，例如：

{% highlight text %}
addr2line -Cifp -a 0x400a29 -e test
{% endhighlight %}

`-f` 用来打印函数名，`-C` 同时 demangle 处理，`-i` 同时处理 inline 函数。

<!--


1. 不包含任何的行信息，无法确定具体那个函数出问题。
2. 无法确定函数的入参以及本地的变量值；
3. 无法确定打印的栈函数是否为静态。

They contain no line number information, so you don’t know where in
a function something happened
You cannot see the values of arguments and local variables
You cannot trust the function names given in the backtrace, since
the debugger doesn’t know about static functions.
















disas 0x41dde0,0x41ef00
disas 0x41e500,0x41efff



对程序进行汇编级调试。


----- (run)重新开始运行
(gdb) r
----- (break)针对地址设置断点
(gdb) b *0x0804ce2b

单步步过
(gdb) ni    (next instruction)
单步步入
(gdb) si    ( step instruction )
继续执行
( gdb )c

执行到返回
(gdb) finish
-->


{% highlight text %}
{% endhighlight %}
