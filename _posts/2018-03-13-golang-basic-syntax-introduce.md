---
title: Golang 语法简介
layout: post
comments: true
language: chinese
category: [program,golang]
keywords: golang,syntax,import,exception,reflection
description: 简单介绍常见的语法，例如 import、异常处理、反射等。
---

简单介绍常见的语法，例如 import、异常处理、反射等。

<!-- more -->

## 常用结构

### 赋值

有两种相关的符号，`=` 表示赋值，`:=` 声明变量并赋值。

{% highlight go %}
//----- 使用 = 必须先用 var 声明
var a
a = 100
var b = 100
var c int = 100

//----- 使用 := 系统会自动推断类型，不需要var关键字
d := 100
{% endhighlight %}

### 类型

<!--
http://colobu.com/2017/06/26/learn-go-type-aliases/
-->

可以通过 Type 把一个类型转换成另外一个类型而保持数据结构不变，如下所示：

{% highlight go %}
type Age int
type Height int
type Grade int
{% endhighlight %}

这里的 type 绝不只是对应于 C/C++ 中的 typedef，它不是用于定义一系列的别名，更关键的是定义了一系列互不相干的行为特征：通过这些互不相干的行为特征，本质上同一的事物表现出不同事物的特征，整数还是整数，但年龄却不是高度也不是分数。

可以分别为 Age、Height、Grade 定义出下列不同的行为(表示为方法或者函数)：

{% highlight go %}
// 超过50岁算老年
func (a Age) Old() bool {
	return a > 50
}
// 高于120cm需要买票
func (l Height) NeedTicket() bool {
	return l > 120
}
// 60分及格
func (g Grade) Pass() bool {
	return g >= 60
}
{% endhighlight %}

### 数组

数组是内置类型，相同数据类型的集合，下标从 0 开始，初始化后长度固定，且无法修改其长度。当作为方法的入参传入时将复制一份数组而不是引用同一指针，而且长度也是其类型的一部分，可以通过内置函数 `len(array)` 获取其长度。

可以通过如下方式初始化。

{% highlight go %}
// 长度为5的数组，其元素分别为1, 2, 3, 4, 5
[5] int {1,2,3,4,5}

// 长度为5的数组，未赋值的默认是 0 ，也就是其元素值依次为1, 2, 0, 0, 0 
[5] int {1,2} 

// 长度为5的数组，其长度是根据初始化时指定的元素个数决定的
[...] int {1,2,3,4,5} 

// 长度为5的数组，key:value,其元素值依次为：0，0，1，2，3。在初始化时指定了2，3，4索引中对应的值：1，2，3
[5] int { 2:1,3:2,4:3} 

// 长度为5的数组，起元素值依次为：0，0，1，0，3。由于指定了最大索引4对应的值3，根据初始化的元素个数确定其长度为5
[...] int {2:1,4:3} 
{% endhighlight %}

#### 数组遍历

通常有如下的两种遍历方式。

{% highlight go %}
package main

import "fmt"

func main() {
        var arr = [...]int{1, 2, 3, 4, 5}

        for idx, val := range arr {
                fmt.Printf("array[%d] = %d\n", idx, val)
        }
        fmt.Println(">>>>>>>>>>>>>")

        for idx := 0; idx < len(arr); idx++ {
                fmt.Printf("array[%d] = %d\n", idx, arr[idx])
        }
}
{% endhighlight %}

### 结构体

通过结构体新建对象时的语法比较多，而且相比而言有些特殊。

{% highlight go %}
type Poem struct {
    Title  string
    Author string
    intro  string
}
{% endhighlight %}

需要注意下访问权限，如果属性的开头字母是大写的则在其它包中可以被访问，否则只能在本包中访问；类的声明和方法亦是如此。

然后可以通过如下的方法进行赋值。

{% highlight go %}
poem1 := &Poem{}
poem1.Author = "Heine"
poem2 := &Poem{Author: "Heine"}
poem3 := new(Poem)
poem3.Author = "Heine"
poem4 := Poem{}
poem4.Author = "Heine"
poem5 := Poem{Author: "Heine"}
{% endhighlight %}

