---
title: ETCD 存储模块
layout: post
comments: true
language: chinese
category: [program,golang,linux]
keywords: golang,go,etcd
description:
---

如前所述，ETCD 中 RAFT 协议的只是实现了其核心的部分，而其中的存储模块需要单独实现。

<!-- more -->

## Storage

应用程序需要实现存储 IO 和网络通讯，其中存储在 RAFT 中通过 `type Storage interface` 定义，包括了读取 log、执行 Snapshot 等接口。

其本身实现了基于内存的` MemoryStorage[raft/storage.go]`，ETCD 将其作为 Cache 使用，每次事务中会先将日志持久化到存储设备上，然后再更新 MemoryStorage 。

{% highlight go %}
type Storage interface {
	// 初始化时会返回持久化之后的HardState和ConfState
	InitialState() (pb.HardState, pb.ConfState, error)
	// 返回范围[lo,hi)内的日志数据
	Entries(lo, hi, maxSize uint64) ([]pb.Entry, error)
	// 获取entry i的term值
	Term(i uint64) (uint64, error)
	// 日志中最新一条日志的序号
	LastIndex() (uint64, error)
	// 日志中的第一条日志序号，老的日志已经保存到snapshot中
	FirstIndex() (uint64, error)
	Snapshot() (pb.Snapshot, error)
}

type MemoryStorage struct {
	sync.Mutex
	hardState pb.HardState
	snapshot  pb.Snapshot
	ents []pb.Entry
}
{% endhighlight %}

如上是，`Storage` 接口和 `MemoryStorage` 结构体的定义。

<!--
与unstable一样，Storage也被嵌入在raftLog结构中。需要说明的一点是：将日志项追加到Storage的动作是由应用完成的，而不是raft协议核心处理层。目前尚不理解Storage存在的意义是什么，与unstable到底有什么区别？
-->

## ETCD 实现

实际上是在 `etcdserver/storage.go` 中的实现，其接口定义名称与上述相同，同样也是 `type Storage interface`，注意不要将两者混淆。

{% highlight go %}
type Storage interface {
	Save(st raftpb.HardState, ents []raftpb.Entry) error
	SaveSnap(snap raftpb.Snapshot) error
	Close() error
}
type storage struct {
	*wal.WAL
	*snap.Snapshotter
}
{% endhighlight %}

在上述定义的 `type storage struct` 结构体中，根据 Go 语言的特性，因为没有声明成员变量的名字，可以直接使用 WAL 和 Snapshotter 定义的方法，也就是该结构体是对后两者的封装。

这里的 `storage` 和 `MemoryStorage` 的结合使用就是在 `etcdserver/raft.go` 实现，对应了 `type raftNode struct` 结构体，其中包含的是 `type raftNodeConfig struct` ，也就是真正的封装。

{% highlight go %}
type raftNodeConfig struct {
	isIDRemoved func(id uint64) bool
	raft.Node
	raftStorage *raft.MemoryStorage
	storage     Storage
	heartbeat   time.Duration // for logging
	transport rafthttp.Transporter
}
{% endhighlight %}

如上，其中 `raftStorage` 是提供给 RAFT 协议层使用的，而 `storage` 则是 ETCD 实现持久化存贮的核心内容。

<!--
在使用中，etcd以连续调用的方式实现二者一致的逻辑。以etcd server重启为例，我们看看同步是如何实现的，且看restartNode()的实现。
-->

## 启动流程

