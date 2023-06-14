# GFS文件系统


随着计算机在数量上的增加，计算机同样开始分散。尽管商业公司过去愿意购买越来越大的大型机，现在的典型情况是，甚至很小的应用程序都同时在多台机器上运行。思考这样做的利弊权衡，即是分布式系统的研究所在，也是越来越重要的一项技能。

# 概述

## 介绍

需要分布式的原因：

- 并行性
- 容错性
- 物理性
- 安全性

基本的挑战：

- 并行性带来的复杂
- 部分故障
- 最终性能

### 实现工具

- RPC
- 多线程
- 多并发控制（锁）

## 性能-可拓展性加速

最好的结果是，线性地增长设备时，性能也能线性地提升（甚至指数性）。

![/images/GFS/Untitled.png](/images/GFS/Untitled.png)

## 容错

计算机会经常故障或者挂掉（集群）。这要求系统拥有可用性和恢复性。

- 可用性: 服务在以上这些挑战下，还是可用的。
- 可恢复性：出现问题时，服务是可恢复的。

解决工具：

- 非易失性存储（硬盘）。通常很慢。
- 复制副本。

## 一致性

存储→key-value表。

put 和 get 会在多副本中产生一些列一致性的问题。

## MapReduce

假设有一些输入块，每个输入块经过一个 Map 函数，得到一些 key。Reduce 部分是指把每个输入块得到的 key，集合起来，相同的 key 传递给一个 Reduce 函数，Reduce 函数负责把这些数据归合起来。最后分别计算数据。

![/images/GFS/Untitled 1.png](/images/GFS/Untitled%201.png)

# Go

## 使用多线程

### 好处：

- IO 资源共享
- 并行性
- 周期性检查
- 事件驱动友好

### 挑战：

- 共享变量带来的 race 问题

## coordination协调手段

channels 频道

sync.Cond 条件变量

waitGroup 等待队列

Deadlock 死锁

# GFS

[Google File System-GFS 论文阅读](https://spongecaptain.cool/post/paper/googlefilesystem/)

## 大数据存储的难题

- 性能表现 →分片分机
- 分片分机 →容错
- 容错度→副本复制
- 副本复制→消除一致性问题
- 一致性→低性能

## 一个副本的情况

考虑存储一个键值表。

一个副本下，也会出现一些简单的一致性的问题

![/images/GFS/Untitled 2.png](/images/GFS/Untitled%202.png)

同时容错率极低，单主机崩溃后 server 直接 down。

## 两个副本的情况

![/images/GFS/Untitled 3.png](/images/GFS/Untitled%203.png)

两个副本下， client 直接向两台服务器同时发送请求。若不能保证client 请求的顺序相同，一致性就会被破坏。

## GFS 的解决方案

[The Google File System中文版.pdf](/images/GFS/The_Google_File_System.pdf)

特点：

- Big，Fast，Global（共享）
- 分割文件：从大文件到小文件。
- 自动化恢复
- 单一的数据中心（Google 内部使用的服务）
- 为大范围的非随机访问优化

设计：

![/images/GFS/Untitled 4.png](/images/GFS/Untitled%204.png)

客户端发送文件名和 chunk index，master 服务器做 map 工作，把 IO 工作分到不同 chunkserver，不同的 chunkserver 带来并行性提高性能。

master 需要保存的信息：

- 文件名→array of chunk handles  （nv）
- chunk handles
    - version     chunk 的版本号 （nv）
    - list of chunk servers   （memory）
    - primary  （memory）
    - lease time      过期时间  （memory）
- log + checkpoint （stable storage 先写入 log 再执行写入操作）

![/images/GFS/Untitled 5.png](/images/GFS/Untitled%205.png)

### 读操作

- client 发送 filename 和 offset 到 master
- master 找到 list of chunkservers
- client 会对某些 chunk server 发送读请求，且会把 chunk server缓存起来
- chunk server 读取文件，返回数据

### 写操作

当 master 没有 primary 时：

- 找到最新副本的 chunk server
- 选择一个作为 primary
- 增加版本号，写入
- 告诉 primary 和 secondary，需要增加版本号
- master 把新的 version 写入硬盘
- primary 告诉所有的 secondary，需要追加写入文件
- 如果所有的 secondary 都回应 yes，那么向 client 回应 success，否则回应 failed。这里如果某个 secondary 写入失败，其他的 secondary 仍然会写入，这里会产生不一致性的问题。但是 GFS 容忍这种不一致性（冗余）。同时 client 会重试请求。

![/images/GFS/Untitled 6.png](/images/GFS/Untitled%206.png)

当 master 有 primary 时，不会更新版本号。

![/images/GFS/Untitled 7.png](/images/GFS/Untitled%207.png)

### split brain 脑裂问题

当网络错误使得 master 无法和 primary 通信，master 会重新指定一个 primary，此时会产生两个 primary。此时 client 直接向两个 primary 发送信息时，就会产生冲突。

解决方法：当 master 需要重新选择 primary 时，master 会等待上一个 primary 的 lease（租约）过期，再去分配新的 primary。

### 缺点

GFS 只有一个 master，而单个 master 的内存资源总是有限的。

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/gfs/  

