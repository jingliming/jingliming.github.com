---
title: Golang 常用模块
layout: post
comments: true
language: chinese
category: [program,golang]
keywords: golang,log
description: 简单介绍常见的三方模块使用，例如 log、unsafe 等。
---

简单介绍常见的三方模块使用，例如 log、unsafe 等。

<!-- more -->


## 定时器 (time)

golang 通过 time 包的函数初始化定时器，可以通过 `reset()`、`stop()` 重置和停止定时器。另外，定时器存在一个 `chan time.Time` 类型的缓冲 channel 字段 C，时间到了之后，定时器就会向自己的 C 字段发送一个 `time.Time` 类型的元素值。

### time

最简单的定时器。

{% highlight go %}
package main

import (
        "fmt"
        "time"
)

func main() {
        t := time.NewTimer(2 * time.Second)

        now := time.Now()
        fmt.Printf("Now time : %v.\n", now)

        expire := <-t.C
        fmt.Printf("Expiration time: %v.\n", expire)
}
{% endhighlight %}

### Ticker

周期性的触发时间事件，会以指定的时间间隔重复的向通道 C 发送时间值。

{% highlight go %}
package main

import (
        "fmt"
        "time"
)

func main() {
        var ticker *time.Ticker = time.NewTicker(1 * time.Second)

        go func() {
                for t := range ticker.C {
                        fmt.Println("Tick at", t)
                }
        }()

        time.Sleep(time.Second * 5)
        ticker.Stop()
        fmt.Println("Ticker stopped")
}
{% endhighlight %}

如果使用周期性的定时器，可以使用 `time.Ticker` 类型。

<!--
package main

import (
        "fmt"
        "time"
)

func main() {
        t := time.NewTicker(1 * time.Second)
        defer t.Stop()

        for {
                select {
                case <-t.C:
                        fmt.Println(time.Now())
                }
        }
}
-->

## fmt

其中最常用的是 `%v`，这是一个通用的格式化方式，用来打印 struct 的成员变量名称。

{% highlight text %}
%v   默认格式，只打印各个字段的值，没有字段名称；
%+v  当打印结构体时，同时会添加字段名称；
%#v  类似+v，同时会打印对象名称，并用语法标示数据类型，例如字符串会加""；
%T   变量对应的类型名称 pakage.struct；
{% endhighlight %}

示例如下。

{% highlight go %}
package main

import (
        "fmt"
)

type User struct {
        name string
        age  int
}

func main() {
        u := &User{name: "andy", age: 30}

        fmt.Printf("%v\n", *u)  // {andy 30}
        fmt.Printf("%+v\n", *u) // {name:andy age:30}
        fmt.Printf("%#v\n", *u) // main.User{name:"andy", age:30}
        fmt.Printf("%T\n", *u)  // main.User
}
{% endhighlight %}

## 日志 (log)

这也是官方标准的日志库，使用方式与 fmt 类似，只是默认输出时会添加时间信息。

{% highlight go %}
package main

import "log"

func main() {
        log.Println("Hello world")
        log.Printf("Hi...")
}
{% endhighlight %}

可以在启动时通过 `init()` 函数进行一些初始化操作，其中 log 库提供了一些常见的配置项，可以通过 `func SetFlags(flag int)` 设置。

{% highlight go %}
const (
	Ldate         = 1 << iota     // 日期示例: 2009/01/23
	Ltime                         // 时间示例: 01:23:23
	Lmicroseconds                 // 毫秒示例: 01:23:23.123123.
	Llongfile                     // 绝对路径和行号: /a/b/c/d.go:23
	Lshortfile                    // 文件和行号: d.go:23.
	LUTC                          // 日期时间转为UTC时区的
	LstdFlags     = Ldate | Ltime // 默认的格式输出
)
{% endhighlight %}

也可以通过 `func SetPrefix(prefix string)` 设置日志的输出前缀。

另外，提供了 `Fatalln()` 和 `Panicln()` 函数，前者会在打印日志后直接退出；后者如果没有使用 `recover()` 函数则会打印错误栈信息后退出。



### 日志格式定制

在 `init()` 函数中，对日志的格式进行定制化处理。

