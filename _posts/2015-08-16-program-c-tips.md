---
title: C 语言的奇技淫巧
layout: post
comments: true
language: chinese
category: [linux,program]
keywords: linux,c,cpp,c++,program
description: 整理下 C 语言中常用的技巧。变长数组。
---

整理下 C 语言中常用的技巧。

<!-- more -->

## 变长数组

实际编程中，经常会使用变长数组，但是 C 语言并不支持变长的数组，可以使用结构体实现。

类似如下的结构体，其中 value 成员变量不占用内存空间，也可以使用 ```char value[]``` ，但是不要使用 ``` char *value```，该变量会占用指针对应的空间。

常见的操作示例如下。

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct foobar {
  int len;
  char value[0];
} foobar_t;

int main(int argc, char *argv[])
{
    // 两者占用的存储空间相同，也就是value不占用空间
    printf("%li %li\n", sizeof(int), sizeof(foobar_t));

    // 初始化
    int *values = (int *)malloc(10*sizeof(int)), i, j, *ptr;
    for (i = 0; i < 10; i++)
      values[i] = 10*i;
    for (i = 0; i < 10; i++)
      printf(" %i", values[i]);
    printf("\n");

    // 针对单个结构体的操作
    foobar_t *buff = (foobar_t *)malloc(sizeof(foobar_t) + 10*sizeof(int));
    buff->len = 10;
    memcpy(buff->value, values, 10*sizeof(int));
    ptr = (int *)buff->value;

    printf("length: %i, vlaues:", buff->len);
    for (i = 0; i < 10; i++)
      printf(" %i", ptr[i]);
    printf("\n");
    free(buff);

    // 针对数组的操作
    #define FOOBAR_T_SIZE(elements) (sizeof(foobar_t) + sizeof(int) * (elements))
    foobar_t **buf = (foobar_t **)malloc(6*FOOBAR_T_SIZE(10));
    foobar_t *ptr_buf;
    for (i = 0; i < 6; i++) {
      ptr_buf = (foobar_t*)((char *)buf + i*FOOBAR_T_SIZE(10));
      ptr_buf->len = i;
      memcpy(ptr_buf->value, values, 10*sizeof(int));

      ptr = (int *)ptr_buf->value;
      printf("length: %i, vlaues:", ptr_buf->len);
      for (j = 0; j < 10; j++)
        printf(" %i", ptr[j]);
      printf("\n");
    }
    free(values);
    free(buf);

    return 0;
}
{% endhighlight %}

## Clang

![clang logo]({{ site.url }}/images/programs/clang_logo.png "clang logo"){: .pull-center }

Clang 是一个 C++ 编写，基于 LLVM 的 C/C++、Objective-C 语言的轻量级编译器，在 2013.04 开始，已经全面支持 C++11 标准。

### pragma

```#pragma``` 宏定义在本质上是声明，常用的功能就是注释，尤其是给 Code 分段注释；另外，还支持处理编译器警告。

{% highlight c %}
#pragma clang diagnostic push

//----- 方法弃用告警
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
//----- 不兼容指针类型
#pragma clang diagnostic ignored "-Wincompatible-pointer-types"
//----- 未使用变量
#pragma clang diagnostic ignored "-Wunused-variable"
//----- 无返回值
#pragma clang diagnostic ignored "-Wmissing-noreturn"
//... ...

#pragma clang diagnostic pop
{% endhighlight %}



{% highlight text %}
{% endhighlight %}
