---
title: ETCD 示例源码
layout: post
comments: true
language: chinese
category: [program,golang,linux]
keywords: golang,go,etcd
description: 现在已知的 Golang 版本的 RAFT 的开源实现主要有两个：一个是 CoreOS 的 etcd 中的实现，使用的项目有比如 tidb、cockroachdb 等；另外一个是 hashcorp 的 RAFT 实现，使用的项目有比如 consul、InfluxDB 等。相比而言，前者只实现了一个整体框架，很多的功能需要用户实现，难度增加但是更加灵活；而后者则是完整的实现，WAL、SnapShot、存储、序列化等。
---

现在已知的 Golang 版本的 RAFT 的开源实现主要有两个：一个是 CoreOS 的 etcd 中的实现，使用的项目有比如 tidb、cockroachdb 等；另外一个是 hashcorp 的 RAFT 实现，使用的项目有比如 consul、InfluxDB 等。

相比而言，前者只实现了一个整体框架，很多的功能需要用户实现，难度增加但是更加灵活；而后者则是完整的实现，WAL、SnapShot、存储、序列化等。

<!-- more -->

## 简介

整体来说，该库实现了 RAFT 协议核心内容，如 append log、选主逻辑、snapshot、成员变更等；但该库没有实现消息传输和接收，只会把待发送消息保存在内存中，通过用户自定义的网络传输层取出消息并发送出去，并且在网络接收端，需要调一个库函数，用于将收到的消息传入库。

同时，该库定义了一个 Storage 接口，需要库的使用者自行实现。

### 示例程序

RAFT 协议的实现主要包括了四部分：协议逻辑、存储、消息序列化和网络传输，而 ETCD 对应的 RAFT 库只实现了最核心算法。

源码可以直接下载 [Github coreos/etcd](https://github.com/coreos/etcd) 其中有一个简单的示例 [contrib/raftexample](https://github.com/coreos/etcd/tree/master/contrib/raftexample)，这是一个内存的 KV 存储引擎。

示例中的 wal 和 snapshot 实际上使用的是 ETCD 中的实现。


<!--
RAFT 交互流程相关的内容都放在 raftNode 中，而节点状态、IO调用、事件触发起点等入口都放在了 node 中，两者都在启动后起了一个 for-select 结构的协程循环处理各自负责的事件。
1) 递增currentTerm，投票给自己；2) 重置ElectionTimer；3) 向所有的服务器发送 RequestVote RPC请求
-->

### 使用

看下如何在系统中使用。

#### 编译

直接修改编译脚本 build 中的编译命令，然后直接运行 `./build` 即可。

{% highlight text %}
go build -o "${out}/raftexample" ${REPO_PATH}/contrib/raftexample || return
{% endhighlight %}

#### 测试

可以在一个节点上启动三个服务进程(也可以只启动一个)。

{% highlight text %}
$ raftexample --id 1 --cluster http://127.0.0.1:12379,http://127.0.0.1:22379,http://127.0.0.1:32379 --port 12380
$ raftexample --id 2 --cluster http://127.0.0.1:12379,http://127.0.0.1:22379,http://127.0.0.1:32379 --port 22380
$ raftexample --id 3 --cluster http://127.0.0.1:12379,http://127.0.0.1:22379,http://127.0.0.1:32379 --port 32380
{% endhighlight %}

如上，列出了整个集群的节点数，启动后会发起一次选举过程。

然后，通过如下方式发送和获取数据。

{% highlight text %}
curl -L http://127.0.0.1:12380/my-key -XPUT -d hello
curl -L http://127.0.0.1:12380/my-key
{% endhighlight %}

#### 新增节点

还可以通过以下方式将一个新节点加入集群:

{% highlight text %}
$ curl -L http://127.0.0.1:12380/4 -XPOST -d http://127.0.0.1:42379
$ raftexample --id 4 --port 42380 --join \
     --cluster http://127.0.0.1:12379,http://127.0.0.1:22379,http://127.0.0.1:32379,http://127.0.0.1:42379
{% endhighlight %}

