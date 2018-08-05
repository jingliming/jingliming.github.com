---
title: gRPC 简介
layout: post
comments: true
language: chinese
category: [misc]
keywords: grpc,golang
description:
---

gRPC 一开始由 google 开发，是一款开源的远程过程调用 (RPC) 系统。

在 gRPC 里客户端应用可以像调用本地对象一样直接调用另一台不同的机器上服务端应用的方法，能够更容易地创建分布式应用和服务。

<!-- more -->

![grpc introduce]({{ site.url }}/images/go/grpc-introduce.png "grpc introduce"){: .pull-center width="70%" }

## HTTP2

### HTTP/1.x

`HTTP/1.x` 协议是一个文本协议，可读性好但是不高效。

#### Parser

如果要解析一个完整的 HTTP 请求，首先需要能正确的读出 `HTTP header`；其各个 fields 使用 `\r\n` 分隔，跟 body 之间使用 `\r\n\r\n` 分隔；然后从 header 里面的 `content-length` 拿到 body 的 size，从而读取 body。

可以参考 [http-parse](https://github.com/nodejs/http-parser) 中的实现。

#### Request/Response

在交互时，每个连接只能一问一答，客户端发送了请求之后必须等待响应之后才能继续发送下一次请求。机制简单但是网络利用率不高，需要大量交互时通常需要保持长连接。

#### Push

1.x 是没有推送机制的，通常使用 `long polling` 或者 `Web-Socket` 方式，而后者严格意义上来说已经不再属于 HTTP 了。

### HTTP/2

这是一个二进制协议，这也就意味着它的可读性几乎为 0，但幸运的是，我们还是有很多工具，譬如 Wireshark， 能够将其解析出来。


<!--
在了解 HTTP/2 之前，需要知道一些通用术语：

Stream(数据流) 双向流，一个连接可以有多个数据流，有唯一的标识符和可选的优先级信息；
Message(消息) 也就是逻辑上面的请求和响应，可以包含多个帧；
Frame(帧) 数据传输的最小单位，可以交错发送，然后在对端进行重新组装。
-->

## gRPC 简介

关于 gRPC 中如何使用的 HTTP2 协议，可以参考 [gRPC over HTTP2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) 。

分别是 Unary 以及 Stream 两种模式，前者是最简单的方式，一次请求一次响应。

当使用流模式时，可以向服务器或者客户端发送批量的数据，服务器和客户端在接收这些数据的时候，不必等接收到全部消息才开始处理，而是接收到第一条消息的时就可以立即处理，显然比类 HTTP 1.1 的方式响应更快。

例如，有一批个人的收入记录，发送给服务器计算个人所得税，那么两端都可以以流式发送，从而两端并行计算。

总计有四种方式：

1. 简单模式。客户端使用存根发送请求到服务器并等待响应返回，就像平常的函数调用。
2. 服务端流式。客户端发送请求到服务器，拿到一个流去读取返回的消息序列，此时客户端需要一直读取直到没有任何消息。
3. 客户端流式。客户端写入一个消息序列并将其发送到服务器，一旦客户端完成写入消息，它等待服务器完成读取返回它的响应。
4. 双向流式。双方使用读写流去发送一个消息序列，两个流独立操作，客户端和服务器可以以任意喜欢的顺序读写。

依次对应的声明为。

{% highlight text %}
rpc GetFeature(Point) returns (Feature) {}
rpc ListFeatures(Rectangle) returns (stream Feature) {}
rpc RecordRoute(stream Point) returns (RouteSummary) {}
rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
{% endhighlight %}

### 安装 GoLang 版本 gRPC

{% highlight text %}
----- 查看当前golang编译器版本
$ go version

----- 通过在线方式安装，-u 更新到最新版本，-v 显示详细信息
$ go get -u -v  google.golang.org/grpc

----- 离线下载解压到如下目录，并安装
$ echo $GOPATH/src
$ go install google.golang.org/grpc
{% endhighlight %}

离线版本可以从 [GitHub gRPC](https://github.com/grpc/grpc-go/releases) 上下载，其中会有一堆的依赖，依次安装即可，其中 `golang.org/x/net` 与 [GitHub Net](https://github.com/golang/net) 相同，`golang.org/x/text` 与 [GitHub Text](https://github.com/golang/text/releases) 相同。

<!--
google.golang.org/genproto/googleapis/rpc/status
https://github.com/google/go-genproto
-->

#### 安装 protoc 及其插件

{% highlight text %}
----- 查看 protoc 是否安装，确保是3.0版本
$ which protoc
$ protoc --version

----- 安装插件
$ go install github.com/golang/protobuf/proto
$ go install github.com/golang/protobuf/protoc-gen-go

----- 测试是否安装成功
$ protoc -I helloworld/ helloworld/helloworld.proto --go_out=plugins=grpc:helloworld
{% endhighlight %}

如果上述的 protoc 版本有误，可以从 [GitHub ProtoBuf](https://github.com/google/protobuf/releases) 上下载，然后添加到 `$GOPATH/bin` 目录下(需确保该路径在PATH前面)。

#### 运行示例

{% highlight text %}
$ cd $GOPATH/src/google.golang.org/grpc/examples
$ go run greeter_server/main.go
$ go run greeter_client/main.go
{% endhighlight %}

### 示例代码

假设有如下的目录结构。

{% highlight text %}
client.go
server.go
proto/
 |-helloworld.proto
{% endhighlight %}

通过如下命令生成 go 文件。

{% highlight text %}
$ mkdir proto/helloworld
$ protoc -I proto proto/helloworld.proto --go_out=plugins=grpc:proto/helloworld
{% endhighlight %}

其中 `helloworld.proto`、`client.go` 和 `server.go` 的代码为。

{% highlight text %}
syntax = "proto3";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
{% endhighlight %}

{% highlight text %}
package main

import (
        "log"
        "os"
        "time"

        "golang.org/x/net/context"
        "google.golang.org/grpc"
        pb "./proto/helloworld"
)

const (
        address     = "localhost:50051"
        defaultName = "world"
)

func main() {
        // Set up a connection to the server.
        conn, err := grpc.Dial(address, grpc.WithInsecure())
        if err != nil {
                log.Fatalf("did not connect: %v", err)
        }
        defer conn.Close()
        c := pb.NewGreeterClient(conn)

        // Contact the server and print out its response.
        name := defaultName
        if len(os.Args) > 1 {
                name = os.Args[1]
        }
        ctx, cancel := context.WithTimeout(context.Background(), time.Second)
        defer cancel()
        r, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})
        if err != nil {
                log.Fatalf("could not greet: %v", err)
        }
        log.Printf("Greeting: %s", r.Message)
}
{% endhighlight %}

{% highlight text %}
package main

import (
        "log"
        "net"

        pb "./proto/helloworld"
        "golang.org/x/net/context"
        "google.golang.org/grpc"
        "google.golang.org/grpc/reflection"
)

const (
        addr = "127.0.0.1:50051"
)

// server is used to implement helloworld.GreeterServer.
type server struct{}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
        return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}