### 嵌入类型

结构体类型可以包含匿名或者嵌入字段，该类型的名字会充当嵌入字段的字段名。

{% highlight go %}
package main

import (
    "log"
)

type User struct {
        Name  string
        Email string
}

type Admin struct {
        User
        Level string
}

func (u *User) Notify() error {
        log.Printf("User: Sending User Email To %s<%s>\n", u.Name, u.Email)
        return nil
}

func (a *Admin) Notify() error {
        log.Printf("Admin: Sending Admin Email To %s<%s>\n", a.Name, a.Email)
        return nil
}

type Notifier interface {
        Notify() error
}

func SendNotification(notify Notifier) error {
        return notify.Notify()
}

func main() {
        admin := &Admin{
                User: User{
                        Name:  "AriesDevil",
                        Email: "ariesdevil@xxoo.com",
                },
                Level: "super",
        }

        SendNotification(admin)
        admin.Notify()
        admin.User.Notify()
}
{% endhighlight %}

如果对于子类重新定义了接口，那么默认调用的时候是子类的，也可以显式调用父类。

### 函数

<!--
http://jordanorelli.com/post/42369331748/function-types-in-go-golang
-->

在 golang 中，支持匿名函数和闭包，其中定义函数的方式如下：

{% highlight go %}
func (p myType) funcName ( a, b int, c string ) ( r, s int ) {
	return
}
{% endhighlight %}

包括了定义函数的关键字 `func`，函数名称 `funcName`，入参 `a, b int, c string`，返回值 `r,s int` 以及函数体 `{}`。

而且，golang 可以为某个类型定义函数，也即为类型对象定义方法，也就是 `p myType` 参数，当然这不是必须的，如果为空则纯粹是一个函数，可以通过包名称访问。

## 字符串操作

这应该是最常见的，在 Golang 中有多种方式可以完成拼接，详见如下的测试程序。

{% highlight go %}
package main

import (
        "bytes"
        "fmt"
        "strings"
        "time"
)

var ways = []string{
        "fmt.Sprintf ",
        "+           ",
        "strings.Join",
        "bytes.Buffer",
}

func benchmarkStringFunction(n int, idx int) {
        var s string
        var buf bytes.Buffer

        v := "hello world, just for test"
        begin := time.Now()
        for i := 0; i < n; i++ {
                switch idx {
                case 0: // fmt.Sprintf
                        s = fmt.Sprintf("%s[%s]", s, v)
                case 1: // string +
                        s = s + "[" + v + "]"
                case 2: // strings.Join
                        s = strings.Join([]string{s, "[", v, "]"}, "")
                case 3: // stable bytes.Buffer
                        buf.WriteString("[")
                        buf.WriteString(v)
                        buf.WriteString("]")
                }

        }
        if idx == 3 {
                s = buf.String()
        }
        fmt.Printf("string len: %d\t", len(s))
        fmt.Printf("time of [%s]=\t %v\n", ways[idx], time.Since(begin))
}

func main() {
        for idx, _ := range ways {
                benchmarkStringFunction(10000, idx)
        }
}
{% endhighlight %}

执行结果如下。

{% highlight text %}
string len: 280000      time of [fmt.Sprintf ]=  366.809538ms
string len: 280000      time of [+           ]=  231.356836ms
string len: 280000      time of [strings.Join]=  497.997435ms
string len: 280000      time of [bytes.Buffer]=  867.259µs
{% endhighlight %}

结论: A) `strings.Join` 最慢；B) 其次为 `fmt.Sprintf` 和 `string +`；C) 最快为 `bytes.Buffer` 。

<!--
https://sheepbao.github.io/post/golang_byte_slice_and_string/
-->

## import

在 golang 中可以通过如下的方式导入。

