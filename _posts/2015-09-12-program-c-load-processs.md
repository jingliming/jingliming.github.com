---
title: C 加载过程
layout: post
comments: true
language: chinese
category: [linux,program]
keywords: linux,c,加载过程
description:
---

利用动态库，可以节省磁盘、内存空间，而且可以提高程序运行效率；不过同时也导致调试比较困难，而且可能存在潜在的安全威胁。

这里主要讨论符号的动态链接过程，即程序在执行过程中，对其中包含的一些未确定地址的符号进行重定位的过程。

<!-- more -->

## 简介

`ld.so` (Dynamic Linker/Loader) 和 `ldd` 都会使用到 ELF 格式中的 `.dynstr` (dynamic linking string table) 字段，如果通过 `strip -R .dynstr hello` 命令将该字段删除，那么 `ldd` 就会报错。

## 加载过程

依赖动态库时，会在加载时根据可执行文件的地址和动态库的对应符号的地址推算出被调用函数的地址，这个过程被称为动态链接。

假设，现在使用的是 Position Independent Code, PIC 模型。

#### 1. 获取动态链接器

首先，读取 ELF 头部信息，解析出 `PT_INTERP` 信息，确定动态链接器的路径，可以通过 `readelf -l foobar` 查看，一般是 `/lib/ld-linux.so.2` 或者 `/lib64/ld-linux-x86-64.so.2` 。

#### 2. 加载动态库

关于加载的详细顺序可以查看 `man ld` 中 rpath-link 的介绍，一般顺序为：

1. 链接时 `-rpath-link` 参数指定路径，只用于链接时使用，编译时通过 `-Wl,rpath-link=` 指定；
2. 链接时通过 `-rpath` 参数指定路径，除了用于链接时使用，还会在运行时使用，编译时可利用 `-Wl,rpath=` 指定，会生成 `DT_RPATH` 或者 `DT_RUNPATH` 定义，可以通过 `readelf -d main | grep -E (RPATH|RUNPATH)` 查看；
3. 查找 `DT_RUNPATH` 或者 `DT_RPATH` 指定的路径，如果前者存在则忽略后者；
3. 依次查看 `LD_RUN_PATH` 和 `LD_LIBRARY_PATH` 环境变量指定路径；
4. 查找默认路径，一般是 `/lib` 和 `/usr/lib` ，然后是 `/etc/ld.so.conf` 文件中的配置。

另外，需要加载哪些库通过 `DT_NEEDED` 字段来获取，每条对应了一个动态库，可以通过 `readelf -d main | grep NEEDED` 查看。

### 示例程序

利用如下的示例程序。

{% highlight c %}
/* filename: foobar.c */
#define _GNU_SOURCE
#include <stdio.h>
#include <dlfcn.h>

int foobar(void)
{
    Dl_info dl_info;
    dladdr((void*)foobar, &dl_info);
    fprintf(stdout, "load .so at: %s\n", dl_info.dli_fname);
    return 0;
}
{% endhighlight %}

{% highlight c %}
/* filename: main.c */
int foobar(void);
int main(int argc, char **argv)
{
  foobar();
  return 0;
}
{% endhighlight %}

{% highlight makefile %}
# filename: Makefile
all:
    gcc --shared -fPIC foobar.c -o libfoobar.so -ldl
    gcc main.c -o main -Wl,-rpath-link=/foobar -ldl -lfoobar -L./
    readelf -d main | grep -E (RPATH|RUNPATH)
{% endhighlight %}

然后可以通过依次设置如上的加载路径进行测试。**注意**，在对 `/etc/ld.so.conf` 文件设置后需要通过 `ldconfig` 更新 cache 才会生效。

另外，推荐使用 `DT_RUNPATH` 而非 `DT_RPATH` ，此时，在编译时需要用到 `--enable-new-dtags` 参数。

<!--
通过C语言直接读取rpath参数
#include <stdio.h>
#include <elf.h>
#include <link.h>