func main() {
        lis, err := net.Listen("tcp", addr)
        if err != nil {
                log.Fatalf("failed to listen: %v", err)
        }

        s := grpc.NewServer()
        pb.RegisterGreeterServer(s, &server{})
        // Register reflection service on gRPC server.
        reflection.Register(s)
        if err := s.Serve(lis); err != nil {
                log.Fatalf("failed to serve: %v", err)
        }
}
{% endhighlight %}

如果要简单运行，可以直接执行如下命令(环境变量PWD是内置的)。

{% highlight text %}
$ GOPATH=$GOROOT:$PWD go run server.go
$ GOPATH=$GOROOT:$PWD go run client.go
{% endhighlight %}

或者直接将两个文件进行编译。

{% highlight text %}
$ go build client.go
$ go build server.go
{% endhighlight %}

因为前者引入了编译的过程，所以看起来运行的比较慢。

另外也可以参考 [GitHub - gRPC Hello Service](https://github.com/kelseyhightower/grpc-hello-service) 以及 [GitHub - gRPC Examples](https://github.com/jergoo/go-grpc-example)，后者提供了一系列的介绍，包括鉴权、拦截器、Trace 等。

<!--
## 反射

关于gRPC中的Reflection可以参考
https://chromium.googlesource.com/external/github.com/grpc/grpc-go/+/refs/heads/master/Documentation/server-reflection-tutorial.md
-->

## 拦截器 Interceptor

可以在服务端接收到请求后，先对请求中的数据做一些处理后再转交给指定的服务处理并响应，常见的如权限校验、日志、接口调用延迟等，这里简单打印下客户端的 IP 地址。

### 获取IP

gRPC 服务和客户端之间是通过 http2 进行交互，其中包含了客户端的地址信息，

在 gRPC 源码 `peer/peer.go` 中包含了创建的上下文信息，其中就记录的远端地址；而且在 gRPC 请求中默认都会含有 Context 值，这样就可以通过如下方法获取。

{% highlight text %}
func getClietIP(ctx context.Context) (string, error) {
	pr, ok := peer.FromContext(ctx)
	if !ok {
		return "", fmt.Errorf("getClinetIP, invoke FromContext() failed")
	}
	if pr.Addr == net.Addr(nil) {
		return "", fmt.Errorf("getClientIP, peer.Addr is nil")
	}
	addSlice := strings.Split(pr.Addr.String(), ":")
	return addSlice[0], nil
}
{% endhighlight %}

注意：在使用 stream 方式时 context 值可以直接从 stream 中获取，也就是 `stream.Context()` 。

### 增加拦截器

将服务端的代码修改为如下。

{% highlight text %}
package main

import (
        "fmt"
        "log"
        "net"

        pb "./proto/helloworld"
        "golang.org/x/net/context"
        "google.golang.org/grpc"
        "google.golang.org/grpc/peer"
        "google.golang.org/grpc/reflection"
)

const (
        port = ":50051"
)

// server is used to implement helloworld.GreeterServer.
type server struct{}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
        return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}