{% highlight go %}
import(
	"fmt"       // 标准库
	"./model"   // 本地库
	"model/png" // 加载$GOPATH/src/model/png中的库
	. "png"     // 直接使用相关的函数即可，无需包前缀
	p "png"     // 重命名包名
)
{% endhighlight %}

另外，常见的一种操作符 `_` ，例如：

{% highlight go %}
import ("database/sql" _ "github.com/ziutek/mymysql/godrv")
{% endhighlight %}

这里其实只是引入该包，当导入一个包时，它所有的 `init()` 函数就会被执行，如果仅仅是希望它的 `init()` 函数被执行，此时就可以使用 `_` 。

引入的初始化顺序为：

![golang logo]({{ site.url }}/images/go/init-sequence.png "golang logo"){: .pull-center width="70%" }

1. import pkg 的初始化过程；
2. pkg 中定义的 const 变量初始化；
3. pkg 中定义的 var 全局变量；
4. pkg 中定义的 init 函数，可能有多个。

## 异常处理

Go 追求的是简洁优雅，没有提供传统的 `try ... catch ... finally` 这种异常处理方式，引入的是 `defer` `panic` `recover` 。也就是在 Go 中抛出一个 `panic` 异常，然后在 `defer` 中通过 `recover` 捕获这个异常，然后正常处理。

Go 对待异常 (准确说是panic) 态度是：没有全面否定异常的存在，但极不鼓励多用异常。

{% highlight go %}
package main

import "fmt"

func main() {
        defer func() {
                if err := recover(); err != nil {
                        fmt.Println(err)
                }
                fmt.Println("Process panic done")
        }()

        foobar()
}

func foobar() {
        fmt.Println("Before panic")
        panic("Panicing ...")
        fmt.Println("After panic")
}
{% endhighlight %}

通过 `go run main.go` 执行会输出如下内容。

{% highlight text %}
Before panic
Panicing ...
Process panic done
{% endhighlight %}

也就是说，在 `Panic` 之后的内容不会再执行。

<!--
参考：http://blog.dccmx.com/2012/01/exception-the-go-way/
http://kejibo.com/golang-exceptions-handle-defer-try/
http://bookjovi.iteye.com/blog/1335282
https://github.com/astaxie/build-web-application-with-golang/blob/master/02.3.md
-->

## 协程

另外，一种方式是一直阻塞在管道中，利用 for 循环遍历。

{% highlight go %}
package main

import (
        "fmt"
        "time"
)

func readCommits(commitC <-chan *string) {
        for data := range commitC {
                if data == nil {
                        fmt.Println("Got nil data")
                        continue
                }
                fmt.Println(*data)
        }
}

func main() {
        commitC := make(chan *string)

        go func() {
                for {
                        s := "hi"
                        time.Sleep(1 * time.Second)

                        commitC <- &s
                }
        }()

        readCommits(commitC)
}
{% endhighlight %}

## 反射

反射允许你可以在运行时检查类型，包括检查、修改、创建变量、函数、结构体等。

{% highlight go %}
package main

import (
        "fmt"
        "reflect"
)

type Foobar struct {
        name string
}

func (f *Foobar) GetName() string {
        return f.name
}

func main() {
        s := "this is string"
        fmt.Println(reflect.TypeOf(s))
        fmt.Println(reflect.ValueOf(s))

        var x float64 = 3.4
        fmt.Println(reflect.ValueOf(x))

        a := new(Foobar)
        a.name = "foo bar"
        t := reflect.TypeOf(a)
        fmt.Println(t.NumMethod())
        b := reflect.ValueOf(a).MethodByName("GetName").Call([]reflect.Value{})
        fmt.Println(b[0])

}
{% endhighlight %}


## 管道 (channel)

<!--
http://colobu.com/2016/04/14/Golang-Channels/
-->

核心类型，可以看做是一个 FIFO 的阻塞消息队列，用于发核心单元之间发送和接收数据，默认是双向的，也可以指定方向，使用前必须要创建并初始化，

{% highlight go %}
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}
{% endhighlight %}