#### 删除节点

{% highlight text %}
$ curl -L http://127.0.0.1:12380/3 -XDELETE
{% endhighlight %}

<!--
仔细阅读了下源代码发现，其主要区别在于启动RaftNode的时候，如果是join，就不会传递集群其他peers信息给raft组件，否则就会传递。但是是否传递该参数对raft组件的行为有什么影响目前还不太清楚，有待接下来继续分析raft组件了。
-->

## 数据结构

简单介绍下常见的数据结构。

### KVStore

通过 kvstore 抽象了应用的全部相关内容。

{% highlight go %}
type kvstore struct {
	proposeC    chan<- string
	mu          sync.RWMutex
	kvStore     map[string]string
	snapshotter *snap.Snapshotter
}
{% endhighlight %}

最核心的成员有：

* proposeC 这是应用和底层 RAFT 库之间的通信管道(Channel)，所有对应用的更新请求都会由应用通过该管道向底层 RAFT 库传递；
* kvStore：内存状态机，存储应用的状态数据；
* snapshotter 应用管理 Snapshot 的接口。

### RaftNode

该结构用来衔接底层 RAFT 协议以及应用层，通过该结构可以简化 RAFT 底层的细节，降低系统耦合度。该结构需要处理的任务包括：

* 应用的更新请求传递给底层 RAFT 协议；
* RAFT 已提交的请求传输给应用以更新应用的状态机；
* 处理 RAFT 组件产生的指令，如选举指令、数据复制指令、集群节点变更指令等；
* 处理 WAL 日志；
* 将底层 RAFT 组件的指令通过网络传输至集群其它节点等。

相较于应用来说，该结构 (type raftNode struct) 功能的实现更为复杂，其核心数据结构定义如下：

* proposeC 应用将客户的更新请求传递至底层 RAFT 组件的管道；
* commitC 底层 RAFT 组件通知应用层准备提交的通道；

<!--
node：是对底层的raft组件的抽象，所有与底层raft组件的交互都通过该结构暴露的API来实现；
wal：WAL日志管理，etcd-raft将日志的管理交给应用层来处理；
snapshotter：同wal，etcd-raft将快照的管理也交给应用层来处理；
transport：应用同其他节点应用的网络传输接口，同wal，etcd-raft将集群节点之间的网络请求发送和接收也交给应用层来处理。

ETCD RAFT 的核心层实现相对轻量级，而带来的后果是应用需要处理较为复杂的与协议有关内容，带来灵活性的同时也增加了应用的复杂性。
-->

## 初始化

RAFT 协议是一种在多节点组成的集群之间进行状态同步的协议，示例是通过 `newRaftNode()` 新建一个 raftNode 抽象对象，在创建时需要指定其它节点的 IP(peers)、该节点的 ID(id) 等信息。

