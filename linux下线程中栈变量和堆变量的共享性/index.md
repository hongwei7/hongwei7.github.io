# Linux下线程中栈变量和堆变量的共享性


在 CSAPP 中，说到在操作系统的线程概念模型中，每个线程有独立的栈，而线程之间不能访问其他栈的变量。

{{< admonition info "概念模型" true >}}
线程独享有：

- 线程 ID
- 栈
- 栈指针
- PC 程序计数器
- 条件状态位
- 寄存器

不独享的有：

- 指令代码
- .data
- 堆
- shared library
{{< /admonition >}}

但是在实际的实现中，事情会变得不一样。
{{< admonition info "实际实现" true >}}
线程实际独立的东西只有：

- 线程 ID
- PC 程序计数器
- 栈指针
- 寄存器
- 条件状态位

而且线程之间的栈可以互相访问。
{{< /admonition >}}

为了探究这一特性，做了相关的测试代码。

```cpp
/*
* 创建两个全局指针 a，b，首先由 线程 1 修改两个指针分别指向堆变量和栈变量
* 线程 2 分别访问两个指针指向的地址，检验堆变量和栈变量的可访问性
*/

#include <dbg.h>
#include <pthread.h>

int* a = nullptr;
int *b = nullptr;

void* fun1(void*) {
    sleep(1);
    dbg("------1s------");

    dbg(a = new int(1000));
    int c = 200;
    dbg(b = &c);

    return nullptr;
}

void* fun2(void*) {
    dbg(a);
    dbg(b);

    sleep(2);
    dbg("------2s------");

    dbg(a, *a);
    dbg(b, *b);
    return nullptr;
} 

int main() {
    pthread_t p1, p2;
    pthread_create(&p1, nullptr, fun1, nullptr);
    pthread_create(&p2, nullptr, fun2, nullptr);
    pthread_join(p2, nullptr);
}
```

{{< admonition tip "提示：" true >}}
推荐使用 {{< link "https://github.com/sharkdp/dbg-macro" dbg-macro >}} 方便地输出调试信息。
{{< /admonition >}}

结果在 Ubuntu 18.0 中，测试结果如下
```
[test.cpp:19 (fun2)] a = nullptr (int*)
[test.cpp:20 (fun2)] b = nullptr (int*)
[test.cpp:9 (fun1)] ------1s------
[test.cpp:11 (fun1)] a = new int(1000) = 0x7fa154000e10 (int*&)
[test.cpp:13 (fun1)] b = &c = 0x7fa162597e64 (int*&)
[test.cpp:23 (fun2)] ------2s------
[test.cpp:25 (fun2)] a = 0x7fa154000e10 (int*)
[test.cpp:25 (fun2)] *a = 1000 (int&)
[test.cpp:26 (fun2)] b = 0x7fa162597e64 (int*)
[test.cpp:26 (fun2)] *b = 200 (int&)
```
在 Mac OS X 中 测试结果如下
```
[test.cpp:19 (fun2)] a = nullptr (int*)
[test.cpp:20 (fun2)] b = nullptr (int*)
[test.cpp:9 (fun1)] ------1s------
[test.cpp:11 (fun1)] a = new int(1000) = 0x7f81c4d040d0 (int*&)
[test.cpp:13 (fun1)] b = &c = 0x7000003f6e9c (int*&)
[test.cpp:23 (fun2)] ------2s------
[test.cpp:25 (fun2)] a = 0x7f81c4d040d0 (int*)
[test.cpp:25 (fun2)] *a = 1000 (int&)
[test.cpp:26 (fun2)] b = 0x7000003f6e9c (int*)
[1]    54143 segmentation fault  "/Users/lulu/Desktop/code/"test
```

{{< admonition success "结果：" true >}}
说明在 Mac OS X 中，线程之间的栈不能相互访问，而在 Ubuntu 中，线程之间可以访问各自的栈变量。而两个系统环境下，不同线程下可以互相访问各自的 heap 变量。
{{< /admonition >}}

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/linux%E4%B8%8B%E7%BA%BF%E7%A8%8B%E4%B8%AD%E6%A0%88%E5%8F%98%E9%87%8F%E5%92%8C%E5%A0%86%E5%8F%98%E9%87%8F%E7%9A%84%E5%85%B1%E4%BA%AB%E6%80%A7/  

