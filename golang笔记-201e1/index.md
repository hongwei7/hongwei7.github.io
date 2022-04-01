# 

# Golang笔记

## 基础

### 基本语法

- 通过官网 A Tour Of Go 学习 golang 基本语法。

### struct

- golang 通过 struct 来实现其他语言中的 class，但是在 golang 中没有继承的概念。
- golang 中使用组合函数的方法来定义结构体函数：
    
    ```go
    func (person Profile) FmtProfile() {
        fmt.Printf("名字：%s\n", person.name)
        fmt.Printf("年龄：%d\n", person.age)
        fmt.Printf("性别：%s\n", person.gender)
    }
    ```
    
- 当需要改变对象内容或者结构体过大需要性能提升时，使用指针作为方法接收者。
    
    ```go
    func (person *Profile) FmtProfile() {
        fmt.Printf("名字：%s\n", person.name)
        fmt.Printf("年龄：%d\n", person.age)
        fmt.Printf("性别：%s\n", person.gender)
    }
    ```
    
- golang 使用组合的方法来实现继承。
- golang 使用接口（Interface）来实现多态。

```go
type Goodinterface {
    settleAccount() int
    orderInfo() string
}

type Phonestruct {
    name string
    quantity int
    price int
}

func (phone Phone) settleAccount() int {
return phone.quantity * phone.price
}
func (phone Phone) orderInfo() string{
return "您要购买" + strconv.Itoa(phone.quantity)+ "个" +
        phone.name + "计：" + strconv.Itoa(phone.settleAccount()) + "元"
}

type FreeGiftstruct {
    name string
    quantity int
    price int
}

func (gift FreeGift) settleAccount() int {
return 0
}
func (gift FreeGift) orderInfo() string{
return "您要购买" + strconv.Itoa(gift.quantity)+ "个" +
        gift.name + "计：" + strconv.Itoa(gift.settleAccount()) + "元"
}

func calculateAllPrice(goods []Good) int {
var allPrice int
for _,good :=range goods{
        fmt.Println(good.orderInfo())
        allPrice += good.settleAccount()
    }
return allPrice
}
```

- golang 利用反射机制来实现 Tag。

### 反射

获取一个对象的类型，属性及方法，这个过程其实就是 **反射**。

reflect包最核心的两个函数为：

- reflect.TypeOf(i)
- reflect.ValueOf(i)

两个函数分别返回 reflect.Type 和 reflect.Value 类型。

优点：

- 可以在一定程度上避免硬编码，提供灵活性和通用性。
- 可以作为一个[第一类对象](https://zh.wikipedia.org/wiki/%E7%AC%AC%E4%B8%80%E9%A1%9E%E7%89%A9%E4%BB%B6)发现并修改源代码的结构（如代码块、类、方法、协议等）。
- 可以在运行时像对待源代码语句一样动态解析字符串中可执行的代码（类似JavaScript的eval()函数），进而可将跟class或function匹配的字符串转换成class或function的调用或引用。
- 可以创建一个新的语言字节码解释器来给编程结构一个新的意义或用途。

缺点：

- 此技术的学习成本高。面向反射的编程需要较多的高级知识，包括框架、关系映射和对象交互，以实现更通用的代码执行。
- 同样因为反射的概念和语法都比较抽象，过多地滥用反射技术会使得代码难以被其他人读懂，不利于合作与交流。
- 由于将部分信息检查工作从编译期推迟到了运行期，此举在提高了代码灵活性的同时，牺牲了一点点运行效率。

### 类型断言

通过类型断言可以判断某个对象不是nil，且为某个类型。

```go
t := i.(T)     // 会触发panic
t, ok:= i.(T)  // 不会触发panic
```

甚至可以通过 type switch 来实现不同的行为：

```go
func findType(i interface{}) {
    switch x := i.(type) {
    case int:
        fmt.Println(x, "is int")
    case string:
        fmt.Println(x, "is string")
    case nil:
        fmt.Println(x, "is nil")
    default:
        fmt.Println(x, "not type matched")
    }
}
```

### goroutine 和 channel

golang 原生支持 gotoutine 协程。

channel 的容量不同，会有不同的用法。

- 当容量为0时，说明信道中不能存放数据，在发送数据时，必须要求立马有人接收，否则会报错。此时的信道称之为**无缓冲信道**。
- 当容量为1时，说明信道只能缓存一个数据，若信道中已有一个数据，此时再往里发送数据，会造成程序阻塞。 利用这点可以利用信道来做锁。
- 当容量大于1时，信道中可以存放多个数据，可以用于多个协程之间的通信管道，共享资源。

注意 channel 传输的是否为引用：

- **值类型** ：String，Array，Int，Struct，Float，Bool
- **引用类型**：Slice，Map

若传入的是值类型，那么传输的是值；若传入的是引用类型，那么传输的是引用（指针）。

从已关闭的 channel 读取消息永远不会阻塞，并且会返回一个为 false 的值，用以判断该 channel 是否已关闭（x,ok := <- ch）

万能的 channel 使用公式：

[4.10 学习 Go 协程：万能的通道模型(公式) - Go编程时光 1.0.0 documentation](https://golang.iswbm.com/c04/c04_10.html)

### Context

实际上 Context 是一个接口：

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{})interface{}
}
```

- Deadline 返回截止时间。
- Done 返回只读的通道，当返回一个 struct{} 时，代表 parent context 发起了取消。
- Err 返回错误原因。
- Value 返回被绑定的值。

有四个常用的函数：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, valinterface{}) Context
```

