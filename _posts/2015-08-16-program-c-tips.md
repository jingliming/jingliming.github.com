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


## 指针

指针或许是 C 语言中最复杂的东西了。

### 指针常量 VS. 常量指针

前面是一个修饰词，后面的是中心词。

#### 常量指针

**常量指针** 首先是一个指针，指向的是常量，即指向常量的指针；可以通过如下的方式定义：

{% highlight c %}
const int a = 7;
const int *p = &a;
{% endhighlight %}

对于常量，我们不能对其内容进行修改；指针的内容本身是一个地址，通过常量指针指向一个常量，为的就是防止我们写程序过程中对指针误操作出现了修改常量这样的错误，如果我们修改常量指针的所指向的空间的时候，编译系统就会提示我们出错信息。

在 C 语言中，通常定义的字符串会返回一个常量指针，因此字符串不能赋值给字符数组，只能赋值到指针。

总结一下，<font color="red">常量指针就是指向常量的指针，指针所指向的地址的内容是不可修改的，指针本身的内容是可以修改的</font> 。


#### 指针常量

**指针常量**  首先是一个常量，再才是一个指针；可以通过如下的方式定义：

{% highlight c %}
int a = 7;
int * const p = &a; // OR int const *p = &a;
{% endhighlight %}

常量的性质是不能修改，指针的内容实际是一个地址，那么指针常量就是内容不能修改的常量，即内容不能修改的指针，指针的内容是什么呀？指针的内容是地址，所以，说到底，就是不能修改这个指针所指向的地址，一开始初始化，指向哪儿，它就只能指向哪儿了，不能指向其他的地方了，就像一个数组的数组名一样，是一个固定的指针，不能对它移动操作。

它只是不能修改它指向的地方，但这个指向的地方里的内容是可以替换的，这和上面说的常量指针是完全不同的概念。

作一下总结，<font color="red">指针常量就是是指针的常量，它是不可改变地址的指针，但是可以对它所指向的内容进行修改</font> 。

源码可以参考 [github const_pointer.c]({{ site.example_repository }}/c_cpp/c/const_pointer.c) 。

### 与一维数组

假设有如下数组，

{% highlight c %}
int Array[] = {1, 2, 3, 4};
int *ptr = Array;
{% endhighlight %}

其中 Array 为指针常量，而 ptr 为指针变量，且 ```ptr = &Array[0]```，那么如下的操作相同 ```ptr[i] <=> *(ptr+i)``` 以及 ```Array[i] <=> *(Array + i)``` 。

如下，简单介绍下常见操作。

#### *ptr++

由于 ```*``` 和 ```++``` 优先级相同，结合性为由右至左，即 ```*ptr++``` 等价于 ```*(ptr++)``` ，由于 ```++``` 为后继加，所以当得到 ```*ptr``` 后再处理 ```++```；所以 ```*ptr++``` 等于 1，进行此项操作后 ```*ptr``` 等于 2。

执行的步骤为 1) ```++``` 操作符产生 ptr 的一份拷贝；2) ```++``` 操作符增加 ptr 的值；3) 在 ptr 上执行间接访问操作。

#### ++*ptr

利用优先级和结合性可得，```++*ptr``` 等价于 ```++(*ptr)``` ，此时 ```Array[0]``` 为 2，返回 2 。

#### *ptr++

利用优先级和结合性可得，```*ptr++``` 等价于 ```*(ptr++)``` ，返回 1，ptr 值加 1 。

## 大小端

当数据类型大于一个字节时，其所占用的字节在内存中的顺序存在两种模式：小端模式 (little endian) 和大端模式 (big endian)，其中 MSB(Most Significant Bit) 最高有效位，LSB(Least Significant Bit) 最低有效位.

{% highlight text %}
小端模式
MSB                             LSB
+-------------------------------+
|   1   |   2   |   3   |   4   | int 0x1234
+-------------------------------+
  0x03    0x02    0x01    0x00   Address

大端模式
MSB                             LSB
+-------------------------------+
|   1   |   2   |   3   |   4   | int 0x1234
+-------------------------------+
  0x00    0x01    0x02    0x03   Address
{% endhighlight %}