如上中的 `make(chan int, 100)` ，上述的第二个参数可选，表示容量，也就是管道可以容纳的最多元素数量，代表管道的缓存大小。

大小默认是 0 ，也就是如果接收、发送没有准备好，另外一端就会阻塞，如果设置了缓存，只有 buffer 满了后 send 才会阻塞，当缓存空了后 receive 才会阻塞。

<!--
if-else语句，比较奇葩的是
https://gobyexample.com/if-else

Select语句
https://segmentfault.com/a/1190000006815341
-->

简单示例如下。

{% highlight go %}
package main

import "time"
import "fmt"

func main() {
	// For our example we'll select across two channels.
	c1 := make(chan string)
	c2 := make(chan string)

	// Each channel will receive a value after some amount
	// of time, to simulate e.g. blocking RPC operations
	// executing in concurrent goroutines.
	go func() {
		time.Sleep(1 * time.Second)
		c1 <- "one"
	}()
	go func() {
		time.Sleep(2 * time.Second)
		c2 <- "two"
	}()

	// We'll use `select` to await both of these values
	// simultaneously, printing each one as it arrives.
	for i := 0; i < 2; i++ {
		select {
		case msg1 := <-c1:
			fmt.Println("received", msg1)
		case msg2 := <-c2:
			fmt.Println("received", msg2)
		}
	}
}
{% endhighlight %}

### Select 行为

从一个 Topic 中提取的代码。

{% highlight go %}
// https://talks.golang.org/2012/concurrency.slide#32
select {
case v1 := <-c1:
	fmt.Printf("received %v from c1\n", v1)
case v2 := <-c2:
	fmt.Printf("received %v from c2\n", v1)
case c3 <- 23:
	fmt.Printf("sent %v to c3\n", 23)
default:
	fmt.Printf("no one was ready to communicate\n")
}
{% endhighlight %}

上述代码中包含了三个 case 子句以及一个 default 子句，前两个是 receive 操作，第三个是 send 操作，最后一个是默认操作。

当代码执行到 select 时，case 语句会按照源代码的顺序被评估，且只评估一次，评估的结果会出现下面这几种情况：

* 除 default 外，如果只有一个 case 语句评估通过，那么就执行这个 case 里的语句；
* 除 default 外，如果有多个 case 语句评估通过，那么通过伪随机的方式随机选一个；
* 如果 default 外的 case 语句都没有通过评估，那么执行 default 里的语句；
* 如果没有 default，那么 代码块会被阻塞，直到有一个 case 通过评估，否则会一直阻塞；

### 其它

另外，需要注意，如果向管道中发送的数据，而有其它的协程阻塞等待，那么条件满足后会先调度执行被阻塞的协程。

{% highlight go %}
package main

import (
        "fmt"
)

func main() {
        readyc := make(chan string)

        go func() {
                select {
                case str := <-readyc:
                        fmt.Printf("%s, Go !!!\n", str)
                }

        }()

		readyc <- "Reaaaaady"
		fmt.Println("Done")
}
{% endhighlight %}

也就是上述的输出为：

{% highlight text %}
Reaaaaady, Go !!!
Done
{% endhighlight %}

### 示例

#### Timeout

{% highlight go %}
package main

import "time"
import "fmt"

func main() {
        timeout := make (chan bool, 1)
        ch := make (chan int)
        go func() {
                time.Sleep(2 * time.Second)
                timeout <- true
        }()
        select {
        case <- ch:
        case <- timeout:
            fmt.Println("timeout!")
        }
}
{% endhighlight %}

因为没有向管道 ch 发送数据，默认应该是一直等待。也可以使用如下示例：

{% highlight go %}
package main

import (
        "fmt"
        "time"
)

func main() {
        donec := make(chan string)

        go func() {
                time.Sleep(1 * time.Second)
                donec <- "done"
        }()

        select {
        case s := <-donec:
                fmt.Printf("Got string '%s'.\n", s)
        case <-time.After(2 * time.Second):
                fmt.Println("Timeout")
        }
}
{% endhighlight %}

