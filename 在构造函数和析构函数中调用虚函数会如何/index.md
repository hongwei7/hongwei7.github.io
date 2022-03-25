# 在构造函数和析构函数中调用虚函数会如何


在《Effective C++》中，作者提到绝对不要在构造函数和析构函数中调用虚函数。在构造函数和析构函数中，虚机制是失效的，调用的永远是该析构函数所在类的对应函数。网上提出一些解释说明：

>这个是未定义行为。vs下的类的首地址的内容为虚表指针，占4个或者8个字节，取决于是32位还是64位程序。在基类初始化时，虚表指针指向基类虚表，子类初始化时，将其改写为子类虚表地址。因此在基类中调用虚方法一定是调用了基类的虚函数。析构函数中也一样，只不过虚表指针的改写顺序是相反的。所以无论如何不要在析构函数或者构造函数中调用虚函数。

```cpp
#include <dbg.h>
using namespace std;

class A {
public:
    A() {dbg("A()");}
    ~A() {dbg("~A()");}
};

class Base {
public:
    A a;
    Base() {dbg("Base()");run();}
    ~Base() {dbg("~Base()");run();}
    virtual void run() {dbg("father virtual function called.");}
};

class Derived: public Base {
public:
    void run() {dbg("child virtual function called.");}
    Derived() { dbg("Derived()"); run();}
    ~Derived() {dbg("~Derived()"); run();}
};

int main() {
    Derived b;
}
```

运行结果：

```
[test.cpp:6 (A)] A()
[test.cpp:13 (Base)] Base()
[test.cpp:15 (run)] father virtual function called.
[test.cpp:21 (Derived)] Derived()
[test.cpp:20 (run)] child virtual function called.
[test.cpp:22 (~Derived)] ~Derived()
[test.cpp:20 (run)] child virtual function called.
[test.cpp:14 (~Base)] ~Base()
[test.cpp:15 (run)] father virtual function called.
[test.cpp:7 (~A)] ~A()
```

可以看到虚机制失效，且能看出来构造函数的调用顺序为：基类构造函数、对象成员构造函数、派生类本身的构造函数。
