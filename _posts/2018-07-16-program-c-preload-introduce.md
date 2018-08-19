---
title: C Preload 简介
layout: post
comments: true
language: chinese
category: [program,misc]
keywords: c,preload
description: Linux C 中有一个很不错的特性，可以在不改变程序的前提下，修改动态库所调用的函数，也就是 Preload 功能。这里简单介绍其使用方法。
---

Linux C 中有一个很不错的特性，可以在不改变程序的前提下，修改动态库所调用的函数，也就是 Preload 功能。

这里简单介绍其使用方法。

<!-- more -->

## 简介

最常见的使用场景是不修改程序，而直接修改动态库中函数的实现，例如重新实现 `malloc()` 和 `free()` 函数。

## 示例

{% highlight c %}
#include <stdio.h>

int main(void)
{
	FILE *fd;

	printf("Calling the fopen() function...\n");

	fd = fopen("test.txt","r");
	if (fd == NULL) {
		printf("fopen() returned NULL\n");
		return 1;
	}
	printf("fopen() succeeded\n");

	return 0;
}
{% endhighlight %}

然后可以通过如下方式编译、执行。

{% highlight text %}
$ gcc foobar.c -o foobar

$ ./foobar
Calling the fopen() function...
fopen() succeeded
{% endhighlight %}

接着我们生成自己定义 `fopen()` 函数。

{% highlight c %}
#include <stdio.h>

FILE *fopen(const char *path, const char *mode)
{
	printf("Always failing fopen\n");
	return NULL;
}
{% endhighlight %}

然后，编译生成动态库。

{% highlight text %}
$ gcc -Wall -fPIC -shared -o libawrap.so awrap.c

$ LD_PRELOAD=./libawrap.so ./foobar
Calling the fopen() function...
Always failing fopen
fopen() returned NULL
{% endhighlight %}

也可以通过如下命令查看符号的查找过程。

{% highlight text %}
LD_DEBUG=symbols ./foobar
{% endhighlight %}

以及其真正依赖的库。

{% highlight text %}
$ ldd ./foobar
        linux-vdso.so.1 =>  (0x00007fffaffe7000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f0c22128000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f0c224f5000)
$ LD_PRELOAD=./libawrap.so ldd ./foobar
        linux-vdso.so.1 =>  (0x00007fff023fe000)
        ./libawrap.so (0x00007fbfa3e08000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fbfa3a3b000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fbfa400a000)
{% endhighlight %}

## 高级用法

假设我们仍然需要调用系统提供的函数，可以使用如下的方法。

{% highlight c %}
#define _GNU_SOURCE

#include <dlfcn.h>
#include <stdio.h>

FILE *fopen(const char *path, const char *mode)
{
	FILE *(*oopen)(const char *, const char*);

	printf("A wrapped fopen\n");
	oopen = dlsym(RTLD_NEXT, "fopen");
	if (oopen == NULL) {
		fprintf(stderr, "Failed to find fopen\n");
		return NULL;
	}

	return oopen(path, mode);
}
{% endhighlight %}

也就是通过 `dlsym()` 查找下个 `fopen()` 符号。

## 参考

详细可以参考 [Dynamic linker tricks: Using LD_PRELOAD to cheat, inject features and investigate programs](https://rafalcieslak.wordpress.com/2013/04/02/dynamic-linker-tricks-using-ld_preload-to-cheat-inject-features-and-investigate-programs/) 或者 [本地文档](/reference/programs/linux-c-preload.html) 。

{% highlight text %}
{% endhighlight %}