func getClietIP(ctx context.Context) (string, error) {
        pr, ok := peer.FromContext(ctx)
        if !ok {
                return "", fmt.Errorf("getClinetIP, invoke FromContext() failed")
        }
        if pr.Addr == net.Addr(nil) {
                return "", fmt.Errorf("getClientIP, peer.Addr is nil")
        }

        return pr.Addr.String(), nil
}

func main() {
        lis, err := net.Listen("tcp", port)
        if err != nil {
                log.Fatalf("failed to listen: %v", err)
        }

        var opts []grpc.ServerOption

        // Register interceptor
        var interceptor grpc.UnaryServerInterceptor
        interceptor = func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo,
			handler grpc.UnaryHandler) (resp interface{}, err error) {
                cli, err := getClietIP(ctx)
                if err != nil {
                        log.Println("Failed to get client address")
                }
                log.Println("Client address is", cli)
                return handler(ctx, req)
        }
        opts = append(opts, grpc.UnaryInterceptor(interceptor))

        s := grpc.NewServer(opts...)
        pb.RegisterGreeterServer(s, &server{})
        // Register reflection service on gRPC server.
        reflection.Register(s)
        if err := s.Serve(lis); err != nil {
                log.Fatalf("failed to serve: %v", err)
        }
}
{% endhighlight %}





## Stream

普通的 gRPC 是直接返回一个定义的 HelloReply 对象，而流式响应可以通过 Send 方法返回多个 HelloReply 对象，对象流序列化后流式返回。

目录结构同上。

{% highlight text %}
$ mkdir proto/helloworld
$ protoc -I proto proto/helloworld.proto --go_out=plugins=grpc:proto/helloworld
{% endhighlight %}

其中 `helloworld.proto`、`client.go` 和 `server.go` 的代码为。

{% highlight text %}
syntax = "proto3";

package helloworld;

service Math {
        rpc Max (stream Request) returns (stream Response) {}
}

message Request {
        int32 num = 1;
}

message Response {
        int32 result = 1;
}
{% endhighlight %}

{% highlight text %}
package main

