# Linux 系统编程


# 进程

## MMU

![/images/LinuxSystem/Untitled.png](/images/LinuxSystem/Untitled.png)

## 进程控制块PCB

进程控制块PCB的结构体形式：task_struct。

![/images/LinuxSystem/Untitled%201.png](/images/LinuxSystem/Untitled%201.png)

虚拟地址空间的信息描述为虚拟地址到物理地址的映射。MMU来维护。

umask掩码：保护文件权限。

ulimit -a 能查看linux资源上限

## 环境变量

以用户为单位设置环境变量。

- PATH
- SHELL 当前SHELL
- TERM 当前终端类型
- HOME
- LANG

类似于命令行参数。

![/images/LinuxSystem/Untitled%202.png](/images/LinuxSystem/Untitled%202.png)

```c
extern char** eniron
```

### getenv 获取环境变量

### setenv设置环境变量

### unsetenv取消环境变量

## 进程控制

### fork()

![/images/LinuxSystem/Untitled%203.png](/images/LinuxSystem/Untitled%203.png)

创建的是子进程。通过返回的pid来判断是否在子进程中。返回0时为子进程，父进程中返回子进程的pid。

### getpid 获取 pid

### getppid 获取父亲 pid

### 创建N个进程

```c
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>

int forkN(int n){
        printf("I am the father %d\n", getpid());
        for(int i = 0; i < n; i++){
                int pid = fork();
                if(pid == -1){
                        perror("fork error!");
                        exit(1);
                }
                else if(pid == 0){
                        printf("I am the %d child %u, my father is %u\n", i + 1, getpid(), getppid());
                        break;
                }
        }
        return 0;
}
```

## 进程共享

刚fork完时，父子进程中

全局变量、data、.text、栈、堆、环境变量、用户ID、目录、信号处理方式都是相同的。（0-3G地址相同）

不同在于进程id、fork返回值、父进程id、进程运行时间、定时器（一个进程有一个）、未决信号集。（PCB不一样）

fork完之后，父子进程之间遵循**读时共享，写时复制**的原则，节省内存开销。

注意父子进程之间不能通过全局变量共享数据。（写时复制）

父子进程间共享的点：

- 文件描述符（多个进程对同一个文件操作）
- mmap建立的映射区（进程间通信）

## gdb调试多进程程序

```c
set follow-fork-mod child
set follow-fork-mod parent
```

## exec类函数

让程序执行一个进程（原进程的用户空间和代码完全被新程序替换）。

注意此时并不是创建一个新进程，而是原进程的用户空间和代码全部被替换。

![/images/LinuxSystem/Untitled%204.png](/images/LinuxSystem/Untitled%204.png)

重点execl和execlp。

- execlp：（list path）加载一个进程，借助环境变量
  
  ```c
  execlp("ls", "ls", "-l", "-a", NULL);
  //第二个参数相当于argv[0] 一般可能不使用
  ```

- execl：通过路径+程序名来加载。
  
  ```c
  execl("/bin/ls", "ls", "-l", "-a", NULL);
  ```

- execle:借助环境变量表

- execv：不含argv[0]，但是参数先要包装成字符数组。

## 保存 ps 结果到 out

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <fcntl.h>
#include <sys/stat.h>