{% highlight text %}
NewServer()                   etcdserver/server.go
 |-store.New() store/store.go 根据入参创建一个初始化的目录
 | |-newStore() 创建数据存储的目录
 |-snap.New() snap/snapshotter.go 这里只是初始化一个对象，并未做实际操作
 |-openBackend() etcdserver/backend.go
 | |-newBackend() 在新的协程中打开，同时会设置10秒的超时时间
 |
 | <<<haveWAL>>> 存在WAL日志，也就是非第一次部署
 |-Snapshotter.Load() snap/snapshotter.go 开始加载snapshot
 | |-Snapshotter.snapNames() 会遍历snap目录下的文件，并逆序排列返回
 | |-loadSnap() 依次加载上述返回的snap文件
 |   |-Read() 读取文件，如果报错那么会添加一个.broken的后缀
 |     |-ioutil.ReadFile() 调用系统接口读取文件
 |     |-snappb.Unmarshal() 反序列化
 |     |-crc32.Update() 更新并校验CRC的值
 |     |-raftpb.Unmarshal() 再次反序列化获取值
 |-store.Recovery() store/store.go 从磁盘中恢复数据
 | |-json.Unmarshal() snap中保存的应该是json体
 | |-newTtlKeyHeap() 一个TTL的最小栈，用来查看将要过期的数据
 | |-recoverAndclean() ???没有理清楚具体删除的是什么过期数据
 |-recoverSnapshotBackend() etcdserver/backend.go 开始恢复snapshot
 | |-openSnapshotBackend() 这里会将最新的一次的snapshot重命名为DB
 |   |-openBackend()
 |
 |-restartNode() etcdserver/raft.go
 | |-readWAL()
 | |-raft.NewMemoryStorage()
 | |-ApplySnapshot()
 | |-SetHardState()
 | |-Append()
 | |-RestartNode()
 |-SetStore()
 |-SetBackend()
 |-Recover()
{% endhighlight %}


<!--


## 日志更新

这个函数的主要处理逻辑就是通过读取 Snapshot 和 WAL，然后通过 SetHardState() 和 Append() 恢复当前 memoryStrorage 的状态。


## SnapShot

涉及到几个重要的问题：

1. 何时触发。
2. 效率如何。

保存的是某个时间节点系统当前状态的一个快照，便用户恢复到此时的状态，ECTD 中 snapshot 的目的是为了回收日志占用的存储空间，包括内存和磁盘。

更新首先被转化为更新日志，按照顺序追加到日志文件，并在集群中进行同步，只有在写入多个节点的日志项会应用到状态机，日志会一直增长，因此需要特定的机制来回收那些无用的日志。

ETCD 中的 snapshot 代表了应用的状态数据，而执行 snapshot 的动作也就是将应用状态数据持久化存储，这样，在该 snapshot 之前的所有日志便成为无效数据，可以删除。

### 结构体

SnapShot 的结构体在 raft/raftpb/raft.proto 中定义，`message Snapshot` 定义了序列化后的格式，其中包括了日志条目的序号等信息。

### 持久化


EtcdServer.snapshot() etcdserver/server.go 真正处理
 |-store.Clone() store/store.go
 | |-newStore() 会新建一个对象，并复制所需要的成员
 | |-KV().Commit()
 |
 |-storage.SaveSnap()
 | |-walpb.Snapshot{} 实例化对象，这里会设置Index和Term
 | |-WAL.SaveSnapshot()
 | |-Snapshotter.SaveSnap() snap/snapshotter.go
 | | |-Snapshotter.save() 将数据序列化后保存到磁盘上
 | |   |-pioutil.WriteAndSyncFile() 格式化并保存
 | |-WAL.ReleaseLockTo()

raftStorage.CreateSnapshot()




BoltDB的COW技术
http://www.d-kai.me/boltdb%E4%B9%8Bcow%E6%8A%80%E6%9C%AF/

ETCD 监控
https://coreos.com/etcd/docs/latest/op-guide/monitoring.html





这里没有理清楚，为什么第一发送数据需要写入到 WAL 以及存储中？？？？？？


也就是说在 Readyc 中包含了需要发送的数据、已经提交的数据、需要应用的数据，那么如何区分呢？？？？？


经过该步之后，日志条目才写入到wal文件和memorty storage。消息也才经Transport发送给其他节点。


消息处理。。。。




http://blog.sina.com.cn/s/blog_4b146a9c0102yml3.html
https://blog.csdn.net/xxb249/article/details/80787501
http://www.ituring.com.cn/book/tupubarticle/16510

https://my.oschina.net/fileoptions/blog/1825531
https://blog.csdn.net/xxb249/article/details/80790587
http://www.opscoder.info/ectd-raft-library.html
https://yuerblog.cc/2017/12/10/principle-about-etcd-v3/


## SnapShot


snapshot是wal快照，为了节约磁盘空间，当wal文件达到一定数据，就会对之前的数据进行压缩，形成快照。
2）snapshot另外一个原因，当新的节点加入到集群中，为了同步数据，就会把snapshot发送到新节点，这样能够节约传输数据(生成的快照文件比wal文件要小很多，5倍左右)，使之尽快加入到集群中。


## Storage

也就是静态存储，用来将数据持久化到磁盘上，是对 WAL 和 Snapshot 的封装。

