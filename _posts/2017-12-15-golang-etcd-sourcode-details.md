---
title: ETCD 源码解析
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



<!--
https://zhuanlan.zhihu.com/distributed-storage

######################################
## RAFT
######################################
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

## ETCD

RAFT 协议的实现主要包括了四部分：协议逻辑、存储、消息序列化和网络传输，而 ETCD 对应的 RAFT 库只实现了最核心算法。

源码可以直接下载 [Github coreos/etcd](https://github.com/coreos/etcd) 其中有一个简单的示例 [contrib/raftexample](https://github.com/coreos/etcd/tree/master/contrib/raftexample) 。

### 数据结构

简单介绍下一些常见的数据结构。

#### type node struct

在 `raft/node.go` 中定义了 `type node struct` 对应的结构，一个 RAFT 结构通过 Node 表示各结点信息，该结构体内定义了各个管道，用于同步信息，下面会逐一遇到。

{% highlight go %}
type node struct {
    propc      chan pb.Message
    recvc      chan pb.Message
    confc      chan pb.ConfChange
    confstatec chan pb.ConfState
    readyc     chan Ready
    advancec   chan struct{}
    tickc      chan struct{}
    done       chan struct{}
    stop       chan struct{}
    status     chan chan Status
}
{% endhighlight %}

其实现，就是通过这些管道在 RAFT 实现与外部应用之间来传递各种消息。

#### type raft struct

在 `raft/raft.go` 中定义了 `type raft struct` 结构，其中有两个关键函数指针 `tick` 和 `step`，在不同的状态时会调用不同的函数，例如 Follower 中使用 `tickElection()` 和 `stepFollower()` 。

{% highlight text %}
type raft struct {
	id uint64

	Term uint64
	Vote uint64

	readStates []ReadState

	// the log
	raftLog *raftLog

	maxInflight int
	maxMsgSize  uint64
	prs         map[uint64]*Progress

	state StateType

	votes map[uint64]bool

	msgs []pb.Message

	// the leader id
	lead uint64
	// leadTransferee is id of the leader transfer target when its value is not zero.
	// Follow the procedure defined in raft thesis 3.10.
	leadTransferee uint64
	// New configuration is ignored if there exists unapplied configuration.
	pendingConf bool

	readOnly *readOnly

	// number of ticks since it reached last electionTimeout when it is leader
	// or candidate.
	// number of ticks since it reached last electionTimeout or received a
	// valid message from current leader when it is a follower.
	electionElapsed int

	// number of ticks since it reached last heartbeatTimeout.
	// only leader keeps heartbeatElapsed.
	heartbeatElapsed int

	checkQuorum bool
	preVote     bool

	heartbeatTimeout int
	electionTimeout  int
	// randomizedElectionTimeout is a random number between
	// [electiontimeout, 2 * electiontimeout - 1]. It gets reset
	// when raft changes its state to follower or candidate.
	randomizedElectionTimeout int

	tick func()          // 两个重要的函数指针
	step stepFunc

	logger Logger
}
{% endhighlight %}

Node 代表了 etcd 中一个节点，是 RAFT 协议核心部分实现的代码，而在 EtcdServer 的应用层与之对应的是 raftNode ，两者一对一，raftNode 中有匿名嵌入了 node 。

### 整体框架

这里的采用的是异步状态机，基于 GoLang 的 Channel 机制，RAFT 状态机作为一个 Background Thread/Routine 运行，会通过 Channel 接收上层传来的消息，状态机处理完成之后，再通过 Ready() 接口返回给上层。


<!--
RAFT 交互流程相关的内容都放在 raftNode 中，而节点状态、IO调用、事件触发起点等入口都放在了 node 中，两者都在启动后起了一个 for-select 结构的协程循环处理各自负责的事件。
1) 递增currentTerm，投票给自己；2) 重置ElectionTimer；3) 向所有的服务器发送 RequestVote RPC请求