其它常见的三方库有 [GitHub ZAP](https://github.com/uber-go/zap) 中的介绍。

<!--
//定义logger, 传入参数 文件，前缀字符串，flag标记
func New(out io.Writer, prefix string, flag int) *Logger

log 的源码实现也很简单，可以参考
http://www.flysnow.org/2017/05/06/go-in-action-go-log.html
-->

## bytes

{% highlight go %}
package main

import (
        "bytes"
        "encoding/binary"
        "fmt"
)

func IntToBytes(n int) []byte {
        bytesBuffer := bytes.NewBuffer([]byte{})
        binary.Write(bytesBuffer, binary.BigEndian, int32(n))
        return bytesBuffer.Bytes()
}

func main() {
        var b0 bytes.Buffer
        b0.Write([]byte("Hello "))
        fmt.Println(b0.String())

        b1 := new(bytes.Buffer)
        b1.WriteString("Hi World")
        b1.WriteByte('!')
        fmt.Println(b1.String())

/*
	// OR
	b2 := bytes.NewBufferString("swift")
	b3 := bytes.NewBuffer([]byte("swift"))
	b4 := bytes.NewBuffer([]byte{'s', 'w', 'i', 'f', 't'})

	// Empty Buffer
	b5 = bytes.NewBufferString("")
	b6 = bytes.NewBuffer([]byte{})
*/
}
{% endhighlight %}

## unsafe

这是一个很特殊的包，可以绕过 GoLang 本身的一些语法检查直接操作对象，从而可能会导致不可移植(可控)，而且使用比较危险。该包中包含了三个函数，以及一种类型：

{% highlight go %}
func Alignof(variable ArbitraryType) uintptr
func Offsetof(selector ArbitraryType) uintptr
func Sizeof(variable ArbitraryType) uintptr

Pointer *ArbitraryType
{% endhighlight %}

上述函数类似于 C 中的宏，在编译时求值，而非运行时，也就是说它的结果可以分配给常量。

另外，`uintptr` 实际上是整型，其大小根据不同的平台变化，可以用来保存指针，例如在 32Bits 下是 4Bytes，在 64Bits 下是 8Bytes 。

### 使用示例

在官方文档中有介绍其常见的使用场景。

#### unsafe.Pointer VS. uintptr

`uintptr` 是一个可以容纳指针地址的整数类型，及时该变量仍然有效，但是其所指向地址的数据可能已经被 GC 回收掉。

`unsafe.Pointer` 是一个通用的指针类型，如果该变量有效，那么其指向的地址出的数据就不会被 GC 回收掉。

因为 `uintptr` 是一个整型，可以进行算术运算；那么就可以使用上述两者绕过限制操作变量，计算结构体中变量的偏移。

如下是两个示例，分别用来操作数组和结构体成员。

{% highlight go %}
package main

import (
        "fmt"
        "unsafe"
)

func main() {
        a := [4]int{0, 1, 2, 3}
        p1 := unsafe.Pointer(&a[1])
        p3 := unsafe.Pointer(uintptr(p1) + 2*unsafe.Sizeof(a[0]))
        *(*int)(p3) = 6
        fmt.Println("a =", a) // a = [0 1 2 6]

        type Person struct {
                name   string
                age    int
                gender bool
        }

        who := Person{"John", 30, true}
        pp := unsafe.Pointer(&who)
        pname := (*string)(unsafe.Pointer(uintptr(pp) + unsafe.Offsetof(who.name)))
        page := (*int)(unsafe.Pointer(uintptr(pp) + unsafe.Offsetof(who.age)))
        pgender := (*bool)(unsafe.Pointer(uintptr(pp) + unsafe.Offsetof(who.gender)))
        *pname = "Alice"
        *page = 28
        *pgender = false
        fmt.Println(who) // {Alice 28 false}
}
{% endhighlight %}

## 并发控制

控制并发有两种经典的方式：`WaitGroup` 和 `Context` 。

### WaitGroup

一种控制并发的方式，它的这种方式是控制多个 goroutine 同时完成，有点类似于 `waitpid()` 函数，用于多个协程同步，通过 `Add()` 增加，`Done()` 减小，`wait()` 会等待到 0 。

{% highlight go %}
package main

import (
        "fmt"
        "sync"
        "time"
)

func main() {
        var wg sync.WaitGroup
        wg.Add(2)
        go func() {
                time.Sleep(1 * time.Second)
                fmt.Println("No. #1 Done")
                wg.Done()
        }()
        go func() {
                time.Sleep(2 * time.Second)
                fmt.Println("No. #2 Done")
                wg.Done()
        }()
        wg.Wait()
        fmt.Println("OK, All Done")
}
{% endhighlight %}

### Channel

简单来说，就是通过 `Channel` 和 `Select` 这种方式进行同步。

{% highlight go %}
package main

import (
        "fmt"
        "time"
)

func main() {
        stop := make(chan bool)
        go func() {
                for {
                        select {
                        case <-stop:
                                fmt.Println("Quit now")
                                return
                        default:
                                fmt.Println("Running")
                                time.Sleep(1 * time.Second)
                        }
                }
        }()
        time.Sleep(5 * time.Second)
        fmt.Println("Time up...")
        stop <- true
        fmt.Println("Bye Bye")
}
{% endhighlight %}

如上的例子中，通过预先定义的一个 chan 来通知后台的协程。

这种方式比较适合与一些可预期的简单场景，例如有多个协程需要控制、协程又派生了子协程。

### Context

对于上述的场景，常见的是 HTTP 请求，每个 Request 都需要开启一个协程，同时这些协程又有可能派生其它的协程，例如处理身份认证、Token校验等；Context 就是提供了一种协程的跟踪方案。

对于 go1.6 及之前版本使用 `golang.org/x/net/context` 而在 1.7 版本之后已移到标准库 context 。

#### 简介

Context 的调用应该是链式的，通过 WithCancel、WithDeadline、WithTimeout 或 WithValue 派生出新的 Context，当父 Context 被取消时，其派生的所有 Context 都将取消。

上述 WithXXX 返回新的 Context 和 CancelFunc，调用 CancelFunc 将取消子代，移除父代对子代的引用，并且停止所有定时器；未调用 CancelFunc 将泄漏子代，直到父代被取消或定时器触发。

<!--
注意，一般来说，在使用时需要遵循以下规则：

1. 不要将 Contexts 放入结构体，相反应该作为第一个参数传入，命名为 ctx；
2. 即使函数允许，也不要传入 nil 的 Context；如果不知道用哪种 Context，可以使用 context.TODO()；
使用context的Value相关方法只应该用于在程序和接口中传递的和请求相关的元数据，不要用它来传递一些可选的参数
相同的 Context 可以传递给在不同的goroutine；Context 是并发安全的。
-->

`context.Background()/TODO()` 会返回一个空 Context，一般将该 Context 作为整个 Context 树的根节点，然后调用 withXXX 函数创建可取消的子 Context，并作为参数传递给其它的协程，从而实现跟踪该协程的目的。

如下是一个示例。

{% highlight go %}
package main

import (
        "context"
        "fmt"
        "time"
)

func main() {
        ctx, cancel := context.WithCancel(context.Background())
        go watch(ctx, "CTX1")
        go watch(ctx, "CTX2")
        go watch(ctx, "CTX3")
        time.Sleep(5 * time.Second)
        fmt.Println("Time up...")
        cancel()
        time.Sleep(1 * time.Second)
}

func watch(ctx context.Context, name string) {
        for {
                select {
                case <-ctx.Done():
                        fmt.Println(name, "Quit now")
                        return
                default:
                        fmt.Println(name, "Running")
                        time.Sleep(1 * time.Second)
                }
        }
}
{% endhighlight %}





<!--
虽然 Go 中的协程相比系统线程来说已经是轻量级了，但是在高并发时，协程的频繁创建和销毁同样对 GC 造成很大的压力。

而协程池 [GitHub grpool](https://github.com/ivpusic/grpool) 就是对协程的封装。

也可以参考
https://www.cnblogs.com/276815076/p/8416652.html

-->


{% highlight go %}
{% endhighlight %}
