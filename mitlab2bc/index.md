# MIT-6.824-Lab2BC-Raft-LogReplication-Persistence

这次实验主要实现 Log 的一致性和持久性。主要是 Debug 的过程中会遇到非常多的问题。这里对遇到的问题做一下回顾。这里主要是对自己的实现做一个记录和总结，常见的问题还是需要去参考 [Raft](https://raft.github.io/raft.pdf) 和 [students-guide-to-raft](https://thesquareplanet.com/blog/students-guide-to-raft/)。首先对论文里面的 Figure2 实现一次，再参照 guide 顺着往下阅读一遍，一边对自己的实现进行修改，就会发现自己很多实现上的 bug 了。我磕磕绊绊地做了将近两周的时间才基本把 2B 和 2C 的测试跑通。有些 Bug 在很多次重复测试后才会偶尔出现一两次，也使得调试变得比较麻烦。注意自己不要参考太多其他人的实现，那样的话有可能会被误导，还是要以论文和 guide 为主要参考。

## 定时器的实现
一开始的实现非常简陋，对定时器的实现非常不严谨，误打误撞过了 2B 的 Test，结果在 2C 的测试中吃了大苦，不得不重新回头重构定时器的机制。2B 这里主要有三个牵涉到定时的地方：
- Election Timer： 到时后决定是否需要发起选举。
- HeartBeat Timer： Leader 在这个定时器到时后，发出 AppendEntries RPC 请求。
- Append Entries timeout Timer： 心跳包的超时定时器。

一开始的定时器实现，混杂了 `Loop` 和 `time.Sleep`，甚至有些定时器会互相影响。比如一个 server 它的选举和发出心跳包的 `Loop` 是同一个，这样的实现无疑是非常粗糙的。最后的做法是，一个 server 在 make 后就常驻一个 `CheckElection()` 线程，定期用 `time.Sleep()` 检查是否可以发起选举。一旦满足选举的条件，再进行选举。同时使用一个 `GotHeartBeat` 的标志量来记录定时器的状态。这样便可以不破坏定时器的精确度。

另外当 server 成为 Leader 后，会启动 `SendHeartBeat()` 线程，这里的定时器实现也是与上面相似的。
另外是两个超时定时器的实现。首先需要说明为了 RPC 的并发性，回调接收是通过 channel 来进行处理的。这里的定时器也就顺理成章地使用 `select` 来实现了。为了正确关闭 channel，还需要使用一个回收线程去 `close()` 这个 channel，否则会有一大堆的阻塞的线程。

```go
func (rf *Raft) StartElection() {

	rf.mu.Lock()

	if rf.Role != 1 {
		rf.mu.Unlock()
		return
	}

	ch := make(chan RequestVoteReply)
	args := RequestVoteArgs{
		Term:         rf.CurrentTerm,
		CandidateId:  rf.me,
		LastLogIndex: len(rf.Log) - 1,
		LastLogTerm:  rf.Log[len(rf.Log)-1].Term,
	}

	for i := 0; i < len(rf.peers); i++ {
		if i == rf.me {
			continue
		}

		go rf.sendRequestVote(i, &args, ch)

	}

	rf.mu.Unlock()

	agree := 1
	randomNum := ElectionTimer + rand.Intn(50)
	ElectionTimeoutChan := time.After(time.Millisecond * time.Duration(randomNum))

	i := 0
	for {
		select {
		case reply := <-ch:
			// DPrintf("server %d GOT vote reply term = %d", rf.me, reply.Term)
			rf.mu.Lock()
			if rf.Role != 1 {
				rf.mu.Unlock()
				return
			}
			if reply.Term > rf.CurrentTerm {
				rf.BecomeFollower(reply.Term)
				rf.mu.Unlock()
				// DPrintf("EXIT1")
				return
			} else if reply.Term < rf.CurrentTerm {
				rf.mu.Unlock()
				// DPrintf("EXIT2")
				// DPrintf("%d < %d", reply.Term, rf.CurrentTerm)
				continue
			} else if reply.VoteGranted {
				agree++
				// DPrintf("EXIT3")
			} else {
				// DPrintf("EXIT4")
			}
			if agree*2 > len(rf.peers) {
				rf.BecomeLeader()
				go rf.SendHeartbeat()
				go rf.closeChan(i, ch)
				rf.mu.Unlock()
				// log.Fatalf("Leader!!!\n")
				return
			}
			rf.mu.Unlock()
		case <-ElectionTimeoutChan:
			go rf.closeChan(i, ch)
			goto startAgain
		}
		i++
	}

startAgain:
	rf.mu.Lock()
	if rf.Role != 1 {
		rf.mu.Unlock()
		return
	}
	rf.BecomeCandidate()
	rf.mu.Unlock()
	go rf.StartElection()

}
```
## 快速恢复
当 Follower 的 Log 落后 Leader 很多时，可以在 Append Entries 的回复中添加 `Xterm` 和 `XIndex` 两个参数，反馈给 Leader 是哪个 term 或者哪个 index 开始落后的。这样可以快速地恢复 Follower，而不是按照 Figure2 里面那样要把 `nextIndex` 慢慢递减。一次递减会产生一次 RPC，这样的话有大量的冗余通信。如果不做这个优化的话，理论上不能通过 TestManyElection2A。

而我在刚开始的实现时，强制让一个 HeartBeat 去阻塞执行恢复一个 Follower 的日志。这样的话也可以通过这个测试。但是这样的设计不能通过 2C 的测试，因为这样并没有严格地实现心跳包。阻塞恢复 Log：
```go
// 这个实现是有问题的！！

// 恢复日志的实现函数
// 判断是否需要恢复，如果需要回复则创建线程去发送 AppendEntries RPC
func (rf *Raft) RecoverLog(reply *AppendEntriesReply, ch chan AppendEntriesReply) bool {

	server := reply.Id

	rf.NextIndex[server]--
	if rf.NextIndex[server] < 1 {
		rf.NextIndex[server] = 1
		return false
	}
	DPrintf("Try to recover %d nextIndex %d", server, rf.NextIndex[server])

	// retry
	lastIndex := rf.NextIndex[server] - 1
	prevLogTerm := -1
	if rf.NextIndex[server] != 0 {
		prevLogTerm = rf.Log[lastIndex].Term
	}

	args := AppendEntriesArgs{
		rf.CurrentTerm,
		rf.me,
		lastIndex,
		prevLogTerm,
		rf.Log[rf.NextIndex[server]:],
		rf.CommitIndex，
	}

	go rf.sendAppliedEntries(server, &args, ch)
	return true
}

// Leader 接受 AppendEntries 后
randomNum := AppendEntriesTimeout + rand.Intn(50)
timeoutChan := time.After(time.Millisecond * time.Duration(randomNum))
for i := 0; i < len(rf.peers)-1; i++ {
	// DPrintf("server %d waiting AppendEntriesReply", rf.me)
	select {
	case reply := <-ch:
		// DPrintf("server %d got heartbeat, id: %d", rf.me, reply.Id)
		if rf.CurrentTerm != term {
			rf.mu.Unlock()
			return
		}
		if reply.Term > rf.CurrentTerm {
			// DPrintf("HEARTBEAT")
			rf.BecomeFollower(reply.Term)
			rf.mu.Unlock()
			return
		} else if reply.Term < rf.CurrentTerm {
			continue
		} else if !reply.Success {
			DPrintf("Leader %d wants to recover %d", rf.me, reply.Id)

			// 阻塞等待恢复日志
			if rf.RecoverLog(&reply, ch) {
				i--
			}

			// 重置超时，等待新的 RPC 响应
			timeoutChan = time.After(time.Millisecond * time.Duration(randomNum))
		}
	case <-timeoutChan:
		// DPrintf("server %d AppendEntries timeout", rf.me)
		goto stopWatingHearbeat
	}
}
```
这样的实现，会使得 Leader 在发现某一个 Follower 的日志落后后，就一直不断向那个 Follower 发出 AppendEntries，不断递减 `nextIndex`，直到 Follower 的日志不再落后为止。期间 Leader 不能对其他 Follower 发出心跳包，这样在可靠的环境下当然是可以的，但是在不可靠的网络环境下，网络的变化会使得这样的恢复行为变得没有意义。

认识到这个问题后，在 AppendEntries 的回复参数中添加快速恢复的 `XTerm` 和 `XIndex`，实现快速恢复后这个问题也就被解决了。

## 一些 guide 里面得到解决的问题
对于一些遇到的，在 guide 里面有描述的问题，这里简单做一下记录。
### Start() 和 HeartBeat 的异同
在一开始，我认为 client 提交完一个 command 后，应该 Leader 马上去产生一堆 AppendEntries 去提交这个命令，而 HeartBeat 只是应该用于发送心跳和恢复 Follower。这样的实现确实可能没问题，但是在这两个行为里面，包含了大量雷同逻辑，实际上两者都是通过 AppendEntries 来实现的，正如 guide 里面所说，提交 command 完全可以通过 HeartBeat 来完成。
### 回应 ApplyMsg 的时机和 Commit 的一个小问题
什么时候去回应 client 呢，最直接的想法是用一个常驻线程，不断地去检查 `CommitIndex` 和 `LastApplied`。那无疑又是一些锁和资源的开销。其实只要在 `CommitIndex` 改变后做一次检查即可。此外，guide 中提到 Figure 8 的问题。

![/images/raft/Figure8.png](/images/raft/Figure8.png)

Leader 的 crash 可能会使得当其他的 server 成为 Leader 后，提交错误的日志。必须在 Leader 更新 `CommitIndex` 之前检查 `rf.CurrentTerm == rf.Log[rf.CommitIndex].Term`。

## 后记
经过两周的各种 Debug，除了并发测试量特别高时（比如 100 个测试并发），`Figure 8 (unreliable)` 会超时几秒钟以外，代码连续 5000 次通过了测试，这个 Lab 暂且算完成了。要完成这个 Lab 真的得把 Raft 的每个细节都理解清楚。非常建议使用 go-test-many.sh 脚本来并发测试，否则有些 Bug 死活测不出来，然后在某一次测试中突然跑出来闹事。另外在跑测试的时候，建议喝杯水放松放松，Bug 该来还是会来的。最后附上测试结果：

```log
Test (2A): initial election ...
  ... Passed --   3.1  3  110   31908    0
Test (2A): election after network failure ...
  ... Passed --   5.0  3  230   46648    0
Test (2A): multiple elections ...
  ... Passed --   5.6  7  936  201618    0
PASS
ok      6.824/raft      13.745s

Test (2B): basic agreement ...
  ... Passed --   0.7  3   16    4588    3
Test (2B): RPC byte count ...
  ... Passed --   1.5  3   48  114552   11
Test (2B): agreement despite follower disconnection ...
  ... Passed --   5.6  3  210   59199    8
Test (2B): no agreement if too many followers disconnect ...
  ... Passed --   3.6  5  352   83558    4
Test (2B): concurrent Start()s ...
  ... Passed --   0.6  3   12    3440    6
Test (2B): rejoin of partitioned leader ...
  ... Passed --   4.6  3  262   59749    4
Test (2B): leader backs up quickly over incorrect follower logs ...
  ... Passed --  16.2  5 2316 1827613  102
Test (2B): RPC counts aren't too high ...
  ... Passed --   2.0  3   66   19678   12
PASS
ok      6.824/raft      34.825s

Test (2C): basic persistence ...
  ... Passed --   3.9  3  126   33452    6
Test (2C): more persistence ...
  ... Passed --  15.5  5 1828  369618   16
Test (2C): partitioned leader and one follower crash, leader restarts ...
  ... Passed --   1.8  3   42   11544    4
Test (2C): Figure 8 ...
  ... Passed --  32.2  5 1120  247514   41
Test (2C): unreliable agreement ...
  ... Passed --   3.1  5  216   77507  246
Test (2C): Figure 8 (unreliable) ...
  ... Passed --  94.0  5 16504 17911361  142
Test (2C): churn ...
  ... Passed --  16.4  5 1296 1341742  384
Test (2C): unreliable churn ...
  ... Passed --  16.2  5  888  495533  392
PASS
ok      6.824/raft      183.121s
```