int main()
{
  const ElfW(Dyn) *dyn = _DYNAMIC;
  const ElfW(Dyn) *rpath = NULL;
  const char *strtab = NULL;
  for (; dyn->d_tag != DT_NULL; ++dyn) {
    if (dyn->d_tag == DT_RPATH) {
      rpath = dyn;
    } else if (dyn->d_tag == DT_STRTAB) {
      strtab = (const char *)dyn->d_un.d_val;
    }
  }

  if (strtab != NULL && rpath != NULL) {
    printf("RPATH: %s\n", strtab + rpath->d_un.d_val);
  }
  return 0;
}
-->

### 版本管理

不同版本的动态库可能会不兼容，那么如果程序在编译时指定动态库是某个低版本，运行是用的一个高版本，可能会导致无法运行。

假设有如下的示例：

{% highlight c %}
/* filename:hello.c */
#include <stdio.h>
void hello(const char* name)
{
    printf("hello %s!\n", name);
}
{% endhighlight %}

{% highlight c %}
/* filename:hello.h */
void hello(const char* name);
{% endhighlight %}

{% highlight makefile %}
# filename: Makefile
all:
    gcc hello.c -fPIC -shared -Wl,-soname,libhello.so.0 -o libhello.so.0.0.1
{% endhighlight %}

需要注意是，参数 `-Wl,soname` 中间没有空格，`-Wl` 选项用来告诉编译器将后面的参数传递给链接器，而 `-soname` 则指定了动态库的 `soname`。运行后在当前目录下会生成一个 `libhello.so.0.0.1` 文件，当运行 `ldconfig -n .` 命令时，当前目录会多一个符号连接。

这个软链接是根据编译生成 `libhello.so.0.0.1` 时指定的 `-soname` 生成的，会保存到编译生成的文件中，可以通过 `readelf -d foobar` 查看依赖的库。

所以关键就是这个 soname，它相当于一个中间者，当我们的动态库只是升级一个小版本时，可以让它的 soname 相同，而可执行程序只认 soname 指定的动态库，这样依赖这个动态库的可执行程序不需重新编译就能使用新版动态库的特性。

<!-- http://littlewhite.us/archives/301 -->

## 常用程序

### readelf

用于读取 ELF 格式文件，包括可执行程序和动态库。

{% highlight text %}
常用参数：
  -a --all
    等价于-h -l -S -s -r -d -V -A -I
  -h --file-header
    文件头信息；
  -l --program-headers
    程序的头部信息；
  -S --section-headers
    各个段的头部信息；
  -e --headers
    全部头信息，等价于-h -l -S；

示例用法：
----- 读取dynstr段，包含了很多需要加载的符号，每个动态库后跟着需要加载函数
$ readelf -p .dynstr hello
----- 查看是否含有调试信息
$ readelf -S hello | grep debug
{% endhighlight %}

<!--
readelf  -S hello
readelf -d hello

  --sections
An alias for –section-headers
-s –syms 符号表 Display the symbol table
--symbols
An alias for –syms
-n –notes 内核注释 Display the core notes (if present)
-r –relocs 重定位 Display the relocations (if present)
-u –unwind Display the unwind info (if present)
-d --dynamic
  显示动态段的内容；
-V –version-info 版本 Display the version sections (if present)
-A –arch-specific CPU构架 Display architecture specific information (if any).
-D –use-dynamic 动态段 Use the dynamic section info when displaying symbols
-x –hex-dump=<number> 显示 段内内容Dump the contents of section <number>
-w[liaprmfFso] or
-I –histogram Display histogram of bucket list lengths
-W –wide 宽行输出 Allow output width to exceed 80 characters
-H –help Display this information
-v –version Display the version number of readelf
-->

## 参考

关于动态库的加载过程，可以参考 [动态符号链接的细节](https://github.com/tinyclub/open-c-book/blob/master/zh/chapters/02-chapter4.markdown)。

<!--
reference/linux/linux-c-dynamic-loading.markdown

ELF文件的加载和动态链接过程
http://jzhihui.iteye.com/blog/1447570
linux是如何加载ELF格式的文件的
http://cn.windyland.me/2014/10/24/how-to-load-a-elf-file/
 -->

{% highlight text %}
{% endhighlight %}