如下是一个测试程序。

{% highlight c %}
#include <stdio.h>

void main(void)
{
   int test = 0x41424344;
   char* pAddress = (char*)&test;

#ifdef DEBUG
   printf("int  Address:%x Value:%x\n", (unsigned int)&test, test);
   printf("\n------------------------------------\n");

   int j;
   for(j=0; j<=3; j++){
      printf("char Address:%x Value:%c\n", (unsigned int)pAddress, *pAddress);
      pAddress++;
   }
   printf("------------------------------------\n\n");
   pAddress = (char*)&test;
#endif
   if(*pAddress == 0x44)
      printf("Little-Endian\n");
   else if(*pAddress == 0x41)
      printf("Big-Endian\n");
   else
      printf("Something Error!\n");
}
{% endhighlight %}

如果采用大端模式，则在向某一个函数通过向下类型装换来传递参数时可能会出错。如一个变量为 ```int i=1;``` 经过函数 ```void foo(short *j);``` 的调用，即 ```foo((short*)&i);```，在 foo() 中将 i 修改为 3 则最后得到的 i 为 0x301 。

大端模式规定 MSB 在存储时放在低地址，在传输时 MSB 放在流的开始；小段模式反之。

## 变参传递

头文件 `stdarg.h` 中对相关的宏进行了定义，其基本内容如下所示：

{% highlight c %}
typedef char * va_list;

#define _INTSIZEOF(n)       ((sizeof(n) + sizeof(int) - 1) & ~(sizeof(int) - 1))
#define va_start(arg_ptr,v) (arg_ptr = (va_list)&v + _INTSIZEOF(v))
#define va_arg(arg_ptr,t)   (*(t *)((arg_ptr += _INTSIZEOF(t)) - _INTSIZEOF(t)))
#define va_end(arg_ptr)     (arg_ptr = (va_list)0)
{% endhighlight %}

<!--
_INTSIZEOF(n)：我们可以经过测试得到，如果在上面的测试程序中用一个double类型（长度为8byte）和一个 long double类型（长度为10byte）做可变参数。我们发现double类型占了8byte,而long double占了12byte，即都是4的整数倍。这里是因为对齐，即对Intel80x86 机器来说就是要求每个变量的地址都是sizeof(int)的倍数。char类型的参数只占了1byte，但是它后面的参数因为对齐的关系只能跳过3byte存储，而那3byte也就浪费掉了。<br><br>

<font color=blue><strong>va_start(arg_ptr,v)</strong></font>：使参数列表指针arg_ptr指向函数参数列表中的第一个可选参数。v是位于第一个可选参数之前的固定参数，如果有一函数的声明是void va_test(char a, char b, char c, …)，则它的固定参数依次是a,b,c，最后一个固定参数v为c，因此就是va_start(arg_ptr, c)。<br><br>

<font color=blue><strong>va_arg(arg_ptr, t)</strong></font>：返回参数列表中指针arg_ptr所指的参数，返回类型为t，并使指针arg_ptr指向参数列表中下一个参数。首先计算(arg_ptr += _INTSIZEOF(t))，此时arg_ptr将指向下一个参数；((arg_ptr += _INTSIZEOF(t)) - _INTSIZEOF(t))等价于arg_ptr，通过(t *)将arg_ptr转换为t类型的指针，并通过*取其值。<br><br>

<font color=blue><strong>va_end(arg_ptr)</strong></font>：使ap不再指向堆栈,而是跟NULL一样。<br><br>
-->

示例如下图所示：

{% highlight c %}
#include <stdarg.h>
#include <stdio.h>

int max( int num,...)
{
    int m = -0x7FFFFFFF; /* 32系统中最小的整数 */
    int i = 0, t = 0;
    va_list ap;
    va_start( ap, num);
    for( i = 0; i < num; i++) {
         t = va_arg( ap, int);
         if( t > m)
            m = t;
    }
    va_end(ap);
    return m;
}

int main(int argc,char *argv[])
{
  int n, m;

  n = max( 5, 5, 6, 3, 8, 5);
  m = max( 7, 5, 1, 9, 8, 5, 7, 0);

  printf("%d\t%d\n",n,m);

  return 0;
}
{% endhighlight %}