{% highlight text %}
main()                                  main.go
  |-make(chan string)                   新建proposeC管道，用来将用户层数据发送给RAFT协议层
  |-make(chan raftpb.ConfChange)        新建confChangeC管道，用来将配置修改信息发送给RAFT协议层
  |-newRaftNode()                       raft.go 返回结构体会作为底层RAFT协议与上层应用的中间结合体
  | |                                       同时会返回commitC errorC管道，用来接收请求和错误信息
  | |-raftNode()                        <<<1>>>新建raftNode对象，重点proposeC
  | |-raftNode.startRaft()              [协程] 启动示例中的代码
  |   |-os.Mkdir()                      如果snapshot目录不存在则创建
  |   |-snap.New()                      调用的是ETCD中snap的实现
  |   |-wal.Exist()                     判断WAL日志目录是否存在，用来确定是否是第一次启动
  |   |-raftNode.replayWAL()            重放WAL日志
  |   | |-raftNode.loadSnapshot()
  |   | | |-snap.Snapshotter.Load()     snap/snapshotter.go 中的Load()函数
  |   | |   |-Snapshotter.snapNames()   会打开目录并遍历目录下的文件，按照文件名进行排序
  |   | |   |-Snapshotter.loadSnap()
  |   | |     |-Snapshotter.Read()      会读取文件并计算CRC校验值，判断文件是否合法
  |   | |-raftNode.openWAL()            如果不存在则创建目录，否则加载snapshot中保存的值并尝试加载
  |   | | |-wal.Open()                  使用的是ETCD中的wal/wal.go文件
  |   | |-ReadAll()                     读取所有日志
  |   | |-raft.NewMemoryStorage()       使用ETCD中的内存存储
  |   | |-raft.SetHardState()
  |   | |-raft.Append()
  |   |
  |   |-raft.Config{}                   构建RAFT组件的配置项
  |   |-raft.RestartNode()              如果已经安装过WAL则直接重启Node，这最常见场景
  |   |-raft.StartNode()                第一次安装，则重新部署
  |   |
  |   |-rafthttp.Transport{}            传输层的配置参数
  |   |-transport.Start()               启动HTTP服务
  |   |
  |   |-raftNode.serveRaft()            [协程] 主要是启动网络监听
  |   |-raftNode.serveChannels()        [协程] raft.go 真正的业务处理，在协程中监听用户请求、配置等命令
  |   | |-raftStorage.Snapshot()        获取snapshot
  |   | |-raft.Node.Propose()           阻塞等待该用户请求被RAFT状态机接受
  |   | |-raft.Node.ProposeConfChange() 发送配置请求
  |   |
  |   |-raft.Node.Tick()                另外的协程处理RAFT组件的同步信息
  |
  |-newKVStore()                        kvstore.go 创建内存KV存储结构
  | |-kvstore{}                         实例化KVStore存储对象
  | |-kvstore.readCommits()             会启动一个协程，也是存储的核心，用于读取已经提交的数据
  |   |                                    这里实际上调用了两次，第一次是函数内调用，第二次是协程
  |   |-snapshot.Load()                 第一次commitC中没有数据，实际上是加载snapshot
  |   |-recoverFromSnapshot()           从snapshot中恢复
  |   |                                 接下来是协程的处理
  |   |-gob.NewDecoder()                反序列化
  |   |-kvStore[]                       保存到内存中
  |
  |-serveHttpKVAPI()                    启动对外提供服务的HTTP端口
    |-srv.ListenAndServe()              真正启动客户端的监听服务
{% endhighlight %}

也就是说，启动流程主要分为了三步：

1. 初始化 RAFT
2. 应用初始化
3. 应用开启对外服务

应用和 RAFT 组件之间是通过 Channel 进行信息传递。

上述的 HTTP 端口中真正处理请求的函数为 `ServeHTTP()` 函数，下面会挨个介绍。

<!--
raftNode
raftNode是etcd中真正的执行者. 它主要是封装了一个Node的interface在里边，然后围绕这个interface做各种事情. 其中包括：

维护几个主要的数据通道: proposeC, confChangeC, commitC, errorC 等
节点的各种信息: id, peers, index, raftStorage, ...
WAL 的读写
snapshot 读写
状态机的操作等等
其中Node主要用到的一些方法有：

Tick()
Stop()
Advance()
ProposeConfChange(ctx context.Context, cc pb.ConfChange) error
ApplyConfChange(cc pb.ConfChange) *pb.ConfState
Propose(ctx context.Context, data []byte) error
Ready() <-chan Ready
Step(ctx context.Context, msg pb.Message) error
这些函数看名字都大体知道其作用，在这里我们先不细说，下面流程中会依次碰到。
-->


### 新建raftNode对象

在初始化时，为了与 RAFT 内的协议层进行通讯，需要提供 4 个 Channel，分别是：

{% highlight text %}
proposeC := make(chan string)
confChangeC := make(chan raftpb.ConfChange)
commitC := make(chan *string)
errorC := make(chan error)
{% endhighlight %}

其中，前两个在创建 `raftNode` 前创建传入，就是数据的入口；而后两个则是在创建完成后返回，相当于数据的出口。