1. 通常 Context 都是做为函数的第一个参数进行传递（规范性做法），并且变量名建议统一叫 ctx
2. Context 是线程安全的，可以放心地在多个 goroutine 中使用。
3. 当你把 Context 传递给多个 goroutine 使用时，只要执行一次 cancel 操作，所有的 goroutine 就可以收到 取消的信号
4. 不要把原本可以由函数参数来传递的变量，交给 Context 的 Value 来传递。
5. 当一个函数需要接收一个 Context 时，但是此时你还不知道要传递什么 Context 时，可以先用 context.TODO 来代替，而不要选择传递一个 nil。
6. 当一个 Context 被 cancel 时，继承自该 Context 的所有 子 Context 都会被 cancel。

## 进阶

### goroutine 的生成调度

golang 的 goroutine 使用的调度方式是 MPG 模型。

![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled.png)

一个M对应一个系统内核线程，一个P对应一个上下文，一个上下文连接一个或多个 G goroutine。蓝色的 G 代表运行中。

当一个 Goroutine 创建被创建时，Goroutine 对象被压入 Processor 的本地队列或者 Go 运行时 全局 Goroutine 队列。
Processor 唤醒一个 Machine，如果 Machine 的 waiting 队列没有等待被 唤醒的 Machine， 则创建一个（只要不超过 Machine 的最大值，10000），Processor 获取到 Machine 后，与 此 Machine 绑定，并执行此 Goroutine。

Machine 执行过程中，随时会发生上下文切换。当发生上下文切换时，需要对执行现场进行 保护，以便下次被调度执行时进行现场恢复。Go 调度器中 Machine 的栈保存在 Goroutine 对 象上，只需要将 Machine 所需要的寄存器(堆栈指针、程序计数器等)保存到 Goroutine 对象上 即可。
如果此时 Goroutine 任务还没有执行完，Machine 可以将 Goroutine 重新压入 Processor 的队 列，等待下一次被调度执行。
如果执行过程遇到阻塞并阻塞超时，Machine 会与 Processor 分离，并等待阻塞结束。此时 Processor 可以继续唤醒 Machine 执行其它的 Goroutine，当阻塞结束时，Machine 会尝试” 偷取”一个 Processor，如果失败，这个 Goroutine 会被加入到全局队列中，然后 Machine 将 自己转入 Waiting 队列，等待被再次唤醒。

### defer

defer 为延后执行语句，在 defer 归属的函数即将返回时，将延迟处理的语句按 defer 的逆序进行执行。通常作为资源的释放。

```go
func fileSize(filename string) int64 {
    f, err := os.Open(filename)
    if err != nil {
        return 0
    }
    // 延迟调用Close, 此时Close不会被调用
    defer f.Close()
    info, err := f.Stat()
    if err != nil {
        // defer机制触发, 调用Close关闭文件
        return 0
    }
    size := info.Size()
    // defer机制触发, 调用Close关闭文件
    return size
}
```

### recover 和 panic

- 有 panic 没 recover，程序宕机。
- 有 panic 也有 recover，程序不会宕机，执行完对应的 defer 后，从宕机点退出当前函数后继续执行。

