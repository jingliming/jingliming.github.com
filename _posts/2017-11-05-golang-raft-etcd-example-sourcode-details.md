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


## 示例程序

RAFT 协议的实现主要包括了四部分：协议逻辑、存储、消息序列化和网络传输，而 ETCD 对应的 RAFT 库只实现了最核心算法。

源码可以直接下载 [Github coreos/etcd](https://github.com/coreos/etcd) 其中有一个简单的示例 [contrib/raftexample](https://github.com/coreos/etcd/tree/master/contrib/raftexample)，这是一个内存的 KV 存储引擎。


<!--
RAFT 交互流程相关的内容都放在 raftNode 中，而节点状态、IO调用、事件触发起点等入口都放在了 node 中，两者都在启动后起了一个 for-select 结构的协程循环处理各自负责的事件。
1) 递增currentTerm，投票给自己；2) 重置ElectionTimer；3) 向所有的服务器发送 RequestVote RPC请求
-->

### 启动流程

{% highlight text %}
main()                                  main.go
  |-newRaftNode()                       raft.go 返回结构体会作为底层RAFT协议与上层应用的中间结合体
  | |-raftNode()                        <<<1>>>新建raftNode对象，重点proposeC
  | |-raftNode.startRaft()              启动，示例中的代码
  |   |-os.Mkdir()                      如果snapshot目录不存在则创建
  |   |-raftNode.replayWAL()            重放WAL日志
  |   |-raftNode.serveRaft()            主要是启动网络监听
  |   |-raftNode.serveChannels()        raft.go 启动协程，监听配置添加等命令
  |     |-raft.Node.Propose()           阻塞等待该用户请求被RAFT状态机接受
  |     |-raft.Node.ProposeConfChange()
  |
  |-newKVStore()                        kvstore.go 创建内存KV存储结构
  | |-kvstore()                         初始化KVStore存储对象
  | |-kvstore.readCommits()             会启动一个协程，也是存储的核心，用于读取已经提交的数据
  |   |-gob.NewDecoder()                反序列化
  |   |-kvStore[]                       保存到内存中
  |
  |-serveHttpKVAPI()                    启动对外提供服务的HTTP端口
    |-srv.ListenAndServe()              真正启动客户端的监听服务
{% endhighlight %}

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


### 1. 新建raftNode对象

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

### 2. 写数据

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

{% highlight text %}
ServeHTTP()                httpapi.go
  |====> PUT方法
  |-ioutil.ReadAll()       从HTTP中读取请求
  |-kvstore.Propose()      kvstore.go 正式提交请求，阻塞直到RAFT状态机提交成功
  | |-glob.NewEncoder()    序列化
  | |-buf.Bytes()          通过proposeC管道发送请求到RAFT核心
  |
  |-http.ResponseWriter.WriteHeader() 返回数据结果
  |
  |====> GET方法
  |-kvstore.Lookup()       查找并返回数据
{% endhighlight %}





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