* proposeC 用于向RAFT协议层提交写请求，也就是 Propose；
* confChangeC 用于向RAFT协议层提交配置变更请求，也就是 ProposeConfChange；
* commitC 把已经提交的entries从RAFT协议层暴露给用户 State Machine；
* errorC 让用户可以及时处理RAFT协议层抛出的错误信息。

## 写数据

简单来说就是，用户发送一个 PUT 请求，用来写入 KV 内存数据，可以分为如下步骤。

{% highlight text %}
http PUT -1-> kvStore.Propose -2-> proposeC -3-> raft -4-> commitC -5-> map[string] string
{% endhighlight %}

HTTP 请求数据调用 `kvStore.Propose()` 方法把请求数据通过 `proposeC` 管道发送给 RAFT 协议核心，在 RAFT 协议中经过一系列的操作后再把数据通过 `commitC` 这个管道暴露出来。

初始化时，会启动一个协程来消费 `commitC` 这个管道，也就是把已经提交的结果最终写入到内存中的 `map[string]stringe` 里边去。


<!--
需要指出的是用户代码在消费commitC的数据之前，还需要处理raft的snapshot数据. 例子中用的是etcd已经实现好的github.com/coreos/etcd/snap这个包来处理的. 在本例中做的事情其实非常简单，snapshot有正反两个相对的操作: 序列化和反序列化. 例子中直接对内存中的map做json.Marshal(s.kvStore) 和 json.Unmarshal(snapshot, &store)。

整个流程中真正能让我们感兴趣的应该在4. raft, 5. commitC, 以及5->6这几个部分。从这里开始复杂起来了，我们也不得不一步一步在代码中挖下去.
-->

### 处理流程

如上所述，HTTP 真正处理请求是在 `ServeHTTP()` 函数中，包括了 PUT(增加数据) GET(查看数据) POST(修改配置) DELETE(删除数据) 四种类型的请求处理。

对于客户端的更新请求，首先通过 HTTP 协议传输给应用，目前无法直接处理更新 KVStore，需要先提交至 RAFT 组件在集群内部对本次提交达成一致。也即是，要将这次请求通过 proposeC 管道将请求发送给 raftNode 结构。

{% highlight text %}
ServeHTTP()                           httpapi.go
  |====> PUT方法
  |-ioutil.ReadAll()                  从HTTP中读取请求
  |-kvstore.Propose()                 kvstore.go 正式提交请求，阻塞直到RAFT状态机提交成功
  | |-glob.NewEncoder()               序列化
  | |-s.proposeC <- buf.String()      通过proposeC管道发送请求到RAFT核心，会阻塞直到返回
  |
  |-http.ResponseWriter.WriteHeader() 返回数据结果
  |
  |====> GET方法
  |-kvstore.Lookup()                  查找并返回数据
{% endhighlight %}

在发送到 proposeC 之后，实际上会开始调用 `serveChannels()` 启动的协程中，然后会一直阻塞直到该请求返回结果。

接下来就是 RAFT 组件的核心处理部分，也即是提供的 `Propose()` API 接口，这里暂时不讨论。

### 提交数据

也就是第二步，会调用 raftNode 中的 `raftNode.node.Propose()` 方法把数据交给 raft 核心处理。

{% highlight go %}
// raft/node.go
func (n *node) Propose(ctx context.Context, data []byte) error {
    return n.step(ctx, pb.Message{Type: pb.MsgProp, Entries: []pb.Entry{{Data: data}}})
}
{% endhighlight %}

其中的 `step` 是一个函数指针，根据角色可以是 `stepFollower()`、`stepCandidate()`、`stepLeader()` 等不同的函数，当然这些处理都是在 RAFT 核心中完成的。



在启动之后，实际上会在后台运行一个 long running 的协程，也就是 `raft/node.go` 中的 `run()` 方法，核心代码的示例如下。

{% highlight go %}
func (n *node) run(r *raft) {
	for { // 死循环
		if advancec != nil {
			readyc = nil
		} else {
			rd = newReady(r, prevSoftSt, prevHardSt)
			if rd.containsUpdates() {
				readyc = n.readyc
			} else {
				readyc = nil
			}
		}
	}
}
{% endhighlight %}