```go
import "fmt"

func set_data(x int) {
    defer func() {
        // recover() 可以将捕获到的panic信息打印
        if err := recover(); err != nil {
            fmt.Println(err)
        }
    }()

    // 故意制造数组越界，触发 panic
    var arr [10]int
    arr[x] = 88
}

func main() {
    set_data(20)

    // 如果能执行到这句，说明panic被捕获了
    // 后续的程序能继续运行
    fmt.Println("everything is ok")
}
```

### array 和 slice

slice可以被认为动态数组，在内存中也是连续分配的。他可以动态的调整长度，可以通过内置的方法append来自动的增长slice长度;也可以通过再次切片来减少slice的长度。

用append给这个slice添加新值，返回一个新的slice，如果容量不够时，go会自动增加容易量，小于一1000个长度时成倍的增长，大于1000个长度时会以1.25或者25%的位数增长。

array 在函数中参数传递的是值，slice 传递的是指针。

新创建的slice2和slice1底层是同一个数组，所以修改任何一个，两个slice共同的指向元素，会导致同时修改的问题.

```go
		// 创建一个容量和长度均为6的slice
    slice1 := []int{5, 23, 10, 2, 61, 33}
    // 对slices1进行切片，长度为2容量为4
    slice2 := slice1[1:3]
    fmt.Println("cap", cap(slice2))
    fmt.Println("slice2", slice2)

    //修改一个共同指向的元素
    //两个slice的值都会修改
    slice2[0] = 11111
    fmt.Println("slice1", slice1)
    fmt.Println("slice2", slice2)
```

创建新的slice可以设置其cap，但是不能超过原底层数组的长度。此时对新的slice进行append，仍然会影响原来的slice。

```go
		// 创建一个容量和长度均为6的slice
    slice1 := []int{5, 23, 10, 2, 61, 33}
    // 对slices1进行切片，长度为2容量为3
    slice2 := slice1[1:3:4]
    fmt.Println("cap", cap(slice2))
    fmt.Println("slice2", slice2)

    //修改一个共同指向的元素
    //两个slice的值都会修改
    slice2[0] = 11111
    fmt.Println("slice1", slice1)
    fmt.Println("slice2", slice2)

    // 增加一个元素
    slice2 = append(slice2, 55555)

    fmt.Println(slice1)
    fmt.Println(slice2)
```

如果在创建新的slice时我们把他的长度和容量的值设置为样的值，那么在append新元素时，底层会创建一个新的array并把之前的值复制过去。这样就不会影响之前共同的底层array了。

```go
	// 创建一个容量和长度均为6的slice
    slice1 := []int{5, 23, 10, 2, 61, 33}
    // 对slices1进行切片，长度为2容量为3
    slice2 := slice1[1:3:3]
    fmt.Println("cap", cap(slice2))
    fmt.Println("slice2", slice2)

    //修改一个共同指向的元素
    //两个slice的值都会修改
    slice2[0] = 11111
    fmt.Println("slice1", slice1)
    fmt.Println("slice2", slice2)

    // 增加一个元素
    slice2 = append(slice2, 55555)

    fmt.Println("slice1: ", slice1)
    fmt.Println("slice2: ", slice2)
```

### 闭包外部变量的生命周期

- 若引用的是全局变量或者没有引用变量，则变量的生命周期为全局变量的生命周期。
- 若引用的是局部变量，每次调用完闭包，不会释放变量。这种场景下，是真正的闭包（函数+引用环境），并且以一个struct{FuncAddr, LocalAddr3, LocalAddr2, LocalAddr1}结构存储该闭包到堆中，等到调用闭包时，会把该结构地址提前放置一个寄存器，闭包内部通过该寄存器访问引用环境的变量。
- 当不再有变量引用闭包时，该闭包会被GC回收，局部变量也一样。

```go
func fib3(x int) func() int {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		x++
		return a+x
	}
}
```

![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled%201.png)

![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled%202.png)

![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled%203.png)

**所有引用局部变量，Golang在生成汇编是帮我们在堆上创建该变量的一个拷贝，并把该变量地址和函数闭包组成一个结构体，并把该结构体传出来作为返回值。**

### 垃圾回收

[Go 语言垃圾收集器的实现原理](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)

### 标记清除法

内存空间中包含多个对象，从根对象开始依次遍历子对象（标记过程），没有被遍历到的对象便为垃圾（随后在清除阶段清理）。

