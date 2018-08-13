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

## 插件实现

Google 的 Protocol Buffer 是一种以二进制格式对消息进行编码和解码的库，目前支持大部分的语言，如果要实现其它语言，需要先实现一个 protoc 编译器的插件。

CentOS 可通过 `yum install protobuf protobuf-compiler` 安装解析器，然后通过如下方式获取相应分支的源码：

{% highlight text %}
$ git clone https://github.com/google/protobuf
$ git checkout v2.5.0
{% endhighlight %}

### Python 插件

这里使用如下的一个 proto 文件 `hello.proto`。

{% highlight text %}
enum Greeting {
	NONE = 0;
	MR = 1;
	MRS = 2;
	MISS = 3;
}
message Hello {
	required Greeting greeting = 1;
	required string name = 2;
}
{% endhighlight %}

编译器 protoc 的接口非常简单，编译器将在 stdin 上传递一个 CodeGeneratorRequest 消息，你的插件将在 stdout 的 CodeGeneratorResponse 中输出生成的代码。

第一步是编写读取请求的代码并写一个空的响应：

{% highlight python %}
#!/usr/bin/env python

import sys
from google.protobuf.compiler import plugin_pb2 as plugin

def generate_code(request, response):
    pass

if __name__ == '__main__':
    # Read request message from stdin
    data = sys.stdin.read()
    # Parse request
    request = plugin.CodeGeneratorRequest()
    request.ParseFromString(data)
    # Create response
    response = plugin.CodeGeneratorResponse()
    # Generate code
    generate_code(request, response)
    # Serialise response message
    output = response.SerializeToString()
    # Write to stdout
    sys.stdout.write(output)
{% endhighlight %}

<!--
编译器 protoc 遵循关于插件名称的命名约定，可以在 PATH 中将代码保存在名为 protoc-gen-custom 的文件中，或使用你喜欢的任何名称保存，并将插件的名称和路径传递给 `--plugin` 命令行选项。


让我们修改 generate_code() 函数来生成 .proto 文件的JSON表示。在这篇文章中，根据之前我们定义的proto文件内容，首先我们需要一个遍历AST并返回所有枚举，消息和嵌套类型的函数：

def traverse(proto_file):
    def _traverse(package, items):
        for item in items:
            yield item, package
            if isinstance(item, DescriptorProto):
                for enum in item.enum_type:
                    yield enum, package
                for nested in item.nested_type:
                    nested_package = package + item.name
                    for nested_item in _traverse(nested, nested_package):
                        yield nested_item, nested_package
    return itertools.chain(
        _traverse(proto_file.package, proto_file.enum_type),
        _traverse(proto_file.package, proto_file.message_type),
    )

现在 generate_code() 代码如下所示：

import itertools
import json
from google.protobuf.descriptor_pb2 import DescriptorProto, EnumDescriptorProto
def generate_code(request, response):
    for proto_file in request.proto_file:
        output = []
        # Parse request
        for item, package in traverse(proto_file):
            data = {
                'package': proto_file.package or '&lt;root&gt;',
                'filename': proto_file.name,
                'name': item.name,
            }
            if isinstance(item, DescriptorProto):
                data.update({
                    'type': 'Message',
                    'properties': [{'name': f.name, 'type': int(f.type)}
                                   for f in item.field]
                })
            elif isinstance(item, EnumDescriptorProto):
                data.update({
                    'type': 'Enum',
                    'values': [{'name': v.name, 'value': v.number}
                               for v in item.value]
                })
            output.append(data)
        # Fill response
        f = response.file.add()
        f.name = proto_file.name + '.json'
        f.content = json.dumps(output, indent=2)

对于 protoc 请求中的每个 .proto 文件，我们遍历所有项目（枚举，消息和嵌套类型），并在字典中写入一些信息。然后，我们向 protoc 的响应中添加一个新文件，我们设置文件名，在该例中等于原始文件名加上 .json 扩展名，以及字典的JSON表示形式的内容。

如果再次运行protobuf编译器，它将在build目录中输出一个名为 hello.proto.json 的文件，内容如下：
-->

核心部分是从 stdin 读取 protoc 请求的接口代码，遍历 AST 并在 stdout 上写入响应。




<!--
这里的插件扩展以 `protoc-gen-go` 插件为例。

https://tunsuy.github.io/2017/02/20/%E4%B8%BAProtobuf%E7%BC%96%E8%AF%91%E5%99%A8protoc%E7%BC%96%E5%86%99%E6%8F%92%E4%BB%B6/
https://hitzhangjie.github.io/2017/05/23/Protoc%E5%8F%8A%E6%8F%92%E4%BB%B6%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90(%E7%B2%BE%E5%8D%8E%E7%89%88).html


{% highlight text %}
$ protoc --plugin=protoc-gen-custom=plugin.py --custom_out=./build hello.proto
{% endhighlight %}
-->








## 参考

其它相关的 C 实现可以参考 [Nanopb protocol buffers with small code size](http://jpa.kapsi.fi/nanopb/) 中的实现，另外还有 minipb 。

<!--
Protobuf 语言指南
https://blog.csdn.net/cchd0001/article/details/50669079

Protocol Buffer 序列化原理大揭秘
https://blog.csdn.net/carson_ho/article/details/70568606
-->


{% highlight text %}
{% endhighlight %}