消息先进入 `r.msgs` 被 `newReady()` 函数取走，用户代码通过消费 `Ready() <-chan Ready` 来处理各种消息。

<!--
之所有消息没有立刻通过commitC暴露给用户的state machine, 就是因为上边我们了解掉的commit的过程。当raft说这个消息已经被commit掉了，它就会以committedEntries身份出现。这时候用户代码需要负责自己把这些committedEntries通过commitC抛给State Machine.
-->


{% highlight go %}
func (rc *raftNode) serveChannels() {

	// event loop on raft state machine updates
	for {
		select {
		case <-ticker.C:
			rc.node.Tick()

		// store raft entries to wal, then publish over commit channel
		case rd := <-rc.node.Ready():
			rc.wal.Save(rd.HardState, rd.Entries) // 保存到持久化存储中
			if !raft.IsEmptySnap(rd.Snapshot) {
				rc.saveSnap(rd.Snapshot)
				rc.raftStorage.ApplySnapshot(rd.Snapshot)
				rc.publishSnapshot(rd.Snapshot)
			}
			rc.raftStorage.Append(rd.Entries)
			rc.transport.Send(rd.Messages)

			// 通过commitC告诉给下游的用户代码
			if ok := rc.publishEntries(rc.entriesToApply(rd.CommittedEntries)); !ok {
				rc.stop()
				return
			}
			rc.maybeTriggerSnapshot()
			rc.node.Advance()   // 处理完成需要主动告诉raft

		case err := <-rc.transport.ErrorC:
			rc.writeError(err)
			return

		case <-rc.stopc:
			rc.stop()
			return
		}
	}
}

// publishEntries writes committed log entries to commit channel and returns
// whether all entries could be published.
func (rc *raftNode) publishEntries(ents []raftpb.Entry) bool {
	for i := range ents {
		switch ents[i].Type {
		// 正常的HTTP PUT会触发一个EntryNormal请求
		case raftpb.EntryNormal:
			if len(ents[i].Data) == 0 {
				// ignore empty messages
				break
			}
			s := string(ents[i].Data)
			select {
			case rc.commitC <- &s:
			case <-rc.stopc:
				return false
			}

		case raftpb.EntryConfChange:
			var cc raftpb.ConfChange
			cc.Unmarshal(ents[i].Data)
			rc.confState = *rc.node.ApplyConfChange(cc)
			switch cc.Type {
			case raftpb.ConfChangeAddNode:
				if len(cc.Context) > 0 {
					rc.transport.AddPeer(types.ID(cc.NodeID), []string{string(cc.Context)})
				}
			case raftpb.ConfChangeRemoveNode:
				if cc.NodeID == uint64(rc.id) {
					log.Println("I've been removed from the cluster! Shutting down.")
					return false
				}
				rc.transport.RemovePeer(types.ID(cc.NodeID))
			}
		}

		// after commit, update appliedIndex
		rc.appliedIndex = ents[i].Index

		// special nil commit to signal replay has finished
		if ents[i].Index == rc.lastIndex {
			select {
			case rc.commitC <- nil:
			case <-rc.stopc:
				return false
			}
		}
	}
	return true
}
{% endhighlight %}

## RAFT 指令处理

现在客户端的请求通过 `proposeC` 管道进入了 RAFT 组件，在数据完成同步之后还是通过 `Ready()` 暴露给应用，然后由应用负责写日志，完成提交，同步给其它的 Follower 节点等。

也就是如何一步步的处理 RAFT 组件内部的指令请求，同样是在 `serveChannels()` 中启动的协程。