### 检查队列是否满

这里使用的是 default 这个特性。

{% highlight go %}
package main

import "fmt"

func main() {
        ch := make (chan int, 1)
        ch <- 1
        select {
                case ch <- 2:
                default:
                        fmt.Println("channel is full !")
        }
}
{% endhighlight %}

因为 ch 插入 1 的时候已经满了， 当 ch 要插入 2 的时候，发现 ch 已经满了此时默认应该是阻塞，不过因为有 default 子句，实际会执行 default 子语， 这样就可以实现对 channel 是否已满的检测， 而不是一直等待。

例如在并发处理多个 job 时，如果队列满了，则返回错误让用户重试。

<!--
Quit Channel/Done Channel 还没有搞明白
https://segmentfault.com/a/1190000006815341
-->

## 竞态条件

如下是一个简单的示例，简单来说，是对一个内存中的值进行累加，

{% highlight go %}
package main

import (
        "fmt"
        "sync"
)

var (
        N         = 0
        waitgroup sync.WaitGroup
)

func counter(number *int) {
        *number++
        waitgroup.Done()
}

func main() {

        for i := 0; i < 1000; i++ {
                waitgroup.Add(1)
                go counter(&N)
        }
        waitgroup.Wait()
        fmt.Println(N)
}
{% endhighlight %}

如果运行多次，可以发现绝大多数情况下其累加值不是 1000 。

解决方法是在执行累加时，对操作加锁。

{% highlight go %}
package main

import (
        "fmt"
        "sync"
)

var (
        N         = 0
        mutex     sync.Mutex
        waitgroup sync.WaitGroup
)

func counter(number *int) {
        mutex.Lock()
        *number++
        mutex.Unlock()
        waitgroup.Done()
}

func main() {

        for i := 0; i < 1000; i++ {
                waitgroup.Add(1)
                go counter(&N)
        }
        waitgroup.Wait()
        fmt.Println(N)
}
{% endhighlight %}

### 竞态条件检测

GoLang 工具内置了竞态检测工具，只需要加上 `-race` 即可。

{% highlight text %}
$ go test -race mypkg    // test the package
$ go run -race mysrc.go  // compile and run the program
$ go build -race mycmd   // build the command
$ go install -race mypkg // install the package
{% endhighlight %}

使用该参数，GO 会记录不同线程对共享变量的访问情况，如果发现非同步的访问则会退出并打印告警信息。


## 语法糖

### ...

主要有两种用法：A) 用于函数有多个不定参数场景；B) slice 可以被打散进行传递。

{% highlight go %}
package main

import "fmt"

func foobar(args ...string) {
        for _, v := range args {
                fmt.Println(v)
        }
}

func main() {
        var foostr = []string{"hello", "world"}
        var barstr = []string{"hi", "fooo", "baar,"}

        foobar("foo", "bar", "foobar")

        fmt.Println(append(barstr, foostr...))
}
{% endhighlight %}

### Byte VS. Rune

两种实际上是 `uint8` 和 `uint32` 类型，byte 用来强调数据是 RawData，而不是数字；而 rune 用来表示 Unicode 编码的 CodePoint。

中文字符使用 3 个字节保存(为什么使用的是3个字节保存，还没有搞清楚)。

{% highlight go %}
s := "hello你好"
fmt.Println(len(s))         // 11
fmt.Println(len([]rune(s))) // 7，需要先转换为rune的切片在使用内置len函数
s = "你好"
fmt.Println(len(s))         // 6
fmt.Println(len([]rune(s))) // 2
s = "你"
fmt.Println([]byte(s)) // 三个字节，也就是中文的表示方法
fmt.Println(rune('你')) // 输出20320(0x4F60)，'你' 的编码
{% endhighlight %}

<!--
Strings, bytes, runes and characters in Go
https://blog.golang.org/strings
-->

{% highlight go %}
{% endhighlight %}
