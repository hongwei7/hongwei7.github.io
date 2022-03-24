# proc 初探


proc是一个系统文件目录,存放着记录当前运行状态的文件.<!-- more-->

## proc 初探（Deepin下）

- acpi 目录 高级配置和电源接口 支持OS设置和控制各个硬件部件
  - wakeup 文件
- asound 文件夹 声音设备相关
- buddyinfo 内存管理算法的状态
- bus 文件夹 总线管理
  - input 文件夹
    - devices 输入设备列表
    - handlers 解决方案
  - pci 文件夹 
    - devices pci连接设备
- cgroups 将进程进行分组，对一组进程进行统一资源监控和限制
- cmdline 启动当前进程的完整命令 or  启动时传递给kernel的参数信息
- consoles 可以查看系统运行的所有终端
- cpuinfo 目前CPU信息
- crypto 内核使用的所有已安装的加密密码及细节
- devices 已经加载的设备
- diskstats 硬盘状态
- dma  已注册使用的ISA DMA频道列表（不明白）
- driver 驱动
  - rtc RTC驱动
- execdomains 可能是内核执行空间
- fb 帧缓冲设备列表文件，包含帧缓冲设备的设备号和相关驱动信息
- filesystems 当前被内核支持的文件系统类型列表
- fs 挂载的文件系统类型及挂载的分区
- interrupts 查看中断情况
- iomem io设备内存缓冲区
- ioports io端口资源分布
- irq 各个中断具体情况
- kallsyms linux内核符号表 内核中使用到的函数和变量地址
- kcore 内存的映射 类比为内存中的alias
- keys 密钥
- key-users 用户密钥
- kmsg  似乎是内核的环形缓冲区
- kpagecgroup 
- kpagecount 包含一个64位值，该值表示每个page被映射的次数，通过PFN索引。
- kpageflags 这个文件包含每一个page的64位的标记集，通过PFN索引
- loadavg 平均负载也就是可运行的进程的平均数
- locks 保存当前由内核锁定的文件的相关信息
- mdstat 内核是否已经加载MD驱动 (利用条带化(stripping)技术将数据块均匀分布到多个磁盘上来提高虚拟设备的读写性能)
- meminfo 内存信息
- misc 查看系统中装载的所有misc类设备
- modules 列出了所有load进入内核的模块列表
- mounts 文件挂载信息
- mtrr 将内存类型与系统内存中的物理地址范围关联起来
- net 网络信息与配置
- pagetypeinfo buddyinfo文件的细分
- partitions 查看分区信息
- sched_debug 打印出所有cpu的信息
- schedstat  CPU调度时间统计
- scsi 查看目前的磁盘信息
- slabinfo  gives information about memory usage on the slab level
- softirqs 查看软中断
- stat 记录的是系统进程整体的统计信息
- swaps 交换区信息
- sys 包括所有的内核参数信息
- sysrq-trigger （Magic System Request Key） 可以调用系统一些行为
- sysvipc 包括进程间通信的信号量（sem）、共享内存（shm）、消息队列（msg）信息
- thread-self 这是一个link，当访问次链接时，就会访问进程的`/proc/self/task/tid`目录
- timer_list 打印per_cpu的hrtimer_bases信息以及基于此的timer列表
- tty 查询当前系统tty的情况
- uptime 启动时间
- version 系统版本
- vfs_changes 内核模块创建的接口文件，用来给上层用户态应用程序访问
- vmallocinfo 由于系统的连续物理内存有限，这使得非连续物理内存的使用在linux内核中出现，查看非连续内存使用信息。
- vmstat  取得虚拟内存统计信息
- zoneinfo Zone的管理调度（虚拟内存相关）