{% highlight text %}
func (rc *raftNode) serveChannels() {
    // 上面部分是启动了一个协程处理Propose请求
    // event loop on raft state machine updates
    for {
        select {
        case <-ticker.C:
            rc.node.Tick()

        // 1. 通过Ready()获取到RAFT组件指令
        case rd := <-rc.node.Ready():
            // 2. 写WAL日志，包含的是当前的状态信息
            rc.wal.Save(rd.HardState, rd.Entries)
            if !raft.IsEmptySnap(rd.Snapshot) {
                rc.saveSnap(rd.Snapshot)
                rc.raftStorage.ApplySnapshot(rd.Snapshot)
                rc.publishSnapshot(rd.Snapshot)
            }
            // 3. 这是干什么?
            rc.raftStorage.Append(rd.Entries)
            // 4. 发送给某个Follower
            rc.transport.Send(rd.Messages)
            // 5. 将已经commit的日志提交到应用状态机
            ok := rc.publishEntries(rc.entriesToApply(rd.CommittedEntries))
            if !ok {
                rc.stop()
                return
            }
            rc.maybeTriggerSnapshot()
			// 6. 通知RAFT组件该请求已经处理完成，可以进行下次的请求了
            rc.node.Advance()
        case err := <-rc.transport.ErrorC:
            rc.writeError(err)
            return
        case <-rc.stopc:
            rc.stop()
            return
        }
    }
}
{% endhighlight %}

当 RAFT 组件判定已经复制到了多个节点之后，也就是认为已经提交(Commit)，此时 RAFT 组件会通过 commitC 管道将请求返回给应用(KVStore)，应用收到请求后将其应用到状态机，也就是内存中的 KV 存储。

简单来说，通过 `readCommits()` 接收用户发送的请求并发送给 RAFT 组件；在 RAFT 组件处理完成提交后，再发送给 `serveChannels()` 继续处理，保存到应用的 KV 存储中。

## 日志管理

示例使用了 ETCD 提供的通用日志库来进行日志管理，这里重点看下应用层如何使用提供的 WAL 日志模块，其实现后面再详细描述。

### 日志追加

为了防止数据丢失，在更新之前会先将日志项追加到日志文件中，也就是如下：