int ps()
{
        int fd = open("out", O_WRONLY | O_CREAT | O_TRUNC, 0644);
        if(fd == -1){
                perror("open file:");
                exit(1);
        }
        dup2(fd, STDOUT_FILENO); //把out文件的描述符复制到std::out
        int res = execlp("ps", "ps", "aux", NULL);
                if(res == -1)perror("exec error:");
                exit(1);
        close(fd);
        return 0;
}
```

exec函数只在出错的时候返回-1。

## 回收子进程

### 孤儿进程

父进程先于子进程结束，子进程则为孤儿进程，子进程父进程变为init进程（孤儿院），父进程的pid变为1。

### 僵尸进程

进程终止了，但是父进程没有回收。子进程PCB残留在内核中，成为僵尸进程。

![/images/LinuxSystem/Untitled%205.png](/images/LinuxSystem/Untitled%205.png)

这里的父进程一直不停止，但是子进程的PCB一直不回收。

![/images/LinuxSystem/Untitled%206.png](/images/LinuxSystem/Untitled%206.png)

### wait waitpid 解决僵尸函数

wait有三个功能

- 阻塞等待子进程退出。
- 回收子进程残留资源。
- 获取子进程结束状态。

一次wait只能回收一个进程，回收先结束的进程。

返回为pid时，回收成功，-1时，没有子进程。

![/images/LinuxSystem/Untitled%207.png](/images/LinuxSystem/Untitled%207.png)

wstatus为传出参数：

![/images/LinuxSystem/Untitled%208.png](/images/LinuxSystem/Untitled%208.png)

kill -l可以查看信号结果。

waitpid函数更灵活一些，指定pid回收，且可以不阻塞。

![/images/LinuxSystem/Untitled%209.png](/images/LinuxSystem/Untitled%209.png)

pid为-1时，回收任意存在的子进程。

![/images/LinuxSystem/Untitled%2010.png](/images/LinuxSystem/Untitled%2010.png)

0 阻塞 WNOHANG 非阻塞，需要轮询。

![/images/LinuxSystem/Untitled%2011.png](/images/LinuxSystem/Untitled%2011.png)

总结：

![/images/LinuxSystem/Untitled%2012.png](/images/LinuxSystem/Untitled%2012.png)

# IPC进程间通信

常用方法：

- 管道（最简单）
- 信号（开销最小）
- 共享映射区（无血缘关系）（mmap）
- 本地套接字（最稳定）

## 管道 pipe

fifo（有名管道，非血缘关系间），pipe（匿名管道）

最简单的想法：通过文件实现进程间通信。抽象成为管道。

管道实际上为内核缓冲区，为伪文件。

一个管道有两个描述符，一个为读一个为写，默认为8k的缓冲区。

内部采用的是循环队列。

![/images/LinuxSystem/Untitled%2013.png](/images/LinuxSystem/Untitled%2013.png)

![/images/LinuxSystem/Untitled%2014.png](/images/LinuxSystem/Untitled%2014.png)

返回两个文件描述符，读和写。

### 简单的管道示例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int pipeTest(){
        int fd[2]; //两个文件描述符
        int ret = pipe(fd);
        if(ret == -1){
                perror("pipe error:");
                exit(1);
        }
        pid_t pid = fork();
        if(pid == -1){
                perror("fork");
                exit(1);
        }else if(pid == 0){ //child read
                close(fd[1]);
                char buf[1024];
                ret = read(fd[0], buf, sizeof(buf));
                if(ret == 0){
                        printf("read done\n");
                }
                write(STDOUT_FILENO, buf, ret);
        }else{ //father write
                close(fd[0]);
                sleep(1);
                char * str = "hello pipe\n";
                ret = write(fd[1], str, strlen(str));
        }
}
```

## mmap 共享内存

![/images/LinuxSystem/Untitled%2015.png](/images/LinuxSystem/Untitled%2015.png)

借助共享内存存放磁盘文件。这样可以通过指针访问磁盘文件。

- addr：映射区的首地址 ，内核自动指定（直接传NULL)
- length：映射区大小（文件大小）
- prot：映射区的权限
  - PROT_READ
  - PROT_WRITE
  - PROT_READ | PROTWRITE
- flags：标志位参数

返回创建映射区的首地址。失败时返回MAP_FIALED。                                                                                                                                                                                                                                                                                                                                                      

![/images/LinuxSystem/Untitled%2016.png](/images/LinuxSystem/Untitled%2016.png)

- fd：文件描述符
- offset：映射文件的偏移（截取文件一部分）

![/images/LinuxSystem/Untitled%2017.png](/images/LinuxSystem/Untitled%2017.png)

```c
#include<stdio.h>
#include<sys/mman.h>
#include<stdlib.h>
#include<fcntl.h>
#include<string.h>
#include<unistd.h>

int mmapTest(){
    char* p = NULL;
    int fd = open("text", O_RDWR | O_CREAT, 0644);
    if(fd < 0){
        perror("open fail:");
        exit(1);
    }
    int len = 10; //size
    if( ftruncate(fd, len) == -1){
        perror("ftruncate"); //拓展文件
        exit(1);
    }
    p = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
       if(p == MAP_FAILED){
           perror("MAP FAILED");
           exit(1);
       }       
       strcpy(p, "abc"); //write to mmap
       int res = close(fd);
    if(res == -1){
               perror("close fail");
        exit(1);
       }
       res = munmap(p, len);
       if(res == -1){
               perror("unmap fail");
        exit(1);
       }
       return 0;
}
```

注意事项：

- 不能创建一个大小为0的映射区（malloc可以）
- 注意munmap释放的必须是以前的首指针。
- **映射区的权限应小于等于实际文件。**
- **映射区创建的时候需要读取文件权限。**
- offset需要4k的整数倍（一个页的大小）。  ****
- 文件描述符可以先关闭。

![/images/LinuxSystem/Untitled%2018.png](/images/LinuxSystem/Untitled%2018.png)

### mmap 父子之间通信

参数应该为MAP_SHARED。

![/images/LinuxSystem/Untitled%2019.png](/images/LinuxSystem/Untitled%2019.png)

**无血缘也相似**，只要对同一个文件调用mmap就行。

读写两边都munmap()。

把映射区当数组看待。

### mmap匿名映射

第三个参数位或`MAP_ANONYMOUS(MAP_ANON)`。此时文件描述符填-1。注意仅在Linux下有，类Unix下无。

类Unix下，应该打开/dev/zero文件来代替。

![/images/LinuxSystem/Untitled%2020.png](/images/LinuxSystem/Untitled%2020.png)

fifo和匿名映射的区别：fifo不可以重复读。

### strace命令

```c
strace binfile
```

可以查看程序调用的资源。

## 阶段性练习

