---
title: C 编译链接
layout: post
comments: true
language: chinese
category: [program,misc]
keywords: c,编译链接
description: 详细介绍下与 C 语言相关的概念。
---

详细介绍下与 C 语言相关的概念。

<!-- more -->

## 目标文件

目标文件是编译器编译完后，还没有进行链接的文件，通常采用和可执行文件相同的存储格式，如 Windows 平台下的 `Portable Executable, PE`，Linux 平台下的 `Executable Linkale Format, ELF`，它们都是 `Common File Format, COFF` 的变种。

除可执行文件，动态链接库 `(Dynamic Linking Library)` 和静态链接库 `(Statci Linking Library)` 也是按照可执行文件进行存储。

对于文件的类型可以通过 file 命令进行查看，常见的类型包括了：

* 可重定位文件(Relocatable File)<br>主要包含了代码和数据，主要用来链接成可执行文件或共享目标文件，如 `.o` 文件。
* 可执行文件(Executable File)<br>主要是可以直接执行的程序，如 `/bin/bash` 。
* 共享目标文件(Shared Object File)<br>包含了代码和数据，常见的有动态和静态链接库，如 `/lib64/libc-2.17.so` 。
* 核心转储文件(Core Dump File)<br>进程意外终止时，系统将该进程的地址空间的内容及终止时的一些其他信息转储到该文件中。

### 示例程序

目标文件通过段 (Segment) 进行存储，在 Windows 中可以通过 `Process Explorer` 查看进程相关信息，Linux 可以通过 `objdump` 查看。主要的段包括了 `.text` `.data` `.bss(Block Started by Symbol)`，当然还有一些其它段，如 `.rodata` `.comment` 等。

{% highlight c %}
int printf(const char* format, ...);

int global_init_var = 84;
int global_uninit_var;

void func1(int i)
{
    printf("%d\n", i);
}

int main(void)
{
    static int static_var = 85;
    static int static_var2;

    int a = 1;
    int b;

    func1(static_var + static_var2 + a + b);

    return a;
}
{% endhighlight %}

然后，可以通过 `gcc -c section.c` 编译，编译完上述的文件之后，可以通过 `objdump -h` 查看头部信息，也可以通过 `-x` 参数查看更详细的信息。

比较重要的是 `File off` 和 `Size` 信息，一般头部信息的大小为 `0x34` ，因此第一个段的地址就会从 `0x34` 开始 (地址从 0 开始计数)，另外，由于需要 4bytes 对齐，因此会从 `54(0x36)` 开始。也可以通过 size 查看，采用的是十进制，最后会用十进制和十六进制表示总的大小。

数据段 `.data` 用来保存已经初始化了的全局变量和局部静态变量，如上述的 `global_init_var` 和 `static_var` 。

只读数据段 `.rodata` ，主要用于保存常量，如 `printf()` 中的字符串和 `const` 类型的变量，该段在加载时也会将其设置为只读。

`BSS` 段保存了未初始化的全局变量和局部静态变量，如上述 `global_uninit_var` 和 `static_var2` 。

<!--
正常应该是 8 字节，但是查看时只有 4 字节，通过符号表(Symbol Table)可以看到，只有 static_var2 保存在 .bss 段，而 global_uninit_var 未存放在任何段，只是一个未定义的 COMMON 符号。这与不同的语言和编译器实现有关，有些编译器会将全局的为初始化变量存放在目标文件 .bss 段，有些则不存放，只是预留一个未定义的全局变量符号，等到最终链接成可执行文件时再在 .bss 段分配空间。
-->

`.text` 为代码段，`.data` 保存含初始值的变量，`.bss` 只保存了变量的符号。


### 添加一个段

将以个二进制文件，如图片、MP3 音乐等作为目标文件的一个段。如下所示，此时可以直接声明 `_binary_example_png_start` 和 `_binary_example_png_end` 并使用。

{% highlight text %}
$ objcopy -I binary -O elf32-i386 -B i386 example.png image.o
$ objdump -ht image.o

image.o:     file format elf32-i386

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .data         000293d6  00000000  00000000  00000034  2**0
                  CONTENTS, ALLOC, LOAD, DATA
SYMBOL TABLE:
00000000 l    d  .data	00000000 .data
00000000 g       .data	00000000 _binary_example_png_start
000293d6 g       .data	00000000 _binary_example_png_end
000293d6 g       *ABS*	00000000 _binary_example_png_size
{% endhighlight %}

如果在编译时想将某个函数或者变量放置在一个段里，可以通过如下的方式进行。

{% highlight c %}
__attribute__((section("FOO"))) int global = 42;
__attribute__((section("BAR"))) void foo() { }
{% endhighlight %}

### 运行库

在 `main()` 运行之前通常会先执行一段代码，运行这些代码的函数称为 **入口函数** 或 **入口点** ，大致的步骤如下：

* 操作系统创建进程后，把控制权交给程序入口，这个入口往往是运行库中的某个入口函数。
* 入口函数对运行库和程序运行环境进行初始化，包括堆、I/O、线程、全局变量构造等。
* 入口函数在完成初始化之后，调用 main 函数，正式开始执行程序主体部分。
* `main()` 执行完后，返回到入口函数，入口函数进行清理工作，包括全局变量析构、堆销毁、关闭 IO 等，然后进行系统调用结束进程。



{% highlight text %}
{% endhighlight %}