import (
        "context"
        "io"
        "log"
        "math/rand"
        "time"

        pb "./proto/helloworld"
        "google.golang.org/grpc"
)

func main() {
        rand.Seed(time.Now().Unix())

        conn, err := grpc.Dial(":50005", grpc.WithInsecure())
        if err != nil {
                log.Fatalf("can not connect with server %v", err)
        }

        // create stream
        client := pb.NewMathClient(conn)
        stream, err := client.Max(context.Background())
        if err != nil {
                log.Fatalf("openn stream error %v", err)
        }

        var max int32
        ctx := stream.Context()
        done := make(chan bool)

        // first goroutine sends random increasing numbers to stream
        // and closes it after 10 iterations
        go func() {
                for i := 1; i <= 10; i++ {
                        // generate random nummber and send it to stream
                        rnd := int32(rand.Intn(i))
                        req := pb.Request{Num: rnd}
                        if err := stream.Send(&req); err != nil {
                                log.Fatalf("can not send %v", err)
                        }
                        log.Printf("%d sent", req.Num)
                        time.Sleep(time.Millisecond * 200)
                }
                if err := stream.CloseSend(); err != nil {
                        log.Println(err)
                }
        }()

        // second goroutine receives data from stream
        // and saves result in max variable
        //
        // if stream is finished it closes done channel
        go func() {
                for {
                        resp, err := stream.Recv()
                        if err == io.EOF {
                                close(done)
                                return
                        }
                        if err != nil {
                                log.Fatalf("can not receive %v", err)
                        }
                        max = resp.Result
                        log.Printf("new max %d received", max)
                }
        }()

        // third goroutine closes done channel if context is done
        go func() {
                <-ctx.Done()
                if err := ctx.Err(); err != nil {
                        log.Println(err)
                }
                close(done)
        }()

        <-done
        log.Printf("finished with max=%d", max)
}
{% endhighlight %}

{% highlight text %}
package main

import (
        "io"
        "log"
        "net"

        pb "./proto/helloworld"
        "google.golang.org/grpc"
)

type server struct{}

func (s *server) Max(srv pb.Math_MaxServer) error {
        log.Println("Start new math server")
        var max int32
        ctx := srv.Context()

        for {
                // exit if context is done or continue
                select {
                case <-ctx.Done():
                        return ctx.Err()
                default:
                }

                // receive data from stream
                req, err := srv.Recv()
                if err == io.EOF {
                        // return will close stream from server side
                        log.Println("exit")
                        return nil
                }
                if err != nil {
                        log.Printf("receive error %v", err)
                        continue
                }

                // continue if number reveived from stream less than max
                if req.Num <= max {
                        continue
                }

                // update max and send it to stream
                max = req.Num
                resp := pb.Response{Result: max}
                if err := srv.Send(&resp); err != nil {
                        log.Printf("send error %v", err)
                }
                log.Printf("send new max=%d", max)
        }
}

func main() {
        lis, err := net.Listen("tcp", ":50005")
        if err != nil {
                log.Fatalf("failed to listen: %v", err)
        }

        s := grpc.NewServer()
        pb.RegisterMathServer(s, &server{})
        reflection.Register(s)
        if err := s.Serve(lis); err != nil {
                log.Fatalf("failed to serve: %v", err)
        }
}
{% endhighlight %}


## 其它

### 监听地址

在调用 `net.Listen()` 时，如果通过 `:8080` 方式指定端口，那么可能会监听到 IPv6 地址上，如果要使用 IPv4 那么需要显示指定 `127.0.0.1:8080` 。



## 参考

[High Performance Browser Networking](https://hpbn.co/http2/) 或者 [本地文档](/reference/linux/http2_protocol.html)。


<!--
HTTP2 的协议详解
https://developers.google.com/web/fundamentals/performance/http2/?hl=zh-cn


Protocol Buffer 序列化原理大揭秘
https://blog.csdn.net/carson_ho/article/details/70568606
https://github.com/grpc/grpc/blob/master/doc/health-checking.md
https://github.com/go-training/grpc-health-check
-->

{% highlight text %}
{% endhighlight %}