## MemoryStorage











## Config

降低 raftexample/raft.go 中的 defaultSnapCount 配置，修改为 100，然后进行测试。

### loadSnapshot

`raftNode.loadSnapshot()` 获取最新的一个 Snapshot，真正读取的时候是在 `snap/snapshotter.go` 中的 `Read()` 函数中实现，实际上是反序列化了两次。

1. 反序列化为 `snappb.Snapshot[snap/snappb/snap.proto]` ，实际上就是在数据外外包了一层 CRC32 的校验值。
2. 将上述反序列化后的数据再次执行反序列化，也即是 `raftpb.Snapshot[raft/raftpb/raft.proto]` 。

// snap/snappb/snap.proto
message snapshot {
        optional uint32 crc  = 1 [(gogoproto.nullable) = false];
        optional bytes data  = 2;
}

// raft/raftpb/raft.proto
message ConfState {
        repeated uint64 nodes    = 1;
        repeated uint64 learners = 2;
}

message SnapshotMetadata {
        optional ConfState conf_state = 1 [(gogoproto.nullable) = false];
        optional uint64    index      = 2 [(gogoproto.nullable) = false];
        optional uint64    term       = 3 [(gogoproto.nullable) = false];
}

message Snapshot {
        optional bytes            data     = 1;
        optional SnapshotMetadata metadata = 2 [(gogoproto.nullable) = false];
}

在 `raftpb.Snapshot` 中，除了要保存的数据之外，还包括了一些配置信息，例如集群配置、`Term`、`Index` 。

### openWAL

如果存在 Snapshot ，根据 Snapshot 中的 Term+Index 构建一个 `walpb.Snapshot[wal/walpb/record.proto]` ，然后在打开 WAL 日志时作为入参传入函数。

message Snapshot {
        optional uint64 index = 1 [(gogoproto.nullable) = false];
        optional uint64 term  = 2 [(gogoproto.nullable) = false];
}

通过 `Open()[wal/wal.go]` 打开 WAL 文件，此时会遍历序号大于 Term 的文件。

然后，通过 `ReadAll()[wal/wal.go]` 读取文件，此时会返回最后一次持久化的状态，以及日志文件。

#### 记录类型

每条记录同样是以 Protobuf 的格式进行保存，其定义的格式在 `raft/raftpb/raft.proto` 中定义。

enum EntryType {
    EntryNormal     = 0;
    EntryConfChange = 1;
}

message Entry {
    optional uint64     Term  = 2 [(gogoproto.nullable) = false]; // must be 64-bit aligned for atomic operations
    optional uint64     Index = 3 [(gogoproto.nullable) = false]; // must be 64-bit aligned for atomic operations
    optional EntryType  Type  = 1 [(gogoproto.nullable) = false];
    optional bytes      Data  = 4;
}

### 其它


注意，示例代码中的 raftStorage 实际上使用的是 `MemoryStorage[raft/storage.go]` 中的实现，也就是说默认是缓存到内存中的，如果重启需要重新更新状态到内存中。


ApplySnapshot() raft/storage.go 更新MemoryStorage.snapshot，并在ents中添加一个记录
SetHardState() raft/storage.go  更新MemoryStorage.hardState
Append() raft/storage.go 将日志添加到MemoryStorage.ents中，这里会判断是否有重复的Entries

replayWAL() 会从 Snapshot 和 WAL 恢复数据，并更新到 MemoryStorage 中。

这里会同时更新 raftNode.lastIndex 的值。



在启动时可以通过 `--snapshot-count` 参数(默认10000)指定何时触发执行 Snapshot ，此时会生成 `.snap` 文件。会在 `triggerSnapshot()` 函数中判断是否需要执行。

EtcdServer.triggerSnapshot() etcdserver/server.go 判断是否需要触发Snapshot操作
 |-EtcdServer.snapshot() 如果需要则执行Snapshot操作
   |-store.Clone() store/store.go 这里实际上会复制一堆的数据结构？？？？具体作用是啥


EtcdServer.applySnapshot() etcdserver/server.go 从Snapshot中恢复

ECTD很不错的介绍
http://blog.zhesih.com/2017/10/02/snap-file-of-etcd-v3/
https://my.oschina.net/fileoptions/blog/1825531



-->

{% highlight text %}
{% endhighlight %}