## 示例程序

在 `contrib/raftexample` 中有个简单的 kvstore 示例程序。

main()
  |-newRaftNode() 返回的结构体会作为底层的RAFT协议与上层应用的中间结合体
  | |-raftNode() 新建raftNode对象，重点proposeC
  | |-raftNode.startRaft()  启动，示例中的代码
  |   |-os.Mkdir() 如果snapshot目录不存在则创建
  |   |-raftNode.replayWAL() 重放WAL日志
  |   |-raftNode.serveRaft() 主要是启动网络监听
  |   |-raftNode.serveChannels() 监听配置添加等命令
  |     |-raft.Node.Propose() 阻塞等待该用户请求被RAFT状态机接受
  |     |-raft.Node.ProposeConfChange()
  |
  |-newKVStore() 创建内存KV存储结构
  | |-kvstore() 初始化KVStore存储对象
  | |-kvstore.readCommits() 存储使用的核心，读取已经提交的数据
  |   |-gob.NewDecoder() 反序列化
  |   |-kvStore[] 保存到内存中
  |
  |-serveHttpKVAPI() 启动对外提供服务的HTTP端口
    |-srv.ListenAndServe() 真正启动客户端的监听服务

上述的 HTTP 端口中真正处理请求的函数为 ServeHTTP() 函数。

ServeHTTP()
  |====> PUT方法
  |-ioutil.ReadAll() 读取请求
  |-kvstore.Propose() 提交请求，会阻塞直到RAFT状态机提交成功
  | |-buf.String() 向Channel proposeC中发送请求
  |-http.ResponseWriter.WriteHeader() 返回数据
  |
  |====> GET方法
  |-kvstore.Lookup() GET方法，查找并返回数据

###　提交数据

在接收到添加数据的请求时，需要先提交至 RAFT 组件在集群之间对本次提交达成一致。实现时，是将这次请求通过 golang 的 channel 提交给应用的 raftNode 结构：

func (s *kvstore) Propose(k string, v string) {
    var buf bytes.Buffer
    if err := gob.NewEncoder(&buf).Encode(kv{k, v}); err != nil {
        log.Fatal(err)
    }
	s.proposeC <- buf.String()
}

kvstore.proposeC 便是应用初始化时创建的与 raftNode 结构之间进行请求传输的 channel，对于更新请求，通过该管道将数据提交给了 raftNode 。

在 raftNode.serveChannels() 函数中，实际上会单独起一个协程处理该请求，最终阻塞到 raftNode.raft.Node.Propose() 函数中，直到完成提交后给客户端返回请求。

### RAFT指令处理

客户端的请求发送到 RAFT 组件后，最终还是通过 Ready() 暴露给应用，由应用负责写日志，发送给集群 Follower 节点等。

同样是在 raftNode.serveChannels() 中实现，其中相关代码如下：

func (rc *raftNode) serveChannels() {
    // event loop on raft state machine updates
    for {
        select {
            // 1. 通过Ready()获取到RAFT的核心指令，获取所有已经完成日志写入的请求
        case rd := <-rc.node.Ready():
            // 2. 写WAL日志
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
			// 6. 完成状态机应用后，通知底层raft组件状态机应用完毕，RAFT组件可以准备下一次Ready消息了
            rc.node.Advance()
        }
    }
}
-->

<!--
## 代码走读

在 `raft/node.go->run()` 函数中，是一个节点 (Node) 的主要处理过程，开始处于 Follower 状态，然后随着 `case <-n.tickc` 进行，开始进入选举。


ETCD 服务器是通过 EtcdServer 结构抽象，对应了 etcdserver/server.go 中的代码，包含属性 r raftNode，代表 RAFT 集群中的一个节点。
在server启动的过程中，会调用raftNode(etcdserver/raft.go)的start方法：

### 数据结构


在 campaign() 中的实现选举逻辑时，实际上实现了两个阶段 PreElection 和 Election 。？？？？
## 1. Leader选举