## 调试

当调试时定义 DEBUG 输出信息，通常有如下的几种方式。

{% highlight c %}
// 常用格式
#ifdef DEBUG
   #define debug(fmt, args...) printf(fmt, ##args) // OR
   #define debug(fmt, ...) printf("sth: " fmt, ## __VA_ARGS__);
#else
   #define debug(fmt,args...)
#endif

// 输出文件名、函数名、行数
#ifdef DEBUG
   #define debug(fmt, args...) printf("%s, %s, %d: " fmt , __FILE__, __FUNCTION__, __LINE__, ##args)
#else
   #define debug(fmt, args...)
#endif

// 输出信息含有彩色
#ifdef DEBUG
   #define debug(fmt,args...)    \
      do{                        \
         printf("\033[32;40m");  \
         printf(fmt, ##args);    \
         printf("\033[0m");      \
      } while(0);
#else
   #define debug(fmt,args...)
#endif
{% endhighlight %}

## 对齐操作

为了性能上的考虑，很多的平台都会从某一个特定的地址开始读取数据，比如偶地址。

数据结构中的数据变量都是按照定义的顺序来定义，第一个变量的地址等同于数据结构的地址，结构体中的成员也要对齐，最后结构体也同样需要对齐。对齐是指 **起始地址对齐**，其中对齐规则如下:

1. 数据成员对齐规则<br>结构体(struct)或联合(union)的数据成员，第一个数据成员放在offset为0的地方，以后每个数据成员的对齐按照`#pragma pack`指定的数值n和这个数据成员自身长度中，比较小的那个进行。

2. 结构体(或联合)的整体对齐规则<br>在数据成员完成各自对齐之后，结构体(或联合)本身也要进行对齐，对齐将按照`#pragma pack`指定的数值n和结构体(或联合)最大数据成员长度中，比较小的那个进行。

3. 当`#pragma pack`的n值等于或超过所有数据成员长度的时候，这个n值的大小将不生任何效果。

现举例如下：

#### 1字节对齐

{% highlight c %}
#pragma pack(1)
struct test_t {
    int    a;   // 长度4 > 1 按1对齐；起始offset=0 0%1=0；存放位置区间[0,3]
    char   b;   // 长度1 = 1 按1对齐；起始offset=4 4%1=0；存放位置区间[4]
    short  c;   // 长度2 > 1 按1对齐；起始offset=5 5%1=0；存放位置区间[5,6]
    char   d;   // 长度1 = 1 按1对齐；起始offset=7 7%1=0；存放位置区间[7]
};
#pragma pack() 	// 取消对齐
{% endhighlight %}

{% highlight text %}
输出结果 sizeof(struct test_t) = 8
整体对齐系数 min((max(int,short,char), 1) = 1
整体大小(size)=$(成员总大小8) 按 $(整体对齐系数) 圆整 = 8
{% endhighlight %}

#### 2字节对齐

{% highlight c %}
#pragma pack(2)
struct test_t {
    int    a;   // 长度4 > 2 按2对齐；起始offset=0 0%2=0；存放位置区间[0,3]
    char   b;   // 长度1 < 2 按1对齐；起始offset=4 4%1=0；存放位置区间[4]
    short  c;   // 长度2 = 2 按2对齐；起始offset=6 6%2=0；存放位置区间[6,7]
    char   d;   // 长度1 < 2 按1对齐；起始offset=8 8%1=0；存放位置区间[8]
};
#pragma pack() 	// 取消对齐
{% endhighlight %}

{% highlight text %}
输出结果 sizeof(struct test_t) = 10
整体对齐系数 = min((max(int,short,char), 2) = 2
整体大小(size)=$(成员总大小9) 按 $(整体对齐系数) 圆整 = 10
{% endhighlight %}

#### 4字节对齐

{% highlight c %}
#pragma pack(4)
struct test_t {
    int    a;   // 长度4 = 4 按4对齐；起始offset=0 0%4=0；存放位置区间[0,3]
    char   b;   // 长度1 < 4 按1对齐；起始offset=4 4%1=0；存放位置区间[4]
    short  c;   // 长度2 < 4 按2对齐；起始offset=6 6%2=0；存放位置区间[6,7]
    char   d;   // 长度1 < 4 按1对齐；起始offset=8 8%1=0；存放位置区间[8]
};
#pragma pack() // 取消对齐
{% endhighlight %}

{% highlight text %}
输出结果 sizeof(struct test_t) = 12
整体对齐系数 = min((max(int,short,char), 4) = 4
整体大小(size)=$(成员总大小9) 按 $(整体对齐系数) 圆整 = 12
{% endhighlight %}

#### 8字节对齐

{% highlight c %}
#pragma pack(8)
struct test_t {
    int    a;   // 长度4 < 8 按4对齐；起始offset=0 0%4=0；存放位置区间[0,3]
    char   b;   // 长度1 < 8 按1对齐；起始offset=4 4%1=0；存放位置区间[4]
    short  c;   // 长度2 < 8 按2对齐；起始offset=6 6%2=0；存放位置区间[6,7]
    char   d;   // 长度1 < 8 按1对齐；起始offset=8 8%1=0；存放位置区间[8]
};
#pragma pack() 	// 取消对齐
{% endhighlight %}

{% highlight text %}
输出结果 sizeof(struct test_t) = 12
整体对齐系数 = min((max(int,short,char), 8) = 4
整体大小(size)=$(成员总大小9) 按 $(整体对齐系数) 圆整 = 12
{% endhighlight %}

### \_\_attribute((aligned(n)))

另一种方式是 `__attribute((aligned(n)))` 让所作用的结构成员对齐在 `n` 字节自然边界上，如果结构中有成员长度大于 `n `，则按照最大的成员的长度对齐。

示例如下：

{% highlight text %}
struct test_t {
    int    a;
    char   b;
    short  c;
    char   d;
} __attribute((aligned(n)));
{% endhighlight %}

`__attribute__((packed))` 取消编译过程中的优化对齐，按照实际占用字节数进行对齐。


<!--
采用宏进行对齐操作，计算a以size为倍数的上下界数，
#define alignment_down(a, size)  (a & (~(size - 1)))
#define alignment_up(a, size)    ((a + size - 1) & (~(size - 1)))
-->

详见参考程序 [github align.c]({{ site.example_repository }}/c_cpp/c/align.c) 。

## 参数解析

`getopt()` 是采用缓冲机制，因此对于多线程编程是 **不安全** 的。

{% highlight text %}
#include <unistd.h>
// 选项的参数指针
extern char *optarg;

// 初值为1, 下次调用时，从optind位置开始检测，通过agrv[optind]可以得到下一个参数，从而可以自行检测
extern int optind;

// 对于不能识别的参数将输出错误信息，可以将opterr设置为0，从而阻止向stderr输出错误信息
extern int opterr;

// 如果选项字符不再optstring中则返回':'或'?'，并将字符保存在optopt中
extern int optopt;

int getopt(int argc, char * const argv[],const char *optstring);
描述：
  该函数处理"-"起始的参数，有些平台可能支持"--"
参数：
  argc、argv分别为main()传入的参数；其中optstring可以有如下的选项:
    单个字符，表示选项。
    单个字符后接一个冒号，表示该选项后必须跟一个参数，参数紧跟在选项后或者以空格隔开，该参数的指针赋给optarg。
    单个字符后跟两个冒号，表示该选项后必须跟一个参数，参数必须紧跟在选项后不能以空格隔开，否则optarg指向为NULL，
        该参数的指针赋给optarg，这个特性是GNU的扩展。
{% endhighlight %}

如 `optstring="ab:c::d::"` ，命令行为 `getopt.exe -a -b host -ckeke -d haha`，在这个命令行参数中，`-a` `-b` 和 `-c` 是选项元素，去掉 `'-'`，a b c 就是选项。

host 是 b 的参数，keke 是 c 的参数，但 haha 并不是 d 的参数，因为它们中间有空格隔开。

**注意**：如果 optstring 中的字符串以 `'+'` 加号开头或者环境变量 `POSIXLY_CORRE` 被设置，那么一遇到不包含选项的命令行参数，getopt 就会停止，返回 -1；命令参数中的 `"--"` 用来强制终止扫描。

默认情况下 getopt 会重新排列命令行参数的顺序，所以到最后所有不包含选项的命令行参数都排到最后，如 `getopt -a ima -b host -ckeke -d haha`，都最后命令行参数的顺序是 `-a -b host -ckeke -d ima haha` 。

如果检测到设置的参数项，则返回参数项；如果检测完成则返回 -1；如果有不能识别的参数则将该参数保存在 optopt 中，输出错误信息到 stderr，如果 optstring 以 `':'` 开头则返回 `':'` 否则返回 `'?'`。

源码可以参考 [github getopt.c]({{ site.example_repository }}/c_cpp/c/getopt.c) 。

### 长选项

{% highlight text %}
#include <getopt.h>
int getopt_long(int argc, char * const argv[],
    const char *optstring, const struct option *longopts, int *longindex);
int getopt_long_only(int argc, char * const argv[],
    const char *optstring, const struct option *longopts, int *longindex);

描述：
  该函数与getopt函数类似，不过可以接收长选项(通常以"--"开头)，如果只接收长选项则optstring应该设置为""，而非NULL。
{% endhighlight %}

<!--
<font color=blue size=3><strong>参数：</strong></font><br>
	前3个参数与getopt类似，struct option定义在&lt;getopt.h&gt;中，<br>
<pre>
struct option {
    const char *name;   //name表示的是长参数名
    int has_arg; // 有3个值，
                 // no_argument(或者是0)，表示该参数后面不跟参数值
                 // required_argument(或者是1),表示该参数后面一定要跟个参数值
                 // optional_argument(或者是2),表示该参数后面可以跟，也可以不跟参数值
    int *flag; // 用来决定，getopt_long()的返回值到底是什么。如果flag是null，
               // 则函数会返回与该项option匹配的val值；否则如果找到参数则返回0，
               // flag指向该option的val，如果没有找到则保持不变。
    int val; // 和flag联合决定返回值
}</pre>
	如果longindex没有设置为空则，longindex指向longopts<br><br>
-->

源码可以参考 [github getopt_long.c]({{ site.example_repository }}/c_cpp/c/getopt_long.c) 。

{% highlight text %}
$ getopt-long a.out -a -b -c -x foo --add --back  --check --extra=foo
{% endhighlight %}

`getopt_long_only()` 和 `getopt_long()` 类似，但是 `'-'` 和 `'--'` 均被认为是长选项，只有当 `'-'` 没有对应的选项时才会与相应的短选项匹配。







## 整型溢出

以 8-bits 的数据为例，unsigned 取值范围为 0~255，signed 的取值范围为 -128~127。在计算机中数据以补码（正数原码与补码相同，原码=除符号位的补码求反+1）的形式存在，且规定 0x80 为-128 。

### 无符号整数

对于无符号整数，当超过 255 后将会溢出，常见的是 Linux 内核中的 jiffies 变量，jiffies 以及相关的宏保存在 linux/jiffies.h 中，如果 a 发生在 b 之后则返回真，即 a>b 返回真，无论是否有溢出。

{% highlight c %}
#define time_after(a,b)     \
    (typecheck(unsigned long, a) && \
     typecheck(unsigned long, b) && \
     ((long)((b) - (a)) < 0))
{% endhighlight %}

<!--
	下面以8-bit数据进行讲解，在 0x00~0x7F 范围内，表示 0~127；在 0x80~0xFF 范围内表示 -128~-1，对于可能出现的数据有四种情况：<ol><li>
		都位于0x00~0x7F<br>
		如果a发生在b之后，则a > b。(char)b - (char)a < 0，100 - 125 = -25 < 0。</li><br><li>

		都位于0x80~0xFF<br>
		如果a发生在b之后，则a > b。(char)b - (char)a < 0，150 - 180 = -106 - (-76) = -30 < 0。</li><br><li>

		b位于0x00~0x7F，a位于0x80~0xFF<br>
		如果a发生在b之后，则a > b。(char)b - (char)a < 0，100 - 150 = 100 - (-106) = 206 = -50 < 0。<br><strong>注意，此时在a-b<=128时有效。</strong></li><br><li>

		a位于0x00~0x7F，b位于0x80~0xFF<br>
	如果a发生在b之后，此时有溢出。(char)b - (char)a < 0，150 - 10 = -106 - 10 = -116 < 0。<br><strong>注意，此时在a-b<=128时有效。</strong></li></ol>

	<div style="padding: 10pt 0pt 10pt 0pt ;" align="right">
	<table frame="void" width="90%">
		<tbody><tr><td><b>Tips:</b><br><span style="color : #009000"><font size="-1">typecheck位于相同文件夹下的typecheck.h文件中，当两个参数不是同一个类型是将会产生警告，提醒用户注意，只是提醒。</font></span></td>
	<td><img src="src/info.png" width="80" heigh="80"></td></tr></tbody></table></div>
	</p>
-->



<!--
<br id="Error_handling"><br><br>
<h1>错误处理_OK</h1>
<p>
	需要包含的头文件及函数原型，<pre>
#include &lt;stdio.h&gt;
void perror(const char *s);

#include &lt;errno.h&gt;
const char *sys_errlist[];
int sys_nerr;  // 前两个变量是参考了BSD，glibc中保存在&lt;stdio.h&gt;
int errno;</pre>

	如果s不为NULL且*s != '\0'则输出s并加上": "+错误信息+换行，否则只输出错误信息+换行。通常s应该为出错的函数名称，此函数需要调用errno。如果函数调用错误后没有直接调用<tt>perror()</tt>，则为防止其他错误将其覆盖，需要保存errno。<br><br>


sys_errlist保存了所有的错误信息，errno(注意出现错误时进行了设置，但是正确调用时可能没有清除)为索引，最大的索引为<tt>(sys_nerr - 1)</tt>。当直接调用sys_errlist时可能错误信息还没有添加。
</p>

## errno

http://www.bo56.com/linux%E4%B8%ADc%E8%AF%AD%E8%A8%80errno%E7%9A%84%E4%BD%BF%E7%94%A8/
https://www.ibm.com/developerworks/cn/aix/library/au-errnovariable/

-->



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

## \_\_attribute\_\_

### \_\_attribute\_\_((format))

该属性用于自实现的字符串格式化参数添加类似 printf() 的格式化参数的校验，判断需要格式化的参数与入参是否相同。

{% highlight text %}
format (archetype, string-index, first-to-check)

__attribute__((format(printf,m,n)))
__attribute__((format(scanf,m,n)))
  m : 第m个参数为格式化字符串(从1开始)；
  n : 变长参数(也即"...")的第一个参数排在总参数的第几个；
{% endhighlight %}

如下是使用示例。

{% highlight text %}
void myprint(const char *format,...) __attribute__((format(printf,1,2)));
void myprint(int l，const char *format,...) __attribute__((format(printf,2,3)));
{% endhighlight %}

如下是一个简单的使用示例。

{% highlight c %}
#include <stdio.h>

extern void myprint(const char *format,...) __attribute__((format(printf,1,2)));

int myprint(char *fmt, ...)
{
    int result;
    va_list args;
    va_start(args, fmt);
    fputs("foobar: ", stderr);
    result = vfprintf(stderr, fmt, args);
    va_end(args);
    return result;
}
int main(int argc, char **argv)
{
    myprint("i=%d\n",6);
    myprint("i=%s\n",6);
    myprint("i=%s\n","abc");
    myprint("%s,%d,%d\n",1,2);
 return 0;
}
{% endhighlight %}

编译时添加 `-Wall` 就会打印 Warning 信息，如果去除，实际上不会显示任何信息，通常可以提前发现常见的问题。

### \_\_attribute\_\_((constructor))

这是 GCC 的扩展机制，通过上述的属性，可以使程序在开始执行或停止时调用指定的函数。

```__attribute__((constructor))``` 在 main() 之前执行，```__attribute__((destructor))``` 在 main() 执行结束之后执行。

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>

static  __attribute__((constructor)) void before()
{
    printf("Hello World\n");
}

static  __attribute__((destructor)) void after()
{
    printf("Bye World!\n");
}

int main(int args,char ** argv)
{
    printf("Live...\n");
    return EXIT_SUCCESS;
}
{% endhighlight %}

如果有多个函数，可以指定优先级，其中 0~100 (含100)系统保留。在 main 之前顺序为有小到大，退出时顺序为由大到小。

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>

static  __attribute__((constructor(102))) void before102()
{
    printf("Hello World 102\n");
}

static  __attribute__((destructor(102))) void after102()
{
    printf("Bye World! 102\n");
}

static  __attribute__((constructor(101))) void before101()
{
    printf("Hello World 101\n");
}

static  __attribute__((destructor(101))) void after101()
{
    printf("Bye World! 101\n");
}

int main(int args,char ** argv)
{
    printf("Live...\n");
    return EXIT_SUCCESS;
}
{% endhighlight %}

在使用时也可以先声明然再定义


{% highlight c %}
#include <stdio.h>
#include <stdlib.h>

void before() __attribute__((constructor));
void after() __attribute__((destructor));

void before()
{
    printf("Hello World\n");
}

void after()
{
    printf("Bye World!\n");
}

int main(int args,char ** argv)
{
    printf("Live...\n");
    return EXIT_SUCCESS;
}
{% endhighlight %}




## atexit()

很多时候我们需要在程序退出的时候做一些诸如释放资源的操作，但程序退出的方式有很多种，比如 main() 函数运行结束、在程序的某个地方用 exit() 结束程序、用户通过 Ctrl+C 或 Ctrl+break 操作来终止程序等等，因此需要有一种与程序退出方式无关的方法来进行程序退出时的必要处理。

方法就是用 atexit() 函数来注册程序正常终止时要被调用的函数。

{% highlight c %}
#include <stdlib.h>
int atexit(void(*func)(void));
{% endhighlight %}

成功时返回零，失败时返回非零。

在一个程序中至少可以用 atexit() 注册 32 个处理函数，依赖于编译器。这些处理函数的调用顺序与其注册的顺序相反，也即最先注册的最后调用，最后注册的最先调用。

{% highlight c %}
void fnExit1 (void) { puts ("Exit function 1."); }
void fnExit2 (void) { puts ("Exit function 2."); }

int main ()
{
    atexit (fnExit1);
    atexit (fnExit2);
    puts ("Main function.");
    return 0;
}
{% endhighlight %}

## do-while

如果通过 ```define``` 定义一个含有多个语句的宏，通常我们使用 ```do{...} while(0);``` 进行定义，具体原因，如下详细介绍。

如果想在宏中包含多个语句，可能会如下这样写。

{% highlight c %}
#define do_something() \
   do_a(); \
   do_b();
{% endhighlight %}

通常，这样就可以用 ```do_somethin()``` 来执行一系列操作，但这样会有个问题：如果通过如下的方式用这个宏，将会出错。

{% highlight c %}
if (...)
   do_something();

// 宏被展开后
if (...)
   do_a();
   do_b();
{% endhighlight %}

原代码的目的是想在 if 为真的时候执行 ```do_a()``` 和 ```do_b()```, 但现在，实际上只有 ```do_a()``` 在条件语句中，而 ```do_b()``` 任何时候都会执行。

当然这时可以通过如下的方式将那个宏改进一下。

{% highlight c %}
#define do_something() { \
   do_a(); \
   do_b(); \
}
{% endhighlight %}

然而，即使是这样，仍然有错。假设有一个宏是这个样子的，

{% highlight c %}
#define do_something() { \
   if (a)     \
      do_a(); \
   else       \
      do_b(); \
{% endhighlight %}

在使用如下情况时，仍会出错。

{% highlight c %}
if (...)
   do_something();
else {
   ...
}

// 宏展开后
if (...)
{
   if (a)
      do_a();
   else
      do_b();
}; else {
   ...
}
{% endhighlight %}

此时第二个 else 前边会有一个分号，那么编译时就会出错。

因此对于含有多条语句的宏我们使用 ```do{...} while(0);``` ，do-while 语句是需要分号来结束的，另外，现代编译器的优化模块能够足够聪明地注意到这个循环只会执行一次而将其优化掉。

综上所述，```do{...} while(0);``` 这个技术就是为了类似的宏可以在任何时候使用。


## assert()

其作用是如果它的条件返回错误，则输出错误信息 (包括文件名，函数名等信息)，并终止程序执行，原型定义：

{% highlight c %}
#include <assert.h>
void assert(int expression);
{% endhighlight %}

如下是一个简单的示例。

{% highlight c %}
#include <stdio.h>
#include <assert.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    FILE *fp;

   fp = fopen( "test.txt", "w" ); // 不存在就创建一个同名文件
   assert( fp );                  // 所以这里不会出错
   fclose( fp );

    fp = fopen( "noexitfile.txt", "r" );  // 不存在就打开文件失败
    assert( fp );                         // 这里出错
    fclose( fp );                         // 程序不会执行到此处

    return 0;
}
{% endhighlight %}

当在 ```<assert.h>``` 之前定义 NDEBUG 时，assert 不会产生任何代码，否则会显示错误。

### 判断程序是否有 assert

在 glibc 中，会定义如下的内容：

{% highlight c %}
#define assert(e) ((e \
    ? ((void)0) \
    :__assert_fail(#e,__FILE__,__LINE__))
{% endhighlight %}

可以通过 nm 查看程序，判断是否存在 ```__assert_fail@@GLIBC_2.2.5``` ，如果存在该函数则说明未关闭 ```assert()``` 。

对于 autotool 可以通过如下的一种方式关闭：

1. 在 ```configure.ac``` 文件中添加 ```AC_HEADER_ASSERT``` ，然后如果关闭是添加 ```--disable-assert``` 参数，注意，一定要保证源码包含了 ```config.h``` 头文件；
2. 执行 configure 命令前设置环境变量，如 ```CPPFLAGS="CPPFLAGS=-DNDEBUG" ./configure```；
3. 也可以在 ```Makefile.am``` 中设置 ```AM_CPPFLAGS += -DNDEBUG``` 参数。


<!--
## 宏定义
http://hbprotoss.github.io/posts/cyu-yan-hong-de-te-shu-yong-fa-he-ji-ge-keng.html
http://gcc.gnu.org/onlinedocs/cpp/Macros.html
-->

## 其它

### is not a symbolic link

正常情况下，类似库 ```libxerces-c-3.0.so``` 应该是个符号链接，而不是实体文件，对于这种情况只需要修改其为符号链接即可。

{% highlight text %}
# mv libxerces-c-3.0.so libxerces-c.so.3.0
# ln -s libxerces-c.so.3.0 libxerces-c-3.0.so
{% endhighlight %}

### 指针参数修改

一个比较容易犯错的地方，愿意是在 `foobar()` 函数内修改 `main()` 中的 v 指向的变量，其中后者实际上是修改的本地栈中保存的临时版本。

{% highlight c %}
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct value_s {
  union {
    char *string;
  } value;
} value_t;

#if 1
void foobar ( value_t *t )
{
  char **v = &t->value.string;
  printf("foobar %p %s\n", *v, *v);
  *v = "yang";
  printf("foobar %p %s\n", *v, *v);
}
#else
void foobar ( value_t *t )
{
  value_t v = *t;
  printf("foobar %p %s\n", v.value.string, v.value.string);
  v.value.string = "yang";
  printf("foobar %p %s\n", v.value.string, v.value.string);
}
#endif

int main()
{
  value_t v;
  v.value.string = "jin";
  printf("       %p %s\n", v.value.string, v.value.string);
  foobar( &v );
  printf("       %p %s\n", v.value.string, v.value.string);

  return(0);
}
{% endhighlight %}

### 取固定大小的变量

Linux 每个数据类型的大小可以在 `sys/types.h` 中查看

{% highlight c %}
#include <sys/types.h>
typedef    int8_t            S8;
typedef    int16_t           S16;
typedef    int32_t           S32;
typedef    int64_t           S64;

typedef    u_int8_t          U8;
typedef    u_int16_t         U16;
typedef	   u_int32_t         U32;
typedef    u_int64_t         U64;
{% endhighlight %}


<!--
http://sobuhu.com/program/2012/12/11/c.html
-->

{% highlight text %}
{% endhighlight %}
