# IPC中的信号量探究


在多个进程同时对一共享变量访问修改时就会产生竞争条件,若不进行互斥和同步控制,程序运行结果就会出现偏差.为了解决这一问题 E.W.Dijkstra 在1965年提出了信号量.<!-- more-->

### 什么是信号量

信号量是一种特殊的变量,使用一个整形变量来累计唤醒次数.设立两种操作`up()` 和`down()`来控制信号量.

```c
void down(semaphore s)//此操作为原子操作
{
    s--;
  if(s<0)
    sleep(s); //进程进入阻塞状态
}
void up(semaphore s)//此操作为原子操作
{
  s++;
  if(s<=0)
    wakeup(s);//从信号量s的进程中唤醒一个
}
```

### 以生产者-消费者模型为例

例子包括了三个信号量:`mutex` 、`empty `和`full`

```c
#define N 100
typedef int semaphore;
semphore mutex = 1;
semphore empty = N;
semphore full = 0;

void producer()
{
  int item;
  while(TRUE){
    item = produce_item();
    down(&empty);
    down(&mutex);
    instert_item(item);
    up(&mutex);
    up(&full);
  }
}
void consumer()
{
  int item;
  while(TRUE){
    down(&full);
    down(&mutex);
    item = remove_item();
    up(&mutex);
    up(&empty);
  }
}
```

#### mutex的作用（互斥信号量）

`mutex`限定了临界区只有一个进程运行(当`mutex`=1,`producer`可以进入临界区;当`mutex`=0,`consumer`可以进入临界区).

#### empty和full的作用(同步信号量)

`empty`与`full`是对偶的关系.两个信号量都可以起到保存信号的作用.例如当在简单的sleep-wakeup模型中,`producer`在阻塞前一瞬间被抢占CPU,切换到`consumer`并试图唤醒`producer`,若此时`consumer`也进入阻塞,那当切换回`producer`,两者都将同时阻塞.

而在信号量模型下,以上事件会变成:`producer`在`down(&empty)`函数执行前被抢占CPU,此时`empty`=0,`consumer`将开始执行,循环多次后`empty`=N,`full`=0,`consumer`进入阻塞,这时切回`producer`,`down(&empty)`却不会使`producer`阻塞,反而在执行到`up(&full)`的时候会唤醒`consumer`.这样一来信号量很好的解决了这个问题.信号量的原子操作起到了决定性作用.

##### 总结作用

- 保证事件发生的顺序
- 缓冲区满时,producer停止运行
- 缓冲区空时,consumer停止运行
- 读者优先:若已经有读者进入,后续读者皆可以进入
- 写者优先:只要有写者等待,后续写者必须等待
- 合理顺序:读者在先来的写者之后

### 解决思路

#### 互斥信号量

限制进程进入进入临界区

#### 同步信号量

1. 定义信号量:**数值代表可用消息数** 数值为负:有等待进程 数值为正:有可用消息
2. 等待消息进程:`down(s)` 无消息则本进程进入等待 有消息则读入消息
3. 发出消息进程:`up(s)` 无等待进程则发出消息 有等待进程则释放等待进程

#### 解析

##### empty信号量

- 信息：缓冲区所拥有的空格
- 行为：生产者调用`down()`减empty，若empty<0进入阻塞态，消费者调用`up()`增加empty，若empty<=0尝试唤醒生产者。

##### full信号量

- 信息：缓冲区装填的格子
- 行为：消费者调用`down()`减full，若full<0进入阻塞态，生产者调用`up()`增加full，若full<=0尝试唤醒消费者。

### 写者优先的读者写者问题

当读者数量远大于写者时，写者可能会一直处于阻塞状态而不能的到CPU。

此时需要写者优先规则（虽然会减少一部分并行效率）。

```c++
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#define ProNum 3  //生产者数量
#define ConNum 30 //消费者数量
#define N 6 //缓冲区大小
#define TRUE 1
typedef int semaphore;
sem_t mutex;
sem_t can_read; //是否允许读者读取（实现写者优先）
char buffer[N];//缓冲区
int pointer=0;//缓冲区指针
void* produce(void * tid)
{
    char item;
    int t=0;
    while(TRUE){
        if(t%2==0)
            item='A'+(t++)%10;
        else item='a'+(t++)%10;
        sleep(2);
        sem_wait(&mutex);
        printf("Write %c\n",item);
        buffer[pointer]=item;
        pointer=(pointer+1)%N;
        sem_post(&mutex);
        sem_post(&can_read); //写者写完后up(&can_read)
    }
}
void* consume(void* tid)
{
    char item;
    while(TRUE){
        printf("%d tid ask for read\n",tid);
        sem_wait(&can_read); //读者读前确保有剩余的读者可读信号量
        sem_wait(&mutex);
        pointer=abs((pointer-1)%N);
        item=buffer[pointer];
        buffer[pointer]=NULL;
        sem_post(&mutex);
        printf("Read: %c\n",item);
    }
}
int main()
{
    printf("started!\n");
    sem_init(&mutex,0,1);
    sem_init(&can_read,0,0);
    pthread_t threads[ProNum+ConNum];
    int status,i;
    for(i=0;i<ProNum;i++)
    {
        status=pthread_create(&threads[i],NULL,produce,(void *)i);
    }
    for(i=ProNum;i<ProNum+ConNum;i++)
    {
        status=pthread_create(&threads[i],NULL,consume,(void *)i);
    }
    pthread_exit(0);
    return 0;
}
```

可以把`sem_post()`看成下放允许，而`sem_wait()`看成是等待允许。


---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/ipc%E4%B8%AD%E7%9A%84%E4%BF%A1%E5%8F%B7%E9%87%8F%E6%8E%A2%E7%A9%B6/  