EtcdServer.Start()
 |-EtcdServer.start()
   |-EtcdServer.run()
     |-raftNode.start() 启动新的协程处理请求
       |-raftNode.Tick()  对于r.ticker.C管道的时钟处理
       |- 针对r.Ready()的请求

raftNode.start()

开始启动的时候，首先会设置为 Follower 状态，而且 term 为 1 。

当 node 初始化完成之后，通过 `node.run()` 开始运行，这里会启动单独的协程读取如上的管道。


NewServer() 通过配置创建一个新的EtcdServer对象etcdserver/server.go
 |-startNode() 新建一个节点，前提是没有WAL日志raft.go
 | |-StartNode() 启动一个节点raft/node.go，开始node的处理过程<<<start>>>
 |   |-newRaft() 创建RAFT对象raft/raft.go
 |   |-raft.becomeFollower() 这里会对关键对象初始化以及赋值，包括step=stepFollower r.tick=r.tickElection函数
 |   | |-raft.reset() 开始启动时设置term为1
 |   |   |-raft.resetRandomizedElectionTimeout() 更新选举的随机超时时间
 |   |   |-
 |   |-newNode() 新建节点
 |   |-node.run() 节点运行raft/node.go <<<messages>>>
 |     |-newReady() 新建type Ready对象
 |     |-raft.tick()  等待n.tickc管道，这里实际就是在上面赋值的tickElection()函数
 |
 |-time.NewTicker() 在通过&EtcdServer{}创建时新建tick时钟

raft.tickElection() 时钟处理函数，默认时间间隔为500ms raft/raft.go
 |-raft.Step() 如果超时，则开始发起选举，构造消息发送给自己，消息类型为MsgHup
   |-raft.campaign() 开始进入选举
     |-raft.becomeCandidate() 进入选举后状态由Follower转换为Candidate
	 | |-指针函数修改step=stepCandidate tick=tickElection
	 | |-raft.reset() 重置，同时term会增加1
     |-raft.send() 向每个节点发送voteMsg消息

### 消息发送

通过 `raft.send()` 发送消息时，最终会调用 `append(r.msgs, m)`，那么这个消息是在什么时候消费的呢？

在 `type node struct` 结构体中，存在一个 readyc 的管道。

type node struct {
	readyc chan Ready
}

在 raft/node.go 中存在一个 node.run() 函数，会读取所有的消息，然后同时通过管道发送。？？？如果没有消息更新，这里会阻塞吗？？？？

func newReady(r *raft, prevSoftSt *SoftState, prevHardSt pb.HardState) Ready {
	rd := Ready{
	    Entries:          r.raftLog.unstableEntries(),
		CommittedEntries: r.raftLog.nextEnts(),
		Messages:         r.msgs,
	}
	... ...
}

也就是说，在 node.go 里 node.run() 中构建了 Ready 对象，对象里就包涵被赋值的 msgs，并最终写到 node.readyc 这个管道里，如下是对应这个 case 的实现：

case readyc <- rd:
	r.msgs = nil
    r.readStates = nil
    advancec = n.advancec

这里的 msgs 已经读取过并写入到了管道中，直接设置为空，并会赋值 advancec，在 etcdserver/raft.go 的 `raftNode.start()` 中，会起一个单独的协程读取数据；其中读取的函数实现在 raft/node.go 中：

func (n *node) Ready() <-chan Ready { return n.readyc }

应用层 (也就是etcd) 读取到的 Ready 里面包含了 Vote 消息，会调用网络层发送消息出去，并且调用 Advance() 。

### 消息接收处理

其它 node 接收到网络层消息后，会调用 raft.Step() 函数。

raft.Step()  raft/raft.go
 |-raft.becomeFollower() 如果本地的term小于消息的term，就把自己置为follower
 |-raft.send() 当接收到的消息term一致时，就返回voteRespMsg为其投票