清理后的对象空间用链表结构串联起来（可以添加cookie或者标志位来实现）。

![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled%204.png)

问题：用户程序在标记清除的过程中不能执行（长时间的 STW ）。

### 三色抽象

三色标记法可以缩短STW时间。

- 白色：潜在的垃圾，可能被回收。
- 黑色：活跃的对象，包括不存在任何外部指针引用，以及从根对象不可达的。
- 灰色：活跃的对象，存在外部指针引用或从根对象可达。

工作过程：

- 根对象被标记成灰色。收集器只会从灰色对象集合中扫描，灰色集合不存在对象时，标记阶段结束。
- 从灰色集合中选择一个灰色对象，标记成黑色。
- 黑色对象指向的所有对象都标记成灰色。（这样该对象不会被回收）
- 重复以上两个步骤不存在灰色。
    
    ![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled%205.png)
    

当三色的标记清除的标记阶段结束之后，应用程序的堆中就不存在任何的灰色对象，我们只能看到黑色的存活对象以及白色的垃圾对象，垃圾收集器可以回收这些白色的垃圾。

但是在运行过程中，会发生指针修改的问题，因此三色标记算法还是不可以并发或增量地运行。

 因此，便有了屏障技术。

### 屏障技术

> 内存屏障技术是一种屏障指令，它可以让 CPU 或者编译器在执行内存相关操作时遵循特定的约束，目前多数的现代处理器都会乱序执行指令以最大化性能，但是该技术能够保证内存操作的顺序性，在内存屏障前执行的操作一定会先于内存屏障后执行的操作。
> 

要想并发或增量地运行三色标记算法，要达成两种三色不变性其中的一种：

- 强三色不变性：黑色对象不会指向白色对象。
- 弱三色不变性：黑色对象指向的白色对象必须包含一条从灰色对象经由多个白色对象的可达路径。（垃圾收集器无法从某个灰色对象出发，经过几个连续的白色对象访问白色的 C 和 D 两个对象）

![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled%206.png)

编程语言往往都会采用写屏障保证三色不变性。Go 语言中使用的两种写屏障技术，分别是 Dijkstra 提出的插入写屏障和 Yuasa 提出的删除写屏障。

### 插入写屏障

写屏障相当于一个钩子方法。

在执行 `*slot = ptr` 时，若 `ptr` 指向的对象为白色， 则标记其为灰色。

例如：

![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled%207.png)

1. 垃圾收集器将根对象指向 A 对象标记成黑色并将 A 对象指向的对象 B 标记成灰色；
2. 用户程序修改 A 对象的指针，将原本指向 B 对象的指针指向 C 对象，这时触发写屏障将 C 对象标记成灰色；
3. 垃圾收集器依次遍历程序中的其他灰色对象，将它们分别标记成黑色；

这是一种比较保守的屏障技术，保证了强三色不变性（所有存活可能地对象都标记成灰色）。

缺点：

- 可以回收的 B 对象，在此次循环中不会被认为可被回收。
- 因为**栈上的对象**在垃圾收集中也会被认为是**根对象**，因此要为其添加写屏障，或是每次标记后针对栈上对象进行扫描。前者添加大量写入指针地额外开销，后者扫描栈对象需要停止程序。

### 删除写屏障

删除写屏障会保证开启写屏障时，堆上所有对象的可达，因此也被叫做快照垃圾收集。

在老对象的引用被删除时，将白色的老对象涂成灰色，这样就保证了弱三色不变性。

![Untitled](Golang%E7%AC%94%E8%AE%B0%20201e1/Untitled%208.png)

1. 垃圾收集器将根对象指向 A 对象标记成黑色并将 A 对象指向的对象 B 标记成灰色；
2. 用户程序将 A 对象原本指向 B 的指针指向 C，触发删除写屏障，但是因为 B 对象已经是灰色的，所以不做改变；
3. **用户程序将 B 对象原本指向 C 的指针删除，触发删除写屏障，白色的 C 对象被涂成灰色**；
4. 垃圾收集器依次遍历程序中的其他灰色对象，将它们分别标记成黑色；

Yuasa 删除写屏障通过对 C 对象的着色，保证了 C 对象和下游的 D 对象能够在这一次垃圾收集的循环中存活，避免发生悬挂指针以保证用户程序的正确性。
