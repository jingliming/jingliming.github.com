---
title: ETCD 简介
layout: post
comments: true
language: chinese
category: [program,golang,linux]
keywords: golang,go,etcd
description: Etcd 是一个分布式可靠的键值存储系统，提供了与 ZooKeeper 相似的功能，通过 GoLang 开发而非 Java ，采用 RAFT 算法而非 PAXOS 算法。相比来所，etcd 的安装使用更加简单有效。
---

Etcd 是一个分布式可靠的键值存储系统，提供了与 ZooKeeper 相似的功能，通过 GoLang 开发而非 Java ，采用 RAFT 算法而非 PAXOS 算法。

相比来所，etcd 的安装使用更加简单有效。

<!-- more -->

## 简介

A distributed, reliable key-value store for the most critical data of a distributed system.

严格来说，ETCD 主要用于保存一些元数据信息，一般小于 1GB 对大于 1GB 的可以使用新型的分布式数据库，例如 TiDB 等，通常适用于 CP 场景。

## 安装

可以直接从 [github release](https://github.com/coreos/etcd/releases) 下载非源码包，也就是已经编译好的二进制包，一般包括了 etcd + etcdctl 。

### 源码安装

下载 ectd 源码构建，在源码中，实际上已经包含了工程所使用的库，在编译时可以直接修改 build 脚本，例如对于 raftexample 的编译，在该脚本中会设置一堆的环境变量，以引用本项目中的三方库。

{% highlight text %}
----- 需要go编译器支持，设置好GOPATH环境变量
$ go version
$ echo $GOPATH

----- 新建目录并下载代码，并编译
$ mkdir -p $GOPATH/src/github.com/coreos
$ cd $GOPATH/src/github.com/coreos
$ git clone https://github.com/coreos/etcd.git
$ cd etcd && git checkout v3.1.0
$ ./build
$ ./bin/etcd
{% endhighlight %}

### 单机单进程测试

启动单进程服务，并进行测试。

{% highlight text %}
----- 启动单个本地进程，会监听127.1:2379端口
$ ./etcd

----- 使用API v3版本，并测试添加获取参数
$ export ETCDCTL_API=3
$ ./etcdctl put foo bar
OK
$ ./etcdctl get foo
foo
bar

$ ./etcdctl --write-out=table --endpoints=localhost:2379 member list

----- 只打印值信息，不打印key
$ ./etcdctl get foo --print-value-only
bar
----- 打印十六进制格式
$ ./etcdctl get foo --hex
\x66\x6f\x6f
\x62\x61\x72
----- 指定范围为foo~foo3
$ ./etcdctl get foo foo3
foo
foo1
foo2
foo3
----- 指定前缀，且只显示前两个
$ ./etcdctl get --prefix --limit=2 foo
foo
foo1
{% endhighlight %}



<!--
--name
  集群中节点名，可区分且不重复即可；
--listen-peer-urls
监听的用于节点之间通信的url，可监听多个，集群内部将通过这些url进行数据交互(如选举，数据同步等)
--initial-advertise-peer-urls
建议用于节点之间通信的url，节点间将以该值进行通信。
--listen-client-urls
监听的用于客户端通信的url,同样可以监听多个。
--advertise-client-urls
建议使用的客户端通信url,该值用于etcd代理或etcd成员与etcd节点通信。
--initial-cluster-token etcd-cluster-1
节点的token值，设置该值后集群将生成唯一id,并为每个节点也生成唯一id,当使用相同配置文件再启动一个集群时，只要该token值不一样，etcd集群就不会相互影响。
--initial-cluster
也就是集群中所有的initial-advertise-peer-urls 的合集
--initial-cluster-state new
新建集群的标志

./etcd --name infra0 --initial-advertise-peer-urls http://10.0.1.111:2380 \
  --listen-peer-urls http://10.0.1.111:2380 \
  --listen-client-urls http://10.0.1.111:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.111:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://10.0.1.111:2380,infra1=http://10.0.1.109:2380,infra2=http://10.0.1.110:2380 \
  --initial-cluster-state new

./etcd --name infra1 --initial-advertise-peer-urls http://10.0.1.109:2380 \
  --listen-peer-urls http://10.0.1.109:2380 \
  --listen-client-urls http://10.0.1.109:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.109:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://10.0.1.111:2380,infra1=http://10.0.1.109:2380,infra2=http://10.0.1.110:2380 \
  --initial-cluster-state new

./etcd --name infra2 --initial-advertise-peer-urls http://10.0.1.110:2380 \
  --listen-peer-urls http://10.0.1.110:2380 \
  --listen-client-urls http://10.0.1.110:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.110:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://10.0.1.111:2380,infra1=http://10.0.1.109:2380,infra2=http://10.0.1.110:2380 \
  --initial-cluster-state new

按如上配置分别启动集群，启动集群后，将会进入集群选举状态，若出现大量超时，则需要检查主机的防火墙是否关闭，或主机之间是否能通过2380端口通信，集群建立后通过以下命令检查集群状态。

数据持久化基于多版本并发控制，


性能压测的核心指标包括了：延迟(Latency)、吞吐量(Throughput)；延迟表示完成一次请求的处理时间；吞吐量表示某个时间段内完成的请求数。

一般来说，当吞吐量增加增加时，对应的延迟也会同样增加；低负载时 1ms 以内可以完成一次请求，高负载时可以完成 3W/s 的请求。对于 ETCD 来说，会涉及到一致性以及持久化的问题，所以其性能与网络带宽、磁盘IO的性能紧密相关。

其底层的存储基于 BoltDB，一个基于 GoLang 的 KV 数据库，支持 ACID 特性，与 lmdb 相似。

https://coreos.com/etcd/docs/latest/op-guide/performance.html
-->


<!--
configuration management, service discovery, and coordinating distributed work. Many organizations use etcd to implement production systems such as container schedulers, service discovery services, and distributed data storage. Common distributed patterns using etcd include leader election, distributed locks, and monitoring machine liveness.
-->

### 单机集群测试

在搭建本地集群时，可以直接使用 goreman 工具，默认使用的是当前目录下的 Procfile 配置文件，运行前需要确保配置正确。

{% highlight text %}
----- 检查配置是否合法
$ goreman check
----- 启动，或者指定配置文件启动
$ goreman start
$ goreman -f MyProcfile start
----- 查看当前的状态
$ goreman run status
----- 停止、启动、重启某个进程(stop start restart)
$ goreman run stop PROCESS_NAME
{% endhighlight %}

简单来说，直接通过 `goreman start` 启动即可，此时会在当前目录下生成 `infra{1,2,3}.etcd` 三个目录，用于保存各个进程的信息。

## API

实际上 API 基本上决定了 etcd 提供了哪些服务，通过 HTTP API 对外提供服务，这种接口更方便各种语言对接，命令行可以使用 httpie 或者 curl 调用。

数据按照树形的结构组织，类似于 Linux 的文件系统，也有目录和文件的区别，不过一般被称为 nodes，其中数据相关的 endpoint 都是以 `/v2/keys` 开头 (v2 表示当前 API 的版本)，比如 `/v2/keys/names/cizixs` 。

要创建一个值，只要使用 PUT 方法在对应的 url endpoint 设置就行。如果对应的 key 已经存在， PUT 也会对 key 进行更新。

### CURD

{% highlight text %}
----- 不存在则创建，否则修改，当超过TTL后，会自动删除
http PUT http://127.0.0.1:2379/v2/keys/message value=="hello, etcd" ttl==5
http GET http://127.0.0.1:2379/v2/keys/message
http DELETE http://127.0.0.1:2379/v2/keys/message
{% endhighlight %}

在创建 key 的时候，如果它所在路径的目录不存在，会自动被创建，所以在多数情况下我们不需要关心目录的创建，如果要单独创建一个目录可以指定参数 `dir=true`。

{% highlight text %}
http PUT http://127.0.0.1:2379/v2/keys/anotherdir dir==true
{% endhighlight %}

注意，ECTD 提供了类似 Linux 中 `.` 开头的隐藏机制，以 `_` 开头的节点也是默认隐藏的，不会在列出目录的时候显示，只有知道隐藏节点的完整路径，才能够访问它的信息。


### 监听机制

通过监听机制，可以在某个 key 发生变化时，通知对应的客户端，主要用于服务发现，集群中有信息更新时可以被及时发现并作出相应处理。

{% highlight text %}
http http://127.0.0.1:2379/v2/keys/foo wait==true
{% endhighlight %}

使用 `recursive=true` 参数，可以用来监听某个目录。

### 比较更新

在分布式环境中，需要解决多个客户端的竞争问题，通过 etcd 提供的原子操作 CompareAndSwap (CAS)，可以很容易实现分布式锁。简单来说，这个命令只有在客户端提供的条件成立的情况下才会更新对应的值。

{% highlight text %}
http PUT http://127.0.0.1:2379/v2/keys/foo prevValue==bar value==changed
{% endhighlight %}

只有当之前的值为 bar 时，才会将其更新成 changed 。

### 比较删除

同样是原子操作，只有在客户端提供的条件成立的情况下，才会执行删除操作；支持 prevValue 和 prevIndex 两种条件检查，没有 prevExist，因为删除不存在的值本身就会报错。

{% highlight text %}
http DELETE http://127.0.0.1:2379/v2/keys/foo prevValue==bar
{% endhighlight %}

### 监控集群

Etcd 还保存了集群的数据信息，包括节点之间的网络信息，操作的统计信息。

<!--
/v2/stats/leader  会返回集群中 leader 的信息，以及 followers 的基本信息
/v2/stats/self 会返回当前节点的信息
/v2/state/store：会返回各种命令的统计信息
-->

### 成员管理

在 `/v2/members` 下保存着集群中各个成员的信息。

<!--
/version   获取版本服务器以及集群的版本号
-->


## 常见操作


### etcdctl

这个实际上是封装了 HTTP 请求的一个客户端，用于更方便的与服务端进行交互。

{% highlight text %}
----- 设置一个key的值
$ etcdctl set /message "hello, etcd"
hello, etcd

----- 获取key的值
$ etcdctl get /message
hello, etcd

----- 获取key值的同时，显示更详细的元数据信息
$ etcdctl -o extended get /message
Key: /message
Created-Index: 1073
Modified-Index: 1073
TTL: 0
Index: 1073

hello, etcd

----- 如果获取的key不存在，则会直接报错
$ etcdctl get /notexist
Error:  100: Key not found (/notexist) [1048]

----- 设置key的ttl，过期后会被自动删除
$ etcdctl set /tempkey "gone with wind" --ttl 5
gone with wind

----- 如果key的值是"hello, etcd"，就把它替换为"goodbye, etcd"
$ etcdctl set --swap-with-value "hello, world" /message "goodbye, etcd"
Error:  101: Compare failed ([hello, world != hello, etcd]) [1050]
$ etcdctl set --swap-with-value "hello, etcd" /message "goodbye, etcd"
goodbye, etcd

----- 仅当key不存在时创建
$ etcdctl mk /foo bar
bar
$ etcdctl mk /foo bar
Error:  105: Key already exists (/foo) [1052]

----- 自动创建排序的key
$ etcdctl mk --in-order /queue job1
job1
$ etcdctl mk --in-order /queue job2
job2
$ etcdctl ls --sort /queue
/queue/00000000000000001053
/queue/00000000000000001054

----- 更新key的值或者ttl，只有当key已经存在的时候才会生效，否则报错
$ etcdctl update /message "I'am changed"
I'am changed
$ etcdctl get /message
I'am changed
$ etcdctl update /notexist "I'am changed"
Error:  100: Key not found (/notexist) [1055]
$ etcdctl update --ttl 3 /message "I'am changed"
I'am changed
$ etcdctl get /message
Error:  100: Key not found (/message) [1057]

----- 删除某个key
$ etcdctl mk /foo bar
bar
$ etcdctl rm /foo
PrevNode.Value: bar
$ etcdctl get /foo
Error:  100: Key not found (/foo) [1062]

----- 只有当key的值匹配的时候，才进行删除
$ etcdctl mk /foo bar
bar
$ etcdctl rm --with-value wrong /foo
Error:  101: Compare failed ([wrong != bar]) [1063]
$ etcdctl rm --with-value bar /foo

----- 创建一个目录
$ etcdctl mkdir /dir

----- 删除空目录
$ etcdctl mkdir /dir/subdir/
$ etcdctl rmdir /dir/subdir/

----- 删除非空目录
$ etcdctl rmdir /dir
Error:  108: Directory not empty (/dir) [1071]
$ etcdctl rm --recursive /dir

----- 列出目录的内容
$ etcdctl ls /
/queue
/anotherdir
/message

----- 递归列出目录的内容
$ etcdctl ls --recursive /
/anotherdir
/message
/queue
/queue/00000000000000001053
/queue/00000000000000001054

----- 监听某个key，当key改变的时候会打印出变化
$ etcdctl watch /message
changed

----- 监听某个目录，当目录中任何node改变的时候，都会打印出来
$ etcdctl watch --recursive /
[set] /message
changed

----- 一直监听，除非CTRL + C导致退出监听
$ etcdctl watch --forever /message
new value
chaned again
Wola

----- 监听目录，并在发生变化的时候执行一个命令
$ etcdctl exec-watch --recursive / -- sh -c "echo change detected."
change detected.
change detected.

----- 检查集群的健康状态
$ etcdctl cluster-health

----- 查看集群的成员列表
$ etcdctl member list
{% endhighlight %}

**注意** 默认只保存了 1000 个历史事件，所以不适合有大量更新操作的场景，这样会导致数据的丢失，其使用的典型应用场景是配置管理和服务发现，这些场景都是读多写少的。

### ClientV3

在 ETCD 的源码目录下保存了一个 clientv3 的代码，详细可以参考 [ETCD ClientV3](https://github.com/coreos/etcd/tree/master/clientv3) 。

#### etcdctl V3

{% highlight text %}
----- 使用V3版本需要提前设置环境变量，否则etcdctl --version查看
$ ETCDCTL_API=3 ./etcdctl version
etcdctl version: 3.3.1
API version: 2

----- 查看当前集群的列表，默认使用本地2379端口，也可以通过参数指定
$ ETCDCTL_API=3 ./etcdctl member list
$ ETCDCTL_API=3 ./etcdctl --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 member list

----- CURD，可以指定输出格式、前缀匹配
$ ETCDCTL_API=3 ./etcdctl put foo "Hello World!"
$ ETCDCTL_API=3 ./etcdctl get foo
$ ETCDCTL_API=3 ./etcdctl --write-out="json" get foo
$ ETCDCTL_API=3 ./etcdctl --prefix get foo
$ ETCDCTL_API=3 ./etcdctl --prefix del foo

----- 查看集群状态
$ ETCDCTL_API=3 ./etcdctl --write-out=table endpoint status
$ ETCDCTL_API=3 ./etcdctl endpoint health

----- 管理集群成员add remove update list
$ ETCDCTL_API=3 ./etcdctl --write-out=table member list

----- 查看告警
$ ETCDCTL_API=3 ./etcdctl alarm list
{% endhighlight %}

### 压测

在源码中内置了一个压测工具 `tools/benchmark` ，类似于 raftexample ，同样可以通过修改 `build` 文件编译。

详细的使用方法可以查看源码中的文档 [Github op-guide performance](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/performance.md) 。

{% highlight text %}
$ go build -o "${out}/benchmark" ${REPO_PATH}/tools/benchmark || return
{% endhighlight %}

{% highlight text %}
----- 可以先查看当前集群的状态
$ ETCDCTL_API=3 ./etcdctl --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 \
     --write-out=table endpoint status

$ ./benchmark --endpoints=127.0.0.1:2379 --target-leader --conns=1 --clients=1 \
	put --key-size=8 --sequential-keys --total=10000 --val-size=256
{% endhighlight %}

<!--
https://indico.cern.ch/event/560399/contributions/2262460/attachments/1318051/1975404/slides.pdf
-->

## 参考


<!--
https://tonydeng.github.io/2015/10/19/etcd-application-scenarios/
http://jolestar.com/etcd-architecture/
http://www.infoq.com/cn/articles/etcd-interpretation-application-scenario-implement-principle

场景一：服务发现（Service Discovery）
场景二：消息发布与订阅
场景三：负载均衡
场景四：分布式通知与协调
场景五：分布式锁、分布式队列
场景六：集群监控与Leader竞选



EtcdServer.start()

startPeer()




mini transaction支持原子性比较多个键值并且操作多个键值。之前的CompareAndSwap实际上一个针对单个key的mini transaction。一个简单的例子是 Tx(compare: A=1 && B=2, success: C = 3, D = 3, fail: C = 0, D = 0)。当etcd收到这条transcation请求，etcd会原子性的判断A和B当前的值和期待的值。如果判断成功，C和D的值会被设置为3。

-I 只显示头
-i 显示头以及返回信息

curl -i http://127.0.0.1:2379/version
curl -i http://127.0.0.1:2380/members

ETCD V3 对外提供的 API 接口保存在 `etcdserver/api/etcdhttp` 目录下，常见的 URI 包括：
/debug/vars 获取调试信息，包括启动参数、资源使用等
/version 当前版本信息
/health 服务器的健康状态

V2 VS. V3
https://www.compose.com/articles/etcd2to3-new-apis-and-new-possibilities/
https://blog.gopheracademy.com/advent-2015/etcd-distributed-key-value-store-with-grpc-http2/
https://github.com/go-up/go-example
https://doc.oschina.net/grpc?t=60133
https://grpc.io/docs/quickstart/go.html

V2 版本只提供了基本的原子操作 CAS(Compare And Swap)，并在此基础上实现分布式锁；在 V3 上则实现了 MVCC 事务模型，抛弃了原来的原子操作。

那么其并发事务的隔离模型是怎样的？如何处理提交时的冲突？

实际上是直接把 MVCC 的版本机制暴露给了用户，在事务提交冲突时完全由用户控制是回滚还是忽略冲突直接提交，从而给用户以最大的灵活性，这也就是 STM 的代码。

software transactional memory

Serializable reads

Linearized Reads

如果要保证线性读，那么客户端需要从主上读取数据。

## 一致性模型

简单来说，可以根据不同的场景定义模型，从而使写的程序可预测。

Strong consistency models，一步步介绍一致性模型
https://aphyr.com/posts/313-strong-consistency-models

## 参考
Serializability and Distributed Software Transactional Memory with etcd3
https://coreos.com/blog/transactional-memory-with-etcd3.html
http://lday.me/2017/02/01/0003_seri-stm-etcd3/


相比 V2 而言 V3 提供了 gRPC 通讯，对于不支持 gRPC 的语言，etcd 提供 JSON 的 grpc-gateway 网关，作为 RESTful 代理，翻译 HTTP/JSON 请求为 gRPC 消息。

https://leonlee110.github.io/kubernetes/2018/03/31/learning-etcd-by-code-1

embed.StartEtcd()
EtcdServer.Start()
EtcdServer.servePeers()
 |-serve()
   |-grpc.NewServer() 新建一个gRPC的服务器etcdserver/api/v3rpc/grpc.go
EtcdServer.serveClients()



在 `type Node interface` 中定义了一些与 RAFT 核心相关的接口函数，

Propose() 用来提交日志
Ready() 返回一个Ready管道，表示已经提交的日志

在 raft/node.go 中会启动一个后台程序，对应了 `func (n *node) run(r *raft)` 函数，

在 ETCD RAFT 的实现中有两个核心的数据结构 `type node struct` 以及 `type raft struct` ，前者定义了用于 RAFT 核心与应用层的通讯的通讯管道，后者则保存了 RAFT 协议中需要保存的状态信息。


在 etcdserver/server.go 文件中定义了 type EtcdServer struct 结构体，保存了与 ETCD 的服务端相关的设置。注意，这里涉及到了一堆的继承关系，EtcdServer -> raftNodeConfig -> raft.Node ，也就是 Propose 对应了 type Node interface 中的函数接口。



processInternalRaftRequestOnce() V3接口提交日志
 |-EtcdServer.r.Propose() 实际上是raftNode中


数据的提交分成了 4 步：

1. 数据写入到Leader的本地存储，一般是内存；
2. Leader向各个Follower同时开始发送数据，并判断是否达到多数派提交成功；
3. Leader在提交成功后发起Apply请求，将日志应用到Leader所在的状态机。
4. Follower接收到Apply请求之后，同时将日志应用到Follower所在的状态机。

继任Leader是否可以将已经复制超过半数的 log 提交掉？不能，

目前还有疑问???????

如果做批量发送？批量的策略是什么？通过Ready对象实现；目前还没有确认是批量N条还是批量N秒内的数据
如何判断是否在集群中持久化成功？正常来说，commit成功之后就可以返回给客户端在集群中已经提交成功。
如果多数派的数据永久丢失，那么如何恢复？实际上在正式的论文中没有讨论，在FM-RAFT中有相关的介绍，也就是通过InitializeCluster接口
怎样定义一个节点是否是健康的？对于Leader？对于Follower可以周期接收到从Leader中发送的心跳信息，可以提供用户只读的是lastApplied的数据，也就是已经持久化到状态机的数据。
Term溢出(TermID一般是无符号整形)会导致什么问题？
https://coolshell.cn/articles/11466.html
https://developer.apple.com/library/content/documentation/Security/Conceptual/SecureCodingGuide/Introduction.html

## Lexical Bytes-Order

也可以称为 Alphabetical Order，另外还有 Numerical Order，例如 1 10 2 是属于前者。



一致性区分
https://github.com/coreos/etcd/blob/master/Documentation/learning/api_guarantees.md


http://www.ituring.com.cn/book/tupubarticle/16510
https://coreos.com/blog/announcing-etcd-3.3

系统调用的错误注入
https://lrita.github.io/2017/06/27/systemtap-inject-syscall-error/

当前系统信息收集
https://github.com/coreos/mayday
分布式的测试工具
https://github.com/coreos/dbtester


自习周报：CoreOS 的黑魔法
https://zhuanlan.zhihu.com/p/29882654







Linearizability 用来保证针对某个对象的一系列操作在墙上时间 (wall-clock) 是有序的，通常针对的是分布式系统。也就是说，针对多个节点可以完成原子操作，例如写完成后，其它节点可以立即读取到当前的状态。

Serializability 保证一系列针对多个对象的操作与针对对象的序列操作是相同的，通常是针对数据库的最高级别操作。

Strict Serializability 实际上是结合了上述的两种方式。

Linearizable read requests go through a quorum of cluster members for consensus to fetch the most recent data. 
Serializable read requests are cheaper than linearizable reads since they are served by any single etcd member, instead of a quorum of members, in exchange for possibly serving stale data.

https://aphyr.com/posts/313-strong-consistency-models




-->

{% highlight text %}
{% endhighlight %}
