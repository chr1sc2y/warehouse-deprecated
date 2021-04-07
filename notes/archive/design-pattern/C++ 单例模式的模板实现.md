---
title: "C++ 单例模式的模板实现"
date: 2020-09-10T21:25:43+08:00
draft: false
categories: ["Design Pattern"]
---

# C++ 单例模式的模板实现

单例模式是一种创建型的设计模式（creational design patterns），使用单例模式进行设计的类在程序中只拥有一个实例（single instance），这个类称为单例类，它会提供一个全局的访问入口（global access point），关于单例模式的讨论可以参考[Singleton revisited](http://www.italiancpp.org/2017/03/19/singleton-revisited-eng/)；基于这两个特点，单例模式可以有以下几种实现：

## Meyer’s Singleton

*Scott Meyers* 在 *Effective C++* 的 *Item 4: Make sure that objects are initialized before they're used* 里面提出了一种利用 C++ 的 `static` 关键字来实现的单例模式，这种实现非常简洁高效，它的特点是：

1. 仅当程序第一次执行到 `GetInstance` 函数时，执行 `instance` 对象的初始化；
2. 在 C++ 11 之后，被 `static` 修饰的变量可以保证是线程安全的；

```cpp
template<typename T>
class Singleton
{
public:
    static T& GetInstance()
    {
        static T instance;
        return instance;
    }
    
    Singleton(T&&) = delete;
    Singleton(const T&) = delete;
    void operator= (const T&) = delete;

protected:
    Singleton() = default;
    virtual ~Singleton() = default;
};
```

通过禁用单例类的 copy constructor，move constructor 和 operator= 可以防止类的唯一实例被拷贝或移动；不暴露单例类的 constructor 和 destructor 可以保证单例类不会通过其他途径被实例化，同时将两者定义为 protected 可以让其被子类继承并使用。

## Lazy Singleton

Lazy Singleton 是一种比较传统的实现方法，通过其名字可以看出来它也具有 lazy-evaluation 的特点，但在实现的时候需要考虑线程安全的问题：

```cpp
template<typename T, bool is_thread_safe = true>
class LazySingleton
{
private:
    static unique_ptr<T> t_;
    static mutex mtx_;

public:
    static T& GetInstance()
    {
        if (is_thread_safe == false)
        {
            if (t_ == nullptr)
                t_ = unique_ptr<T>(new T);
            return *t_;
        }
        
        if (t_ == nullptr)
        {
            unique_lock<mutex> unique_locker(mtx_);
            if (t_ == nullptr)
                t_ = unique_ptr<T>(new T);
            return *t_;
        }

    }

    LazySingleton(T&&) = delete;
    LazySingleton(const T&) = delete;
    void operator= (const T&) = delete;

protected:
    LazySingleton() = default;
    virtual ~LazySingleton() = default;
};

template<typename T, bool is_thread_safe>
unique_ptr<T> LazySingleton<T, is_thread_safe>::t_;

template<typename T, bool is_thread_safe>
mutex LazySingleton<T, is_thread_safe>::mtx_;
```

我们通过模板参数 `is_thread_safe` 来控制这个类是否是线程安全的，因为在某些场景下我们会希望每个线程拥有一个实例：

1. 当 `is_thread_safe == false`，即非线程安全时，我们在 `GetInstance` 函数中直接判断，初始化并返回单例对象；这里使用了 `unique_ptr` 防止线程销毁时发生内存泄漏，也可以在析构函数中销毁指针；
2. 当 `is_thread_safe == true` 时，我们通过 double-checked locking 来进行检查并加锁，防止单例类在每个线程上都被实例化。

## Eager Singleton

和 Lazy Singleton 相反，Eager Singleton 利用 static member variable 的特性，在程序进入 main 函数之前进行初始化，这样就绕开了线程安全的问题：

```cpp
template<typename T>
class EagerSingleton
{
private:
    static T* t_;

public:
    static T& GetInstance()
    {
        return *t_;
    }

    EagerSingleton(T&&) = delete;
    EagerSingleton(const T&) = delete;
    void operator= (const T&) = delete;

protected:
    EagerSingleton() = default;
    virtual ~EagerSingleton() = default;
};

template<typename T>
T* EagerSingleton<T>::t_ = new (std::nothrow) T;
```

但是它也有两个问题：

1. 即使单例对象不被使用，单例类对象也会进行初始化；
2. [static initialization order fiasco](https://isocpp.org/wiki/faq/ctors#static-init-order)，即 t_ 对象和 `GetInstance` 函数的初始化先后顺序是不固定的；

## Testing

将上面实现的四种 Singleton 分别继承下来作为 functor 传入线程对象进行测试：

```cpp
class Foo : public Singleton<Foo>
{
public:
    void operator() ()
    {
        cout << &GetInstance() << endl;
    }
};

class LazyFoo : public LazySingleton<LazyFoo, false>
{
public:
    void operator() ()
    {
        cout << &GetInstance() << endl;
    }
};

class ThreadSafeLazyFoo : public LazySingleton<ThreadSafeLazyFoo>
{
public:
    void operator() ()
    {
        cout << &GetInstance() << endl;
    }
};

class EagerFoo : public EagerSingleton<EagerFoo>
{
public:
    void operator() ()
    {
        cout << &GetInstance() << endl;
    }
};

void SingletonTest()
{
    thread t1((Foo()));
    thread t2((Foo()));
    t1.join();
    t2.join();
    this_thread::sleep_for(chrono::milliseconds(100));
    
    t1 = thread((LazyFoo()));
    t2 = thread((LazyFoo()));
    t1.join();
    t2.join();
    this_thread::sleep_for(chrono::milliseconds(100));
    
    t1 = thread((ThreadSafeLazyFoo()));
    t2 = thread((ThreadSafeLazyFoo()));
    t1.join();
    t2.join();
    this_thread::sleep_for(chrono::milliseconds(100));
    
    t1 = thread((EagerFoo()));
    t2 = thread((EagerFoo()));
    t1.join();
    t2.join();
}
```

输出结果为：

```bash
0x60d110
0x60d110
0x7f92380008c0
0x7f92300008c0
0x7f92300008e0
0x7f92300008e0
0x1132010
0x1132010
```

可以看到只有第二组非线程安全的 `LazySingleton` 在两个线程中输出的实例地址是不同的，其它的 Singleton 均是线程安全的。