[Linux系统编程-实践练习](https://www.notion.so/Linux-13d606ad446947b8a677217dc9a41b3e)

### 多文件拷贝

实现方法：文件分成多份，让多个进程去复制各个部分。

### 简单 shell

![/images/LinuxSystem/Untitled%2021.png](/images/LinuxSystem/Untitled%2021.png)

![/images/LinuxSystem/Untitled%2022.png](/images/LinuxSystem/Untitled%2022.png)

### 本地聊天室功能

![/images/LinuxSystem/Untitled%2023.png](/images/LinuxSystem/Untitled%2023.png)

![/images/LinuxSystem/Untitled%2024.png](/images/LinuxSystem/Untitled%2024.png)

使用fcntl()函数把输入改造为非阻塞的。

[Linux fcntl函数设置阻塞与非阻塞](https://www.cnblogs.com/zhangxuan/p/6306326.html)

# 信号

**区别于信号量。**

简单，不能携带大量信息，满足某个条件才会发送。

发送通过**内核**。程序收到信号后会停止下来处理信号（类似于时钟中断）。但是是通过软件来实现，也称为软中断。

软中断的延时非常大，但是对于用户来说不易察觉。CPU可以察觉。

四要素：

- 编号
- 名称
- 事件
- 默认处理动作

## 信号的传输过程

![/images/LinuxSystem/Untitled%2025.png](/images/LinuxSystem/Untitled%2025.png)

未决信号集代表了未处决的信号。

![/images/LinuxSystem/Untitled%2026.png](/images/LinuxSystem/Untitled%2026.png)

阻塞信号集（信号屏蔽字）代表了处于阻塞态的信号。

两者都处于PCB中。都是set。

![/images/LinuxSystem/Untitled%2027.png](/images/LinuxSystem/Untitled%2027.png)

当信号屏蔽字为1时，该位的信号将会保留下来，不能被处理。

**信号的处理动作：**

- 默认处理动作
- 忽略（丢弃）
- 捕捉（捕捉后调用用户处理函数（发送方））

![/images/LinuxSystem/Untitled%2028.png](/images/LinuxSystem/Untitled%2028.png)

![/images/LinuxSystem/Untitled%2029.png](/images/LinuxSystem/Untitled%2029.png)

信号编号：kill -l    man 7 signal

![/images/LinuxSystem/Untitled%2030.png](/images/LinuxSystem/Untitled%2030.png)

9和19号不允许被捕捉或阻塞。

## 信号的产生

产生方法：

- 按键
  - Ctrl + c ： 2）SIGINT（终止） Interrupt
  - Ctrl + z： 20）SIGTSTOP（终端程序暂停） Stop
  - Ctrl + \： 3）SIGQUIT（退出） Quit
- 系统调用
- 软件
- 硬件异常
  - /0  (8)SIGFPE（浮点数例外）
  - 非法访问内存 (11)SIGSEGV(段错误）
  - 总线错误 （7）SIGBUS
- 命令调用
  - killl  `kill -19 pid` 发送19信号
  - alarm
  - setitimer，

### kill 函数

![/images/LinuxSystem/Untitled%2031.png](/images/LinuxSystem/Untitled%2031.png)

### raise 与 abort 函数

![/images/LinuxSystem/Untitled%2032.png](/images/LinuxSystem/Untitled%2032.png)

### alarm 函数

每个进程只有一个定时器。

指定需要的second后，内核会给当前进程发送（14）SIGALRM，默认行为为终止进程。

**返回值为上一次alarm的剩余时间**。

![/images/LinuxSystem/Untitled%2033.png](/images/LinuxSystem/Untitled%2033.png)

另外可以time ./app查看运行时间。

### setitimer

可以取代alarm，精确到微秒，也可以周期性定时。

![/images/LinuxSystem/Untitled%2034.png](/images/LinuxSystem/Untitled%2034.png)

which函数代表如何定时（自然定时还是用户空间还是用户加内核）。

![/images/LinuxSystem/Untitled%2035.png](/images/LinuxSystem/Untitled%2035.png)

结构体看man page。

## 信号集操作

### 信号集设定

sigset_t 使用unsigned long构成的set（8字节）。

![/images/LinuxSystem/Untitled%2036.png](/images/LinuxSystem/Untitled%2036.png)

### sigprocmask 函数

把自定义的信号set导入到阻塞集。

![/images/LinuxSystem/Untitled%2037.png](/images/LinuxSystem/Untitled%2037.png)

![/images/LinuxSystem/Untitled%2038.png](/images/LinuxSystem/Untitled%2038.png)

### sigpending 函数

读取当前进程的未决信号集。

![/images/LinuxSystem/Untitled%2039.png](/images/LinuxSystem/Untitled%2039.png)

### 示例：打印未决信号集

```c
#include <stdio.h>
#include <unistd.h>
#include<signal.h>
#include <stdlib.h>

void printsig(sigset_t *set){
        for(int i = 1; i < 32; i++){
                printf("%d", sigismember(set, i));
        }printf("\n");
}

int main()
{
        sigset_t myset, oldset, pend;

        sigemptyset(&myset);
        sigaddset(&myset, SIGQUIT);
        int res = sigprocmask(SIG_BLOCK, &myset, &oldset);
        if(res == -1){
                perror("sig error");
                exit(1);
        }

        sigpending(&pend);
        printsig(&pend);
        raise(SIGQUIT);
        sigpending(&pend);
        while(1){
                sleep(1);
                printsig(&pend);
        }
        return 0;
}
```

## 信号的捕捉

### signal 函数

使用signal函数可以捕捉到信号，调用自定义的处理函数。

![/images/LinuxSystem/Untitled%2040.png](/images/LinuxSystem/Untitled%2040.png)

### sigaction 函数

![/images/LinuxSystem/Untitled%2041.png](/images/LinuxSystem/Untitled%2041.png)

  其中sigaction结构为

![/images/LinuxSystem/Untitled%2042.png](/images/LinuxSystem/Untitled%2042.png)

第二个参数不管。

sigset_t 定义handler运行期间的信号屏蔽字。

sa_flags 0时为默认。

![/images/LinuxSystem/Untitled%2043.png](/images/LinuxSystem/Untitled%2043.png)

![/images/LinuxSystem/Untitled%2044.png](/images/LinuxSystem/Untitled%2044.png)

### 捕捉原理

![/images/LinuxSystem/Untitled%2045.png](/images/LinuxSystem/Untitled%2045.png)

# 竞态条件

## pause函数：

可以造成进程主动挂起，等待信号唤醒。注意先要对信号进行捕捉。

![/images/LinuxSystem/Untitled%2046.png](/images/LinuxSystem/Untitled%2046.png)

注意这里返回的是-1。

**使用sigaction和pause实现sleep：**

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

void catchSignal(int signo){
    ;
}

int mySleep(unsigned int seconds){
    struct sigaction act, oldact;
    act.sa_handler = catchSignal;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;

    int ret = sigaction(SIGALRM, &act, &oldact);
    if(ret == -1){
        perror("sigaction");
        exit(1);
    }
    alarm(seconds);
    ret = pause();
    if(ret == -1 && errno == EINTR){
        printf("signal caught!\n");
    }

    ret = alarm(0);
    sigaction(SIGALRM, &oldact, NULL);
    return ret;
}

int main(){
    while(1)mySleep(1);
}
```

## 时序竞态

### 一个会出问题的例子

![/images/LinuxSystem/Untitled%2047.png](/images/LinuxSystem/Untitled%2047.png)

若CPU失去时间太长，pause将永远不会被唤醒。

### sigsuspend 函数

![/images/LinuxSystem/Untitled%2048.png](/images/LinuxSystem/Untitled%2048.png)

相当于带有信号屏蔽字的pause函数，可以解决以上问题。（原子操作）

![/images/LinuxSystem/Untitled%2049.png](/images/LinuxSystem/Untitled%2049.png)

流程：

- 注册SIGALRM处理函数
- 阻塞SIGALRM
- alarm→sigsuspend【解除阻塞SIGALRM→pause→继续阻塞SIGALRM】
- 解除SIGALRM处理函数
- 解除阻塞SIGALRM。

这样可以解决pause的时序问题。

### 全局变量异步IO问题

父子进程之间交替数数程序。运行一段时间后两者都永久等待。

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
#include <stdlib.h>

int n = 0, flag = 0;  //定义两个全局变量（注意了）

void sys_err(char *str)
{
    perror(str);
    exit(1);
}

void do_sig_child(int num)  //子进程的用户处理函数
{
    printf("I am child  %d\t%d\n", getpid(), n);
    n += 2;
    flag = 1;  //对全局变量的修改
}

void do_sig_parent(int num)   //父进程的用户处理函数
{
    printf("I am parent %d\t%d\n", getpid(), n);
    n += 2;
    flag = 1;  //对全局变量的修改
}

int main(void)
{
    pid_t pid;
    struct sigaction act;

    if ((pid = fork()) < 0)
        sys_err("fork");

    else if (pid > 0) {
        n = 1;          //父进程从1开始数
        sleep(1);       //父进程睡眠1s确保在父进程向子进程发信号之前，子进程完成了对信号的注册
        act.sa_handler = do_sig_parent;
        sigemptyset(&act.sa_mask);
        act.sa_flags = 0;
        sigaction(SIGUSR2, &act, NULL);             //注册自己的信号捕捉函数，父进程使用SIGUSR2信号

        do_sig_parent(0);   //父进程先进行数数，从1开始

        while(1) {
            /* wait for signal */;
           if (flag == 1) {                         //父进程数数完成
                kill(pid, SIGUSR1);
                flag = 0;                        //标志已经给子进程发送完信号
            }
        }

    } else if (pid == 0){
        n = 2;      //子进程从2开始数
        act.sa_handler = do_sig_child;
        sigemptyset(&act.sa_mask);
        act.sa_flags = 0;
        sigaction(SIGUSR1, &act, NULL);

        while(1) {
            /* wait for signal */;
            if (flag == 1) {
                kill(getppid(), SIGUSR2);
                flag = 0;
            }
        }
    }

    return 0;
}
```

问题在于flag的修改，若某一进程修改不及时，另一进程已经发送了信号，那么就会导致原进程在执行完信号后才修改flag为0，那么两者就会死锁。

这里的全局是原程序和内核回调函数的并行。

改进方法：直接不使用flag，把信号发送放到回调函数里面。

### 可重入函数

![/images/LinuxSystem/Untitled%2050.png](/images/LinuxSystem/Untitled%2050.png)

显然这个是不可重入的函数，主要原因是使用了全局变量。

注意事项：

- 可重入函数不能有全局变量以及static变量。不能有malloc和free。
- 信号捕捉函数应该设计为可重入函数。
- 信号处理程序可以调用的可重入函数可以参阅man 7 signal.
- 没有在以上列表中的函数大多是不可重入的。
  - 静态数据结构
  - 调用了malloc和free
  - 是标准的IO函数

## SIGCHLD回收子进程

![/images/LinuxSystem/Untitled%2051.png](/images/LinuxSystem/Untitled%2051.png)

对父进程发送信号。可以利用SIGCHLD信号回收子进程。

为了避免信号的覆盖，需要在回收函数里面使用while来处理所有的子进程。

![/images/LinuxSystem/Untitled%2052.png](/images/LinuxSystem/Untitled%2052.png)

这里使用if时，会使得一些进程没有回收。

## 信号传参

### 发送参数

sigqueue函数对应kill函数，可以在发送信号的同时携带参数。

![/images/LinuxSystem/Untitled%2053.png](/images/LinuxSystem/Untitled%2053.png)

但是注意传地址时，每个进程的虚拟地址空间独立，把虚拟地址传到另一进程没有实际意义。

### 接收参数

![/images/LinuxSystem/Untitled%2054.png](/images/LinuxSystem/Untitled%2054.png)

## 中断系统调用

系统调用：

- 慢速系统调用：可能会使得本进程永久阻塞。如pause、wait、waitpid、read
- 其他系统调用。

对于慢速系统调用，被信号打断后，按需求希望恢复操作或者跳过操作（read和pause）。

![/images/LinuxSystem/Untitled%2055.png](/images/LinuxSystem/Untitled%2055.png)

发送信号时，sa_flags参数可以设置是否被信号中断后重启。也可以设置该信号不自动被屏蔽。

# 终端、进程组、会话、守护进程

## 终端

是所有输入输出设备的总称。所有的进程都有一个父进程init。每个进程都可以通过/dev/tty访问它的控制终端。

### 启动流程

![/images/LinuxSystem/Untitled%2056.png](/images/LinuxSystem/Untitled%2056.png)

线路规程像一个过滤器，对某些字符进行特殊处理。如ctrl+c会被线路规程截获，而不是读到read中。

![/images/LinuxSystem/Untitled%2057.png](/images/LinuxSystem/Untitled%2057.png)

### ttyname函数

借助ttyname函数可以看到不同终端对应的设备文件名。

### 网络终端

![/images/LinuxSystem/Untitled%2058.png](/images/LinuxSystem/Untitled%2058.png)

## 进程组 ps ajx

作业就是进程组，代表一个或多个进程的集合。

当父进程创建子进程，默认子进程与父进程为同一个组，组ID为父进程的ID（组长）。

![/images/LinuxSystem/Untitled%2059.png](/images/LinuxSystem/Untitled%2059.png)

### getpgrp 获取进程组ID

### getpgid 获取指定进程的进程组id

### setpgid 设定指定进程的进程组id

注意非root进程只能改变自己的子进程。

## 会话

一组进程组可以编号成一个会话。

### 创建会话

![/images/LinuxSystem/Untitled%2060.png](/images/LinuxSystem/Untitled%2060.png)

### getsid 查看会话id

### setsid 设置会话id

会话可以做守护进程。

## 守护进程

Daemon进程，是Linux中的后台服务进程，通常独立于控制终端且周期性执行某种任务或等待某个事件。通常以d结尾（httpd、sshd）。且不受用户登录退出的影响。

 最关键的一步：使用setsid创建一个新的session，并成为session leader。

### 创建方式

- 创建子进程，父进程退出 fork

- 在子进程中创建新会话 setsid

- 设置当前目录为根目录（防止被卸载） chdir

- 重设文件权限掩码 umask函数（防止继承的文件创建屏蔽字拒绝某些权限）

- 关闭文件描述符（不要浪费系统资源）
  
    通常重定向到/dev/null   dup2()

- 开始守护核心工作

- 守护进程退出模型（很少用到）

```c
#include<stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>

int main()
{
    pid_t pid, spid;

    pid = fork();
    if(pid == 0) //child
    {
        spid = setsid();
        int ret = chdir("/home/hongwei");
        if(ret == -1){
            perror("chdir");
            exit(1);
        }
        umask(0002);

        close(STDIN_FILENO);
        int fd = open("/dev/null", O_RDWR);
        if(fd < 0){
            perror("open");
            exit(1);
        }
        dup2(fd, STDOUT_FILENO);
        dup2(fd, STDERR_FILENO);
        while(1){
            sleep(1);    
        }

    }
    else if(pid > 0){
        return 0;
    }
    return 0;
}
```

在bashrc文件中加入运行命令，则可以保证每次开机自动启动。

# 线程

## 概念

Linux下仍是轻量级的进程，差别不大。

进程：独立的地址空间、PCB

线程：也有PCB，但是没有独立的地址空间（共享）。

![/images/LinuxSystem/Untitled%2061.png](/images/LinuxSystem/Untitled%2061.png)

进程（独居），线程（合租）

**线程是最小的执行单位，进程是最小的资源分配的单位。**

### Linux线程实现原理

- 线程是由进程发展而来，底层实现类似。都是用clone()，也有PCB。
- 从内核的角度来看，进程和线程都一样，有不同的PCB。但是PCB中指向内存的三级页表对于线程来说是相同的。

![/images/LinuxSystem/Untitled%2062.png](/images/LinuxSystem/Untitled%2062.png)

![/images/LinuxSystem/Untitled%2063.png](/images/LinuxSystem/Untitled%2063.png)

- 线程可以看成是寄存器和栈的集合。

![/images/LinuxSystem/Untitled%2064.png](/images/LinuxSystem/Untitled%2064.png)

内核中的栈用于记录临时的寄存器值，便于恢复。

LWP号（线程号，区分于线程id）   ps -Lf pid。

线程号：CPU分配运行时间的依据。

线程id：进程内部区分线程的方法。

### 线程共享资源

- 文件描述符表
- 每种信号的处理方式（尽量不要线程和信号一起使用）
- 当前工作目录
- 用户ID和组ID
- 内存地址空间 .text .data .bss heap 共享库

### 非共享资源

- 线程id
- 处理器现场和栈指针（内核栈）
- 独立的栈空间（用户栈）
- errno变量
- 信号屏蔽字
- 调度优先级

### 优缺点

优点：

- 提高程序并发性
- 开销小
- 数据通信、共享数据方便

缺点：

- 库函数不稳定（pthread）
- 调试困难，gdb不支持
- 对信号支持不好

在Linux下，由于实现方法导致进程线程的差别不大。 

## 控制原语

### pthread_self

对应getpid（）获取线程id。

### pthread_create

对应fork（）

![/images/LinuxSystem/Untitled%2065.png](/images/LinuxSystem/Untitled%2065.png)

传出线程id，传入属性（可以是NULL），主控函数，函数参数。

注意返回值为错误编号（非-1）。

存疑：注意传参时最好为值传递，内存地址情况可能会发生改变。

### pthread_exit

将单个线程退出。如果是主控线程，则可以使得子线程继续运行而非退出。exit函数会直接把进程结束。return作用为返回值到调用者处。                                                                                                                                                 

### pthread_join

对应waitpid（）。回收线程。

![/images/LinuxSystem/Untitled%2066.png](/images/LinuxSystem/Untitled%2066.png)

返回的retval若传指针则需要malloc。也可以在main中malloc。

![/images/LinuxSystem/Untitled%2067.png](/images/LinuxSystem/Untitled%2067.png)

线程的回收不需要一定是主控线程。

### pthread_detach

实现线程分离，对线程在状态上实现分离，其退出状态不被获取，且自己释放，不会成为僵尸线程。

![/images/LinuxSystem/Untitled%2068.png](/images/LinuxSystem/Untitled%2068.png)

### pthread_cancel

对应kill，直接杀死某线程。但是需要线程到达一定的检查点。

使用man 7 pthreads 查看所有取消点。如read pause open creat close...

也可以自定义取消点 pthread_testcancel()。

### 原语对比

getpid : pthread_self 

fork:  pthread_create

wait:  pthread_join(tid, void**)

exit(): pthread_exit(void*)

kill():  pthread_cancel();  到达取消点

: pthread_detach 自动清理pcb

## 线程属性

![/images/LinuxSystem/Untitled%2069.png](/images/LinuxSystem/Untitled%2069.png)

pthread_attr_init/destroy 函数初始化和销毁这个属性结构体。

![/images/LinuxSystem/Untitled%2070.png](/images/LinuxSystem/Untitled%2070.png)

![/images/LinuxSystem/Untitled%2071.png](/images/LinuxSystem/Untitled%2071.png)

调整栈的大小使得能建立更多的线程。

![/images/LinuxSystem/Untitled%2072.png](/images/LinuxSystem/Untitled%2072.png)

注意此时是在堆上创建线程。

## NPTL 线程库版本

getconf GNU_LIBPTHREAD_VERSION

## 注意事项

- 避免僵尸进程 join detach
- 避免使用fork（）
- 避免使用信号

# 线程同步

## 概念

线程的同步是指协同，互相配合，线程按照一定的先后次序运行。

一个线程发出某一功能调用时，直到这个功能调用结束为止，其他线程不调用这个功能。

数据发生时间错误条件：

- 数据共享。
- 竞争。
- 多个对象没有合理的同步机制。

## 互斥量 mutex

结构体pthread_mutex_t(可以看做一个整数）。

### pthread_mutex_init（destory）

![/images/LinuxSystem/Untitled%2073.png](/images/LinuxSystem/Untitled%2073.png)

![/images/LinuxSystem/Untitled%2074.png](/images/LinuxSystem/Untitled%2074.png)

restrict关键字：

[如何理解C语言关键字restrict？](https://www.zhihu.com/question/41653775)

restrict 是为了告诉编译器额外信息（两个指针不指向同一数据），从而生成更优化的机器码。

![/images/LinuxSystem/Untitled%2075.png](/images/LinuxSystem/Untitled%2075.png)

### pthread_mutex_lock

用阻塞等待方法加锁。

![/images/LinuxSystem/Untitled%2076.png](/images/LinuxSystem/Untitled%2076.png)

### pthread_mutex_trylock

不阻塞方法加锁。（轮询）

### pthread_mutex_unlock

解锁。

锁的粒度应该越小越好。

### 死锁

1、线程尝试对锁加两次锁。

2、多个线程互相等待。

![/images/LinuxSystem/Untitled%2077.png](/images/LinuxSystem/Untitled%2077.png)

解决方法：

- 使用pthread_mutex_trylock，若失败则放弃已有的锁，日后再加锁。

## 读写锁

读写锁是一把锁。具备三种状态

- 读锁
- 写锁
- 不加锁

写独占，读共享，写锁优先度高。（写者优先）pthread_rwlock_t

![/images/LinuxSystem/Untitled%2078.png](/images/LinuxSystem/Untitled%2078.png)

## 条件变量

条件变量不是锁，但是可以造成线程阻塞，通常与互斥锁搭配使用。 pthread_cond_t

![/images/LinuxSystem/Untitled%2079.png](/images/LinuxSystem/Untitled%2079.png)

![/images/LinuxSystem/Untitled%2080.png](/images/LinuxSystem/Untitled%2080.png)

### pthread_cond_wait

pthread_cond_wait函数有三个功能：

- 阻塞等待一个条件变量满足。
- 释放已经掌握的互斥锁。
  - 以上两个为原子操作
- 当被唤醒，解除阻塞并重新申请获取互斥锁。

使用pthread_cond_signal唤醒一个线程。pthead_cond_boardcast唤醒所有线程。

### pthread_cond_timedwait

在指定时间内等待。

![/images/LinuxSystem/Untitled%2081.png](/images/LinuxSystem/Untitled%2081.png)

这里的abstime为timespec结构体：

![/images/LinuxSystem/Untitled%2082.png](/images/LinuxSystem/Untitled%2082.png)

abstime是绝对时间(相对于unix诞生时间):

```c
time_t cur = time(NULL);
struct timespec t;
t.tv_sec = cur + 1;
pthread_cond_timedwait(&cond, &mutex, &t);
```

### 使用条件变量解决生产者消费者模型

![/images/LinuxSystem/Untitled%2083.png](/images/LinuxSystem/Untitled%2083.png)

条件变量代表是否缓冲区有产品。

条件变量的优点在于减少了竞争，消费者之间在生产时不再需要竞争互斥锁。

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include<pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t has_product = PTHREAD_COND_INITIALIZER;

struct msg{
    int num;
    struct msg *next;
};

struct msg *head = NULL, *mp = NULL, *mc = NULL;

void* producer(void* arg){
    while(1){
        mp = malloc(sizeof(struct msg));
        mp -> num = rand() % 400 + 1;
        printf("=>produced %d\n", mp -> num);

        pthread_mutex_lock(&mutex);
        mp -> next = head;
        head = mp;
        pthread_mutex_unlock(&mutex);
        int ret = pthread_cond_signal(&has_product);
        if(ret != 0){
            printf("signal failed\n");
            pthread_exit(NULL);
        }
        sleep(rand() % 3);
    }
    return NULL;
}

void* consumer(void* arg){
    while(1){
        pthread_mutex_lock(&mutex);
        while(head == NULL){
            pthread_cond_wait(&has_product, &mutex);
        }
        mc = head;
        head = mc -> next;
        pthread_mutex_unlock(&mutex);
        printf("=>consumed: %d\n", mc->num);
        free(mc);
        mc = NULL;
        sleep(rand() % 2);
    }
    return NULL;
}

int main()
{
    pthread_t ptid, ctid;
    int ret = pthread_create(&ptid, NULL, producer, NULL);
    if(ret != 0){
        printf("create error\n");
        exit(1);
    }
    ret = pthread_create(&ctid, NULL, consumer, NULL);
    if(ret != 0){
        printf("create error\n");
        exit(1);
    }

    pthread_join(ptid, NULL);
    pthread_join(ctid, NULL);

}
```

## 信号量（也可以应用于进程间同步）

进化版的互斥锁。留意信号量的函数都是用errno来返回错误信息。

sem_t 信号量结构体。

### sem_init

初始化信号量。

![/images/LinuxSystem/Untitled%2084.png](/images/LinuxSystem/Untitled%2084.png)

第二个参数代表是否能在进程间共享，当非0，表示可以共享。

### sem_destroy

销毁信号。

### sem_wait

信号量减一

若信号量小于0，线程阻塞。

![/images/LinuxSystem/Untitled%2085.png](/images/LinuxSystem/Untitled%2085.png)

### sem_trywait

### sem_timedwait

### sem_post

信号量加一，若信号量小于等于0，同时唤醒信号量上的进程。

### 生产者消费者

```c
void *producer(void *arg)
{
    int i = 0;

    while (1) {
        sem_wait(&blank_number);                    //生产者将空格子数--,为0则阻塞等待
        queue[i] = rand() % 1000 + 1;               //生产一个产品
        printf("----Produce---%d\n", queue[i]);        
        sem_post(&product_number);                  //将产品数++

        i = (i+1) % NUM;                            //借助下标实现环形
        //sleep(rand()%3);
    }
}

void *consumer(void *arg)
{
    int i = 0;

    while (1) {
        sem_wait(&product_number);                  //消费者将产品数--,为0则阻塞等待
        printf("-Consume---%d\n", queue[i]);
        queue[i] = 0;                               //消费一个产品 
        sem_post(&blank_number);                    //消费掉以后,将空格子数++

        i = (i+1) % NUM;
        //sleep(rand()%3);
    }
}
```

为什么不需要互斥锁？

## 进程间同步

### 互斥量

进程中的互斥量需要修改mutex的属性。

![/images/LinuxSystem/Untitled%2086.png](/images/LinuxSystem/Untitled%2086.png)

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <sys/mman.h>
#include <sys/wait.h>

struct mt {
    int num;
    pthread_mutex_t mutex;
    pthread_mutexattr_t mutexattr;
};

int main(void)
{
    int i;
    struct mt *mm;
    pid_t pid;
/*
    int fd = open("mt_test", O_CREAT | O_RDWR, 0777);
    ftruncate(fd, sizeof(*mm));
    mm = mmap(NULL, sizeof(*mm), PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
    close(fd);
    unlink("mt_test");
*/
    mm = mmap(NULL, sizeof(*mm), PROT_READ|PROT_WRITE, MAP_SHARED|MAP_ANON, -1, 0);
    memset(mm, 0, sizeof(*mm));

    pthread_mutexattr_init(&mm->mutexattr);                                  //初始化mutex属性对象
    pthread_mutexattr_setpshared(&mm->mutexattr, PTHREAD_PROCESS_SHARED);    //修改属性为进程间共享

    pthread_mutex_init(&mm->mutex, &mm->mutexattr);                          //初始化一把mutex琐

    pid = fork();
    if (pid == 0) {
        for (i = 0; i < 10; i++) {
            pthread_mutex_lock(&mm->mutex);
            (mm->num)++;
            pthread_mutex_unlock(&mm->mutex);
            printf("-child----------num++   %d\n", mm->num);
        }
    } else if (pid > 0) {
        for ( i = 0; i < 10; i++) {
        //    sleep(1);
            pthread_mutex_lock(&mm->mutex);
            mm->num += 2;
            pthread_mutex_unlock(&mm->mutex);
            printf("-------parent---num+=2  %d\n", mm->num);
        }
        wait(NULL);
    }

    pthread_mutexattr_destroy(&mm->mutexattr);          //销毁mutex属性对象
    pthread_mutex_destroy(&mm->mutex);                  //销毁mutex
    munmap(mm,sizeof(*mm));                             //释放映射区

    return 0;
}
```

## 文件锁

### fcntl函数

修改已经打开文件的属性，修改阻塞和非阻塞。

![/images/LinuxSystem/Untitled%2087.png](/images/LinuxSystem/Untitled%2087.png)

可以借助其来设置文件锁。

![/images/LinuxSystem/Untitled%2088.png](/images/LinuxSystem/Untitled%2088.png)

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>

void sys_err(char *str)
{
    perror(str);
    exit(1);
}

int main(int argc, char *argv[])
{
    int fd;
    struct flock f_lock;

    if (argc < 2) {
        printf("./a.out filename\n");
        exit(1);
    }

    if ((fd = open(argv[1], O_RDWR)) < 0)
        sys_err("open");

    f_lock.l_type = F_WRLCK;        /*选用写琐*/
//    f_lock.l_type = F_RDLCK;      /*选用读琐*/ 

    f_lock.l_whence = SEEK_SET;
    f_lock.l_start = 0;
    f_lock.l_len = 0;               /* 0表示整个文件加锁 */

    fcntl(fd, F_SETLKW, &f_lock);
    printf("get flock\n");

    sleep(10);

    f_lock.l_type = F_UNLCK;
    fcntl(fd, F_SETLKW, &f_lock);
    printf("un flock\n");

    close(fd);

    return 0;
}
```

多线程间不可以用文件锁，因为文件描述符共享。

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/linuxsystem/  

