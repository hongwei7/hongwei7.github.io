# 数据链路层复习


# 数据链路层（逻辑链路层）

数据链路层包含逻辑链路层和 MAC 子层.

## 数据链路层的流量控制

- 差错检测和控制
- 流量控制
    - 基于速率
    - 基于反馈（由接收方反馈信息给发送方）

## 数据结构（帧）

帧=帧头+载荷+帧尾

![/images/Network-1/Untitled.png](/images/Network-1/Untitled.png)

![/images/Network-1/Untitled%201.png](/images/Network-1/Untitled%201.png)

### 帧头

- 物理地址信息

### 帧尾

- 校验和

## 成帧的方法

- 字符计数法（头部添加帧长）
- 字节填充的标志字节法（每一帧采用特殊的字符作为帧界 flag byte）

    使用转义符解决标志字节法的问题，实际用于PPP 协议中

    ![/images/Network-1/Untitled%202.png](/images/Network-1/Untitled%202.png)

- 比特填充的比特标记法解决这个问题。（5 个 1 后面加一个 0）

    ![/images/Network-1/Untitled%203.png](/images/Network-1/Untitled%203.png)

- 或者使用冗余的信号作为帧界

    ![/images/Network-1/Untitled%204.png](/images/Network-1/Untitled%204.png)

## 差错处理（纠错、检错）

纠错码主要用于无线网络（冗余开销大）

检错码可以发现错误，使用在局域网中，使用重传恢复。

### 海明距离

两个码字之间的海明距离为不同位的个数。

![/images/Network-1/Untitled%205.png](/images/Network-1/Untitled%205.png)

可以使用异或计算，再统计 1 的个数。

> 全部码字的海明距离定义为任意两个码字之间海明距离的最小值。

### 海明距离的意义

如果海明距离为 d，则一个码字需要发生 $d$ 个 1 位错误才能变成另外一个码字。

### 海明距离与检错

> 海明距离为 $d+1$ 的编码可以检测出 d 位差错。

一个码字至少要变 $d+1$ 次才能变为另外一个码字。

![/images/Network-1/Untitled%206.png](/images/Network-1/Untitled%206.png)

海明距离代表了需要检错的数据位的成本开销。

![/images/Network-1/Untitled%207.png](/images/Network-1/Untitled%207.png)

### 海明距离与纠错

> 海明距离为 $2d+1$ 的编码，可以纠错 $d$ 位差错。

![/images/Network-1/Untitled%208.png](/images/Network-1/Untitled%208.png)

它可以选择海明距离最低的码字进行纠正。

![/images/Network-1/Untitled%209.png](/images/Network-1/Untitled%209.png)

海明距离越大，纠错能力越强。

### 纠一位错的海明码

![/images/Network-1/Untitled%2010.png](/images/Network-1/Untitled%2010.png)

### 检一位错的海明码

- 奇偶校验码（添加一位，保证 1 位奇数或者偶数个）
- 互联网校验和（常见的 16 位补码互联网校验和）
- 循环冗余校验码CRC（使用异或运算，检查多项式是否能被通用的生成多项式CRC32整除）

## 基本协议

- 无限制的单工协议（乌托邦协议）
- 半双工协议（有反馈机制，等待上一个帧的收到回应后，再发送下一个帧）
- 肯定确认重传（确认正确帧，超时重发，添加序列号）

## 滑动窗口

### 协议 4：滑动窗口协议

- 窗口大小为 1
- 发送窗口：已经发送未被确认的序号
- 接收窗口：期望接收的序列号

seq 使用一个位来表示，0，1 交替出现。

![/images/Network-1/Untitled%2011.png](/images/Network-1/Untitled%2011.png)

![/images/Network-1/Untitled%2012.png](/images/Network-1/Untitled%2012.png)

存在问题：定时器设置过短的时候，会产生很多重复帧。

### 协议 5：回退 n 帧

当某一个帧出错时，重传从那个帧开始之后的所有帧。

采用了累计确认。

- 定义 seq 和滑动窗口
- 发送方连续发送，直到发送窗口满
- 接收窗口为 1，对出错帧不确认（引发超时）
- 发送方超时重传，从未被确认的帧开始

存在问题：maxSeq 的混搅

![/images/Network-1/Untitled%2013.png](/images/Network-1/Untitled%2013.png)

解决方法：发送窗口小于 maxSeq。

### 协议 6：选择重传

