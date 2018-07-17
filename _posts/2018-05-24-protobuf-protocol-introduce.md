---
title: Protobuf 协议简介
layout: post
comments: true
language: chinese
category: [misc]
keywords: protobuf
description: Protobuf (Google Protocol Buffers) 是 google 开发的的一套用于数据存储，网络通信时用于协议编解码的工具库，与 XML 和 JSON 数据格式类似，但采用的是二进制的数据格式，具有更高的传输，打包和解包效率。
---

Protobuf (Google Protocol Buffers) 是 google 开发的的一套用于数据存储，网络通信时用于协议编解码的工具库，与 XML 和 JSON 数据格式类似，但采用的是二进制的数据格式，具有更高的传输，打包和解包效率。

<!-- more -->

## 源码安装

源码安装最新版本的 protoc 。

{% highlight text %}
----- 如果非Release版本需要执行如下脚本生成configure文件
$ ./autogen.sh
----- 指定安装路径
$ ./configure --prefix=/usr/local/protobuf
$ make
----- 如下测试流程会很耗时，可以忽略
$ make check
# make install
{% endhighlight %}

`protobuf-c` 最新版本是支持 proto3 的，如果平台没有对应的包，那么可以直接从源码开始编译。

{% highlight text %}
PKG_CONFIG_PATH=/usr/local/protobuf/lib/pkgconfig \
./configure --includedir=/usr/local/protobuf/include/google/protobuf --prefix=/usr/local/protobuf-c
{% endhighlight %}

注意，通过上述命令安装完 protobuf-c 之后，头文件实际会安装在 `/usr/local/protobuf/include/google/protobuf/protobuf-c` 目录下。

因为按转到了非标准目录下，在编译时就需要通过 `-I` 和 `-L` 参数指定目录路径；包括在执行时添加 `LD_LIBRARY_PATH=/usr/local/protobuf-c/lib/` 变量。


### 示例

如下是一个示例，相关的代码也可以参考 [github protobuf-c examples](https://github.com/protobuf-c/protobuf-c/wiki/Examples) 。

{% highlight text %}
/* request.proto */
syntax = "proto2";

message Request {
        required string hostname = 1;

        enum ProtocolType {
                ICMP = 0;
                TCP = 1;
        };
        required ProtocolType protocol = 2;
        optional uint32 interval = 3;
        optional uint32 timeout = 4;
};
{% endhighlight %}

测试代码如下。

{% highlight c %}
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include "request.pb-c.h"

/* NOTE: Keep this same with request.proto */
enum {
        PROTOCOL_TYPE_ICMP = 0,
        PROTOCOL_TYPE_TCP  = 0
};

int main()
{
        Request req = REQUEST__INIT, *request;
        unsigned char *buffer = NULL;
        size_t len ;

        req.hostname = "192.168.9.10";
        req.protocol = PROTOCOL_TYPE_ICMP;
        req.has_interval = 1;
        req.interval = 10;

        len = request__get_packed_size(&req);
        fprintf(stdout, "Packed size is %lu\n", len);
        assert(len);
        buffer = (unsigned char *)malloc(len);
        if (buffer == NULL) {
                fprintf(stderr, "Out of memory\n");
                return -1;
        }
        request__pack(&req, buffer);

        request = request__unpack(NULL, len, buffer);
        if (request == NULL) {
                fprintf(stderr, "error unpacking incoming message\n");
                return -1;
        }

        printf("   hostname = %s\n", request->hostname);
        printf("   protocol = %d\n", request->protocol);
        if (request->has_interval)
                printf("   interval = %d\n", request->interval);

        request__free_unpacked(request, NULL);
        free(buffer);

        return 0;
}
{% endhighlight %}

`Makefile` 编译。

{% highlight text %}
OUTDIR=proto-out
PROTOFILE=request

all: pre example

pre:
        mkdir -p ${OUTDIR}

example: example.c
        protoc-c --c_out=${OUTDIR} ${PROTOFILE}.proto
        gcc -Wall -I${OUTDIR} ${OUTDIR}/${PROTOFILE}.pb-c.c $^ -o $@ -lprotobuf-c

clean:
        rm -rf ${OUTDIR} example
{% endhighlight %}

### 版本 V3

V2 和 V3 的改动包括了：

1. 只保留 repeated 来标记数组类型，而 optional 和 required 都被去掉了；
2. 删除 default，当序列化和反序列化端有不同的默认值时，会导致最终的结果不一致；

另外，在 V3 的版本其中有个 oneof 特性，而 protobuf-c 的实现比较坑，没有提供对应的 API 接口，需要进行手动设置。例如：

{% highlight text %}
message Request {
	oneof addr {
		string state = 1;
		string contry = 2;
	};
};
{% endhighlight %}

那么在进行序列化时，需要通过如下方式设置。

{% highlight text %}
Request req = REQUEST__INIT;
req.addr_case = REQUEST__ADDR_STATE;
req.state = "china";
{% endhighlight %}

而在进行反序列化时，可以通过 switch 进行判断，例如：

{% highlight text %}
switch (request->addr_case) {
case REQUEST__ADDR_STATE:
	printf("Got state: %s\n", request->state);
	break;
case REQUEST__ADDR__NOT_SET:
default:
	break;
}
{% endhighlight %}

## 参考

其它相关的 C 实现可以参考 [Nanopb protocol buffers with small code size](http://jpa.kapsi.fi/nanopb/) 中的实现。

<!--
https://blog.csdn.net/cchd0001/article/details/50669079
-->


{% highlight text %}
{% endhighlight %}
