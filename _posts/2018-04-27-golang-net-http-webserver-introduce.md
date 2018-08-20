---
title: Golang HTTP
layout: post
comments: true
language: chinese
category: [program,golang,linux]
keywords: golang,webserver,net,http
description:
---


<!-- more -->



如下是一个最简单的示例。

{% highlight go %}
package main

import (
        "fmt"
        "net/http"
)

func IndexHandler(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "hello world")
}

func FoobarHandler(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hi, foobar\n"))
}

func main() {
        http.HandleFunc("/", IndexHandler)
        http.Handle("/foobar", http.HandlerFunc(FoobarHandler))
        http.ListenAndServe("127.0.0.1:8000", nil)
}
{% endhighlight %}

然后可以通过 `curl http://127.0.0.1:8000` 访问。

## 源码解析

对应的代码在 `$GOPATH/src/net/http` 中实现。

在 `server.go` 中，有默认的 Mux 实现，也就是 `DefaultServeMux` 。

{% highlight go %}
var DefaultServeMux = &defaultServeMux
var defaultServeMux ServeMux
{% endhighlight %}

如果用户没有创建自己的 MUX 那么实际使用的就是该变量。

### Handler注册

默认调用的 `Handler()` 以及 `HandleFunc()` 实现如下。

{% highlight go %}
func Handle(pattern string, handler Handler) {
	DefaultServeMux.Handle(pattern, handler)
}

func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
        DefaultServeMux.HandleFunc(pattern, handler)
}
{% endhighlight %}

而根据如上的定义可知，真正调用的是 `ServeMux` 中的实现，上述两个接口真正调用的实际上是 `ServeMux.Handle()` 函数。

而真正有效的是 `mux.m[pattern] = muxEntry{h: handler, pattern: pattern}` 这行代码。

### 监听服务端

在注册完用户需要的 Handler 之后，就开始调用 `http.ListenAndServe()` 监听端口并处理请求。

除此之外，还提供了 TLS 版本的 `http.ListenAndServeTLS()` 接口。

{% highlight text %}
ListenAndServe()
 |-net.Listen() 开始监听
 |-Server.Serve()
   |-Accetp()
   |-Server.newConn() 新建链接
   |-conn.serve() 启动单独的协程开始处理
     |-serverHandler.ServeHTTP() 真正的处理
       |-ServeMux.ServeHTTP() 如果没有定义，那么使用默认的MUX
       | |-ServeMux.Handler() 这里会从Request.URL.Path中取出请求的路径
       | | |-ServeMux.handler()
       | |   |-Servemux.match() 匹配map表中的路由Entry规则，也就是通过Handle()/HandleFunc()注册的回调函数
	   | |-Handler.ServeHTTP() 这里实际上已经实现了MAP的功能，也就是路由
       |-Handler.ServeHTTP() 如果在初始化时有自定义的Handler，这里会调用用户的代码
{% endhighlight %}

对于用户自定义的实现，有两种方式，一种是在新建 http.Server 时设置 Handler 成员，此时需要同时处理 URI 的路由规则。

另外一种，也是最常用的，自定义 MUX 以及 Handler 规则。

## 服务端

对于服务器来说，在接收请求的过程中，最重要的就是路由 (Router)，也就是实现一个 Multiplexer 器。

其中有一个内置的 DefautServeMux，当然也可以自定义，其目的就是为了找到真正的处理函数 (Handler)，并构建 Response 。

### Handler

<!--
用来生成 HTTP 响应的头和正文，实际上

。任何满足了http.Handler接口的对象都可作为一个处理器。通俗的说，对象只要有个如下签名的ServeHTTP方法即可：
-->

<!--
https://segmentfault.com/a/1190000006812688
-->

{% highlight go %}
package main

import (
        "fmt"
        "net/http"
        "time"
)

type customHandler struct {
}

func (cb *customHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Custom Handler!!! URI %s", r.URL.Path)
}

func main() {
        var server *http.Server = &http.Server{
                Addr:           ":8080",
                Handler:        &customHandler{},
                ReadTimeout:    10 * time.Second,
                WriteTimeout:   10 * time.Second,
                MaxHeaderBytes: 1 << 20,
        }
        server.ListenAndServe()
}
{% endhighlight %}

## 参考

<!--
https://www.alexedwards.net/blog/a-recap-of-request-handling
https://blog.csdn.net/xxb249/article/details/80779577
https://www.codetd.com/article/1766635

https://gowebexamples.com/http-server/
-->

为了防止由于压力过大导致雪崩，可以限制客户端的数量，详细可以参考 `golang.org/x/net/netutil/listen.go` 中关于 `LimitListener` 的实现。

{% highlight go %}
{% endhighlight %}
