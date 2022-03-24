# 实现shared_ptr


shared_ptr 主要依赖一个引用计数指针和一个 mutex 锁来实现管理对象生命周期。

## code

```cpp
#include <iostream>
#include <mutex>

using namespace std;

template<typename T>
class myshared_ptr {
public:
    myshared_ptr(T* ptr)
    : realPtr(ptr), refCount(new int(1)), lock(new mutex) {
        cout << "construct, refCount: 1" << endl; 
    }

    myshared_ptr(const myshared_ptr<T>& other)
    : realPtr(other.realPtr), refCount(other.refCount), lock(other.lock) {
        lock->lock();
        ++(*refCount);
        cout << "refCount: " << *refCount << endl;
        lock->unlock();
    }

    T& operator*() {
        return *realPtr;
    }

    T* operator->() {
        return realPtr;
    }

    ~myshared_ptr() {
        lock->lock();
        --(*refCount);
        if(*refCount == 0) {
            delete realPtr;
            cout << "delete realPtr" << endl;
            lock->unlock();
            delete lock;
            return;
        }
        lock->unlock();
    }

private:
    const myshared_ptr<T>& operator=(const myshared_ptr<T> other) { }

private:
    T* realPtr;
    int* refCount;
    mutex* lock;
};

class A {
public:
    int val;
    char c;
};

void fun() {
    myshared_ptr<A> aPtr(new A);   
    myshared_ptr<A> bPtr = aPtr;

    aPtr->val = 101;
    aPtr->c = 'Y';

    cout << (*aPtr).val << " " << (*bPtr).c << endl;
}

int main() {
    fun();
}
```