voteRespMsg 的返回信息被之前的发送方接收到了之后，就会计算收到的选票数目是否大于所有 node 的一半，如果大于则自己成为 leader，否则将自己置为 follower；

stepCandidate() raft/raft.go
 |-[case myVoteRespType] 如果收到了投票返回的消息
   |-raft.poll() 检查是否满足多数派原则


case myVoteRespType:
    gr := r.poll(m.From, m.Type, !m.Reject)
    switch r.quorum() {
    case gr:
        if r.state == StatePreCandidate {
            r.campaign(campaignElection)
        } else {
            r.becomeLeader()
            r.bcastAppend()
        }
    case len(r.votes) - gr:
        r.becomeFollower(r.Term, None)
    }
在成为leader之后，和上面的两个角色一样的，最重要的是step被置为了stepLeader，具体stepLeader中涉及到的一些操作，更多的是下一个问题会用到，这里就不多说了。

func (r *raft) becomeLeader() {
    r.step = stepLeader
}
-->

## Progress

RAFT 实现的内部，本身还维护了一个子状态机。

{% highlight text %}
                            +--------------------------------------------------------+
                            |                  send snapshot                         |
                            |                                                        |
                  +---------+----------+                                  +----------v---------+
              +--->       probe        |                                  |      snapshot      |
              |   |  max inflight = 1  <----------------------------------+  max inflight = 0  |
              |   +---------+----------+                                  +--------------------+
              |             |            1. snapshot success
              |             |               (next=snapshot.index + 1)
              |             |            2. snapshot failure
              |             |               (no change)
              |             |            3. receives msgAppResp(rej=false&&index>lastsnap.index)
              |             |               (match=m.index,next=match+1)
receives msgAppResp(rej=true)
(next=match+1)|             |
              |             |
              |             |
              |             |   receives msgAppResp(rej=false&&index>match)
              |             |   (match=m.index,next=match+1)
              |             |
              |             |
              |             |
              |   +---------v----------+
              |   |     replicate      |
              +---+  max inflight = n  |
                  +--------------------+
{% endhighlight %}

详细可以查看 [raft/design.md](https://github.com/coreos/etcd/blob/master/raft/design.md) 中的介绍，对于 Progress 实际上就是 Leader 维护的各个 Follower 的状态信息，总共分为三种状态：probe, replicate, snapshot 。

应该是 AppendEntries 接口的一种实现方式，为每个节点维护两个 Index 信息：A) matchIndex 已知服务器的最新 Index，如果还未确定则是 0；<!-- 作用是啥??????；--> B) nextIndex 用来标示需要从那个索引开始复制。那么 Leader 实际上就是将 nextIndex 到最新的日志复制到 Follower 节点。

<!--
如果多数派已经收到了请求，并进行了相关的处理，在响应之前主崩溃，那么新的日志可能在集群写入成功吗???????
-->



## 参考

两种不同的实现方式 [Github CoreOS-etcd](https://github.com/coreos/etcd)、[Github Hashicorp-raft](https://github.com/hashicorp/raft) 。

<!--
据说一个性能比lmdb好很多的存储引擎
https://github.com/leo-yuriev/libmdbx




RAFT论文的中文翻译
http://www.opscoder.info/ectd-raft-library.html
http://vlambda.com/wz_xberuk7dlD.html
http://chenneal.github.io/2017/03/16/phxpaxos%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8B%E4%B8%80%EF%BC%9A%E8%B5%B0%E9%A9%AC%E8%A7%82%E8%8A%B1/
https://www.jianshu.com/p/ae462a2d49a8
https://www.jianshu.com/p/ae1031906ef4
http://blog.neverchanje.com/2017/01/30/etcd_raft_core/
http://www.cnblogs.com/foxmailed/p/7173137.html
https://www.jianshu.com/p/5aed73b288f7

https://zhuanlan.zhihu.com/p/27767675
-->


{% highlight text %}
{% endhighlight %}
