# STL 中如何为自定义的类编写比较函数和哈希函数


 在 STL 中，unordered_set 需要所选对象具有哈希函数，而 set 要求所选对象的 key 有比较大小的函数。本文就来简单展开讨论。

## 自定义哈希函数

unordered_set 的底层为 hashtable，解决哈希冲突的方法为链表法。默认的哈希函数定义为 `hash<key>`。我们可以自定义一个 hash 函数例如：

```cpp
#include <unordered_set>
#include <iostream>

using namespace std;

class A {
public:
    A(int aa, int bb): a(aa), b(bb) {}
    char a;
    int b;
};

class hash_funA {
public:
    int operator()(const A &a) const {
        return a.a + a.b;
    }
};

class equalA {  //仿函数，比较两个对象是否相同
public:
    bool operator()(const A& a, const A& b) const {
        return a.a == b.a && a.b == b.b;
    }
};

int main() {
    unordered_set<A, hash_funA, equalA> s;
    for(int i = 0; i < 100; ++i)
        s.insert(A(i, i));
}
```

其基本原理是：使用一个数组，把 key 映射到某个数组下标，就把对象存放在那个数组下标上。但是会发生哈希冲突的问题。因此哈希表的插入过程为：

- 得到 key
- 算出 hash
- 求模后得到 index
- 存放对象

取出过程相似：

- 从 key 得到 hash
- 求模后得到 index
- 检查 index的元素是否与 key 相同。
- 取出对象

那么为什么需要比较两个对象是否一样的 `equalA` 呢。答案是因为多个 key 可能对应相同的 hash，因此一个查到对应的桶之后，需要检查桶里面的 key 是否与 key 真的一致，这就需要`euqalA`了。

或者这里也不需要 `equalA`，直接在类内定义一个`==`的重载函数即可。

```cpp
class A {
public:
    A(int aa, int bb): a(aa), b(bb) {}
    char a;
    int b;
    bool operator==(const A& other) const {
        return other.a == a && other.b == b;
    }
};
```


## 自定义比较函数

我们在使用 set 时，若是自定义类的话，需要定义类的比较函数。

```cpp
#include <set>
#include <iostream>

using namespace std;

class A {
public:
    A(int aa, int bb): a(aa), b(bb) {}
    char a;
    int b;
    bool operator<(const A& other) const {
        return (a == other.a)? b < other.b: a < other.a;
    }
};

int main() {
    set<A> s;
    for(int i = 0; i < 100; ++i)
        s.insert(A(i, i));
}
```

同样是通过操作符重载来实现。

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/stl-hash-map/  

