# 一些简单的设计模式


作为补充，开始了解一些简单的设计模式。

# 应该掌握的

- 单例
- 工厂
- 代理
- 策略模式
- 模板方法

## 简单工厂模式

考虑用一个单独的类去创建实例。

```cpp
#include<iostream>
#define DBG_MACRO_NO_WARNING 
#include<dbg.h>
using namespace std;

template<typename T>
class Operation{
public:
    Operation(){}
    Operation(const T& n1,const T& n2): N1(n1), N2(n2){}
    virtual T getResult() const{
        return 0;
    }
    void setA(const T& val){N1 = val;}
    void setB(const T& val){N2 = val;}
protected:
    T N1;
    T N2;
};

template<typename T>
class OperationPlus: public Operation<T>{
public:
    OperationPlus(const T n1, const T n2): Operation<T>(n1, n2){};
    T getResult() const override{
        return this->N1 + this->N2;
    }
};

template<typename T>
class Factory{
public:
    Operation<T>* createOperation(const T& a, const T& b, string oper){
        if(oper == "+"){
            op = new OperationPlus<T>(a, b);
        }
        return op;
    }
    virtual ~Factory(){
        delete op;
    }
private:
    Operation<T> *op;
};

int main(){
    Factory<int> factory;
    auto oper = factory.createOperation(1, 2, "+");
    dbg(oper->getResult());
}
```

![/images/designPattern/Untitled.png](/images/designPattern/Untitled.png)

## 策略模式

定义了算法家族，分别封装起来，可以相互替换。使得算法的变化不影响客户。

把某个实现方法抽象为接口，使用组合而不是继承。

想修改具体某个算法时，不需要其用户改变。

![/images/designPattern/Untitled%201.png](/images/designPattern/Untitled%201.png)

![/images/designPattern/Untitled%202.png](/images/designPattern/Untitled%202.png)

一言以蔽之，使用组合把算法做成接口。

## 单一职责原则

就一个类而言，应该仅有一个引起它变化的原因。

## 开放-封闭原则

软件实体应该可以扩展，但是不可修改。

## 依赖倒转原则

抽象不应该依赖细节，细节应该依赖于抽象。针对接口编程。

## 里氏代换原则

子类必须能够替换掉他们的父类。

## 装饰模式

动态地给一个对象添加一些额外的职责。

![/images/designPattern/Untitled%203.png](/images/designPattern/Untitled%203.png)

![/images/designPattern/Untitled%204.png](/images/designPattern/Untitled%204.png)

## 代理模式

![/images/designPattern/Untitled%205.png](/images/designPattern/Untitled%205.png)

为其他对象提供一种代理以控制对这个对象的访问。Proxy 类里面有一个 RealSubject，真实访问类。

![/images/designPattern/Untitled%206.png](/images/designPattern/Untitled%206.png)

## 工厂方法模式

![/images/designPattern/Untitled%207.png](/images/designPattern/Untitled%207.png)

![/images/designPattern/Untitled%208.png](/images/designPattern/Untitled%208.png)

定义一个用于创建对象的接口，让子类决定实例化哪个类。即将一个类的实例化延迟到子类。

简单工厂实际上就是工厂方法去掉 Creator。简单工厂只有一种生产方式，而工厂方法可以实现多种工厂以应对多种生产方式。又或者可以传递参数给工厂，实现工厂的动态变化生产。

## 模板方法模式

定义一个操作中算法的骨架，将一些步骤延迟到子类。模板方法使得子类能不改变算法的结构也能重定义算法的某些步骤。

![/images/designPattern/Untitled%209.png](/images/designPattern/Untitled%209.png)

## 原型模式

用原型实例创建指定对象的种类，并通过拷贝创建新的对象。

## 抽象工厂方法

实际上抽象工厂就是多个工厂方法模式，区别就是在于抽象工厂会产生多种类的实例。而工厂方法仅仅是一种类。在抽象工厂中，任何一个工厂实例都有能力生产各种实例。

抽象工厂提供了一个创建实例的接口，用于创建相关或依赖对象的家族，但是不需要指定具体的类。

![/images/designPattern/Untitled%2010.png](/images/designPattern/Untitled%2010.png)

![/images/designPattern/Untitled%2011.png](/images/designPattern/Untitled%2011.png)

![/images/designPattern/Untitled%2012.png](/images/designPattern/Untitled%2012.png)

## 单例模式

保证一个类仅有一个实例，且提供一个访问它的全局访问点。

```cpp
class singleton {
private:
    singleton() {}
    static singleton *p;
public:
    static singleton *instance();
};

singleton *singleton::p = nullptr;

singleton* singleton::instance() {
    if (p == nullptr)
        p = new singleton();
    return p;
}
```

线程安全的版本（定义时即初始化）

```cpp
class singleton {
private:
    singleton() {}
    static singleton *p;
public:
    static singleton *instance();
};

singleton *singleton::p = new singleton();
singleton* singleton::instance() {
    return p;
}
```

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/%E4%B8%80%E4%BA%9B%E7%AE%80%E5%8D%95%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/  

