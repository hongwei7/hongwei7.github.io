# C++偏特化的使用


上次面试被问到了偏特化的使用, 现在来总结一下C++里面偏特化的用法。

## 类模板

类在不同的模板下，数据成员可以不同。每一份模板参数生成一份代码，这个过程发生在编译期。编译器会针对每一份模板参数生成一份代码，且对静态的数据成员，不同的模板下的静态数据成员独立存储。

类可以被全特化，也可以被偏特化。类的成员函数可以被全特化。

## 函数模板

函数模板的用法与类类似，但是函数模板不能偏特化。原因：

- C++标准没有规定，而且函数重载可以实现偏特化的效果。

## code

```cpp
#include <iostream>

using namespace std;

template <typename A, typename B>
class Basic {
public:
    A a;
    B b;
    void run() {
        cout << "run from Basic<A, B>" << endl;
    }
};

template <>
void Basic<bool, bool>::run() { cout << "run from Basic<bool, bool>" << endl; }

template <typename B>
class Basic<int, B> {
public:
    double a;
    B b;
    void run() {
        cout << "run from Basic<int, B>" << endl;
    }
};

template <typename A, typename B>
void f() { cout << "<A, B>" << endl; }

template <>
void f<int, char>() { cout << "<int, char>" << endl; }

int main() {
    Basic<int, char> a;
    Basic<char, char> aa;
    a.a = 1.1;
    aa.a = 'a';
    cout << a.a << endl;
    cout << aa.a << endl;
    a.run();
    aa.run();
    Basic<bool, bool>().run();

    f<int, int>();
    f<int, char>();
}

//函数不能偏特化，可以通过函数重载实现偏特化的目的
```


---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/c-%E5%81%8F%E7%89%B9%E5%8C%96%E7%9A%84%E4%BD%BF%E7%94%A8/  