{% highlight text %}
func (rc *raftNode) serveChannels() {
    ......
    for {
        select {
        case <-ticker.C:
           rc.node.Tick()

        // 正常更新请求,第一步先追加日志
        case rd := <-rc.node.Ready():
            rc.wal.Save(rd.HardState, rd.Entries)
        ......
    }
    ......
}
{% endhighlight %}

### 日志重放

在程序启动时，第一步便是进行日志重放，构建内存状态机。

{% highlight text %}
func (rc *raftNode) replayWAL() *wal.WAL {
    snapshot := rc.loadSnapshot()
    w := rc.openWAL(snapshot)
    _, st, ents, err := w.ReadAll()
    if err != nil {
        log.Fatalf("raftexample: failed to read WAL (%v)", err)
    }
    rc.raftStorage = raft.NewMemoryStorage()
    if snapshot != nil {
        rc.raftStorage.ApplySnapshot(*snapshot)
    }

    rc.raftStorage.SetHardState(st)
    rc.raftStorage.Append(ents)
    if len(ents) > 0 {
        rc.lastIndex = ents[len(ents)-1].Index
    } else {
        rc.commitC <- nil
    }
    return w
}
{% endhighlight %}

<!--
因为日志又总是和snapshot搅和在一起的，因此，构建内存状态机必须是Snapshot + 日志一起。

最关键的问题是：由于糅合了snapshot，我们需要明确需要重放哪些日志。在重放日志时，应用程序首先会load最新的snapshot，这个在下面的snapshot管理中会描述。然后根据这个snapshot的日志index在WAL目录下查找该index之后的日志，接下来只需要回放这些日志即可。

日志压缩

日志在追加的过程中可能会一直增长，因此，需要通过一种机制来抑制这种增长，标准做法是snapshot：即将内存当前状态进行压缩成为snapshot存储在文件中，然后，该snapshot之前的日志便可以全部丢弃。

etcd-raft example中如何生成snapshot我们在下面会描述。

在示例应用中好像没有实现日志压缩功能。

## Snapshot

本质上是应用状态的一份拷贝，将状态机当前的状态保存到磁盘上，其主要目的是为了回收日志文件，防止更新日志会越来越大。Snapshot 之前的数据完全可以直接删除，系统重启之后只需要最新的 snapshot 并回放其之后的更新日志即可。

### 创建时机

创建 snapshot 的成本过高，因此不会过于频繁，示例中是没更新 1W 条日志才会创建一次 snapshot 。

maybeTriggerSnapshot()
 |                       首先会判断是否需要进行SnapShot操作
 |-getSnapshot() 这里实际上是一个函数指针，在创建RaftNode时传入
 | |-kvstore.getSnapshot() 调用KVStore中的相应函数，实际会保存所有内存中的数据
 |   |-json.Marshal()
 |-raftStorage.CreateSnapshot() 创建snapshot
 |-saveSnap() 保存
 |-raftStorage.Compact() 用来压缩日志

注意，snapshot 针对的是那些已经被应用到状态机的日志，还未被应用到状态机的更新日志(appliedIndex)是不能被回收的。

在示例应用中，每次进行snapshot后会将snapshot保存在磁盘中，其中包括两方面数据：

snapshot数据：即当前应用内存状态的实际数据，一般被存储在当前的快照目录中；
snapshot索引数据：即当前快照的index信息，这个信息对WAL至关重要，这个index决定了日志压缩的时候哪些可以被回收，也决定了日志重放的时候哪些可以被略过。snapshot索引数据被存放在日志目录下。示例应用好像没有实现日志compact功能。


### 重新加载

readCommits()
 |-recoverFromSnapshot()
   |-json.Unmarshal()

示例实现的代码很简单。

示例应用的snapshot的加载也使用了etcd自身提供的snapshot管理组件，其特点是加载过程中会优先选择最新的snapshot，只有当前snapshot被破坏了才会选择更旧一点的snapshot。





总结

通过上面示例分析，我们了解到，如果应用程序需要使用etcd-raft实现一个分布式系统，就必须要在该library的基础上增加如下子系统：

WAL: 即日志系统，应用程序需要负责日志的append和load；
Snapshot: 负责状态机的定期快照和WAL日志的回收；
应用状态机：实现自己的应用逻辑
raft协议消息的网络收发
而etcd-raft库只是实现了raft协议的核心部分，包括：

选主
多节点一致性语义实现
节点变更
etcd-raft和应用之间是通过channel进行消息的通信，而消息的结构也是由raft库定义好。具体来说，应用通过raft库提供的Ready()接口获取到消息传输管道，并从该管道接收raft库发出的各种指令(Message)，最后再通过Advance()通知raft库命令处理结果。应用处理指令的典型流程是：

将指令写入WAL日志
将指令写入raft组件内存中（为什么要做这个？）
将消息中指定的已经commit的日志进行提交，也即：应用到应用状态机中
调用Advance接口，应该是通知raft当前命令执行完成，可以继续提供下一条指令了。
因此，raft模块所需要完成的工作就相对比较简单了：

为应用准备好需要执行的指令，这些指令是根据raft协议而定义的
应用在执行完成指令后通知raft，raft根据该指令的执行结果（例如，该指令是否已经在多数节点上完成执行）决定是否向前推进commit index，并且，raft会继续向应用准备下一条指令
需要注意的一点是：所有的客户端请求都是直接发往应用的。应用需要将这些请求先提交给raft组件以保证在集群多数节点之间完成数据同步。应用提交的过程其实就是调用raft模块的Propose()接口。


-->












## 参考

<!--
https://zhuanlan.zhihu.com/distributed-storage

RAFT C语言的实现
https://github.com/willemt/raft

Leader election
Log replicationLog compaction
Membership changesLeader transfer
Linearizable/Lease read


基本流程是？
优化点包含了哪些？
核心处理流程：A) AppendLog；B) 选主；C) Snapshot；D) 成员变更等。
存储的接口通过 type Storage interface 指定，其中示例中直接使用了库中的 MemoryStorage 实现，每次从 WAL 和 Snapshot 中读取并恢复到内存中。
-->

{% highlight text %}
{% endhighlight %}
