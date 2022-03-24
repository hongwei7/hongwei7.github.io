# MIT-6.824-Lab1-MapReduce 实验


这次实验的目的是自己完成一个 MapReduce 的分布式实现。对我来说最大的困难在于语言的不熟悉，这里主要使用的是 Go 语言。由于这是我第一次接触 Go，有很多实现细节其实可以做的更好一些。

## 基本要求

- master 进程分配 m 个 Map 任务和 n 个 Reduce 任务给 worker，worker 完成任务后通知 master。
- 若 worker 完成的是 Map 任务，则完成后把结果返回给 master，master 做哈希映射到 n 个 Reduce 模块。若完成的是 Reduce 任务，则写入磁盘后通知 master 任务已完成。
- Reduce 任务需要等到所有 Map 任务完成后再执行。
- 为了通过最后的 crash 测试，当 worker 超时，master 会分配新的 worker 去完成任务。这里需要保证对分配重复任务的 worker，只取一次的结果。

{{< admonition info "Task RPC" true >}}

本着简单实现的想法，设计了以下 RPC 结构。

```go
type Task struct {
	Seq         int
	Status      int // 0  1  2  3  4 
	MapFile     string
	ReduceFiles []KeyValue
	LastTime    time.Time
}
```
这样就可以完整描述一个 Map 任务或者是一个 Reduce 任务了。
{{< /admonition >}}
{{< admonition info "master" true >}}

接着是 master 的结构设计。

```go
type Coordinator struct {
	TaskQueue        []Task
	NReduce          int
	MapFinished      bool
	ReduceFinished   bool
	RemainMapTask    int
	RemainReduceTask int
	Mu               sync.Mutex
}
```
这里用一个数组来记录任务队列。`MapFinished` 和 `ReduceFinished` 代表的是 Map 任务是否已经全部完成以及 Reduce 任务是否已经全部完成。
{{< /admonition >}}

{{< admonition info "More RPC" true >}}
设计两个 RPC 接口，一个是请求任务的接口，一个是通知完成任务的接口。注意这里使用 `checkTime()` 函数来检查任务是否超时，一旦超时，则分配新的 worker 去完成任务。为了简化处理，这里没有对超时的 woker 做回收，而是忽略它通知完成任务的行为。

```go
func checkTime(oldTime time.Time) bool {
	m, _ := time.ParseDuration("-10s")
	m1 := time.Now().Add(m)
	if m1.After(oldTime) {
		fmt.Println("delay occurs")
	}
	return m1.After(oldTime)
}

func (c *Coordinator) RequestForTask(request int, reply *Task) error {
	// fmt.Printf("%d ask for Task, Map Remaining %d, Reduce Remaining %d\n", request, c.RemainMapTask, c.RemainReduceTask)
	c.Mu.Lock()
	defer c.Mu.Unlock()
	if !c.MapFinished {
		for i := c.NReduce; i < len(c.TaskQueue); i++ {
			if c.TaskQueue[i].Status == 0 || (c.TaskQueue[i].Status == 1 && checkTime(c.TaskQueue[i].LastTime)){
				c.TaskQueue[i].Status = 1
				c.TaskQueue[i].LastTime = time.Now()
				*reply = c.TaskQueue[i]
				return nil
			}
		}
	} else if !c.ReduceFinished {
		for i := 0; i < c.NReduce; i++ {
			if c.TaskQueue[i].Status == 2 || (c.TaskQueue[i].Status == 3 && checkTime(c.TaskQueue[i].LastTime)) {
				c.TaskQueue[i].Status = 3
				c.TaskQueue[i].LastTime = time.Now()
				*reply = c.TaskQueue[i]
				return nil
			}
		}
	}

	if !c.ReduceFinished {
		(*reply).Seq = -1
	} else {
		(*reply).Seq = -2
	}
	return nil
}

func (c *Coordinator) SubmmitTask(t Task, p *int) error {
	c.Mu.Lock()
	defer c.Mu.Unlock()

	if c.TaskQueue[t.Seq].Status == -1 {
		return nil
	}

	status := t.Status

	// fmt.Println(c.TaskQueue[seq].ReduceFiles)

	if status == 1 {
		// fmt.Printf("Receving %s\n", t.MapFile)
		for _, wi := range t.ReduceFiles {
			i := ihash(wi.Key) % c.NReduce
			c.TaskQueue[i].ReduceFiles = append(c.TaskQueue[i].ReduceFiles, wi)
		}
		c.RemainMapTask--
		if c.RemainMapTask == 0 {
			c.MapFinished = true
		}
	} else if status == 3 {
		c.RemainReduceTask--
		if c.RemainReduceTask == 0 {
			c.ReduceFinished = true
		}
	}
	c.TaskQueue[t.Seq].Status = -1
	return nil
}
```
{{< /admonition >}}

使用 `./test-mr.sh` 测试，结果如下

```
./test-mr.sh
../../mrapps/wc.so
*** Starting wc test.
2021/06/02 17:49:08 rpc.Register: method "Done" has 1 input parameters; needs exactly three
--- wc test: PASS
../../mrapps/indexer.so
*** Starting indexer test.
2021/06/02 17:49:18 rpc.Register: method "Done" has 1 input parameters; needs exactly three
--- indexer test: PASS
*** Starting map parallelism test.
2021/06/02 17:49:24 rpc.Register: method "Done" has 1 input parameters; needs exactly three
--- map parallelism test: PASS
*** Starting reduce parallelism test.
2021/06/02 17:49:33 rpc.Register: method "Done" has 1 input parameters; needs exactly three
--- reduce parallelism test: PASS
*** Starting job count test.
2021/06/02 17:49:44 rpc.Register: method "Done" has 1 input parameters; needs exactly three
--- job count test: PASS
*** Starting early exit test.
2021/06/02 17:50:03 rpc.Register: method "Done" has 1 input parameters; needs exactly three
--- early exit test: PASS
*** Starting crash test.
../../mrapps/nocrash.so
2021/06/02 17:50:12 rpc.Register: method "Done" has 1 input parameters; needs exactly three
delay occurs
delay occurs
delay occurs
delay occurs
delay occurs
delay occurs
2021/06/02 17:50:51 dialing:dial unix /var/tmp/824-mr-1000: connect: connection refused
2021/06/02 17:50:51 dialing:dial unix /var/tmp/824-mr-1000: connect: connection refused
2021/06/02 17:50:51 dialing:dial unix /var/tmp/824-mr-1000: connect: connection refused
--- crash test: PASS
*** PASSED ALL TESTS
```

{{< admonition success "结果：" true >}}
可以看到所有测试都通过了，到这里Lab1就做完了。
{{< /admonition >}}