当某一个帧出错时，重传那个帧。

![/images/Network-1/Untitled%2014.png](/images/Network-1/Untitled%2014.png)

使用否定确认 NAK（告诉发送方某帧出错，而不是等超时）

![/images/Network-1/Untitled%2015.png](/images/Network-1/Untitled%2015.png)

接收窗口：$W=(maxSeq+1)/2$。保证新老窗口不重复即可。

![/images/Network-1/Untitled%2016.png](/images/Network-1/Untitled%2016.png)

# 数据链路层（MAC 子层）

## 介质的多路访问控制

- 静态分配：预先分配给各个用户。
- 动态分配：信号是开放的。

### 多路访问协议

- 随机访问协议
    - ALOHA
    - CSMA
    - CSMA/CD（以太网）
- 受控访问协议

### ALOHA

- 纯 ALOHA（冲突后等待随机时间）
- 分隙 ALOHA（把时间分成时隙，占用后发送）

![/images/Network-1/Untitled%2017.png](/images/Network-1/Untitled%2017.png)

### CSMA（改进的 ALOHA）

- 非持续式（介质如果忙，等待一个随机分布时间后再询问）
- 持续式（如果介质忙，持续侦听）
    - 1-持续（如果空闲时，直接发送）
    - p-持续（如果空闲时，以 $P$ 的概率发送）

### 冲突窗口 RTT

定义为两个最远工作站的传输时间（两个帧时）。

![/images/Network-1/Untitled%2018.png](/images/Network-1/Untitled%2018.png)

### CSMA/CD（带冲突检测的 CSMA）

实际上是一个 1 持续的 CSMA。

> 先听后发，边发边听。

- 若介质忙，持续监听，空闲立即发送。
- 发送的同时监听自己的信号，若不一致，停止帧的发送，并发出一个拥塞信号。

### 冲突检测

![/images/Network-1/Untitled%2019.png](/images/Network-1/Untitled%2019.png)

# 以太网

以太网位于数据链路层和物理层。

![/images/Network-1/Untitled%2020.png](/images/Network-1/Untitled%2020.png)

## IEEE

![/images/Network-1/Untitled%2021.png](/images/Network-1/Untitled%2021.png)

## 以太网采用曼彻斯特编码的CSMA/CD 技术

## IEEE802.3 以太网帧（区别于 DIX 以太网帧）

![/images/Network-1/Untitled%2022.png](/images/Network-1/Untitled%2022.png)

![/images/Network-1/Untitled%2023.png](/images/Network-1/Untitled%2023.png)

- 前导码+11 一共 64bit（8 bytes）
- 目的地址和源地址都是 MAC 地址。
- 以太网：类型（上层使用的协议）
- IEEE：数据长度（64 字节~1518 字节，只不包括前导码）

![/images/Network-1/Untitled%2024.png](/images/Network-1/Untitled%2024.png)

- 数据（46~1500 字节）
- 校验和（CRC 冗余校验，除了前导码以外的校验。4 个字节）

### 为什么帧长度要大于 64bytes

CSMA/CD 要求最短帧的发送时间大于等于争用时隙。

## 二层交换的原理

网桥（交换机）负责二层交换，不会检查网络层，只做转发。

![/images/Network-1/Untitled%2025.png](/images/Network-1/Untitled%2025.png)

- 通过透明网桥把多个 LAN 连接起来。
- 透明网桥接收所有的帧，并决定丢弃还是转发。
- 决策是通过网桥内部的地址表的目的 MAC 地址来做出的。
- 当找不到地址时，**广播这个帧**。
- 收到一个帧，会学习帧的源地址。

![/images/Network-1/Untitled%2026.png](/images/Network-1/Untitled%2026.png)

网桥分割了区域，又提高了性能。

### 生成树算法 STP

生成无环的树，防止网络风暴、多帧传送、MAC 地址库不稳定。

### 两层交换设备

- 以太网卡（具有发送和监听的功能）
- 网桥（连接不同的 LAN）
- 交换机（多端口的网桥）

工作原理：

- flooding：泛洪
- forwarding: 转发
- filtering：过滤
- learning：学习

![/images/Network-1/Untitled%2027.png](/images/Network-1/Untitled%2027.png)

- 存储转发（来得及校验）
- 直接转发（来不及校验）
- 无分片交换（读够 64 个字节后再转发）

![/images/Network-1/Untitled%2028.png](/images/Network-1/Untitled%2028.png)
