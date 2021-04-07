---
title: "C++ 智能指针（1）：auto_ptr"
date: 2018-12-27T15:21:35+11:00
draft: false
categories: ["C++"]
---

# C++智能指针（1）：auto_ptr

## 分析

C++ 中经常会出现因为没有 delete 指针而造成的内存泄漏，例如有一个 Object 类

```c++
class Object {
public:
    Object() { std::cout << "Construct" << std::endl; }

    Object(const Object &other) { std::cout << "Copy" << std::endl; }

    Object(Object &&other) noexcept { std::cout << "Move" << std::endl; }

    ~Object() { std::cout << "Destruct" << std::endl; }

    void Print() { std::cout << "Print" << std::endl; }
};
```

创建一个指向 Object 类型的指针

```c++
int main() {
    Object *o = new Object();
    o->Print();
    return 0;
}
/*
output:
Construct
Print
*/
```

我们没有进行delete o的操作，导致o没有被正确地析构，造成了内存泄漏。作为对比，创建一个Obj类型的对象

```c++
int main() {
    Object *o1 = new Object();
    o1->Print();
    Object o2 = Object();
    o2.Print();
    return 0;
}
/*
output:
Construct
Print
Construct
Print
Destruct
*/
```

产生这样的结果是因为对象创建在栈（[stack](https://isocpp.org/blog/2015/09/stack-heap-pool-tony-bulldozer00-bd00-dasilva)）上，编译器会自动进行对象的创建和销毁，而指针是创建在堆（heap）上，需要手动进行创建和销毁。为了规避这样的问题，我们可以封装一个智能指针类，用类来管理指针，防止造成内存泄漏，并且尽可能的模仿指针的用法。

## 实现

根据auto_ptr的源码，能够大致实现 AutoPointer 类

```c++
template<typename T>
class AutoPointer {
public:
    explicit AutoPointer(T *t);

    ~AutoPointer();

    T &operator*();

    T *operator->();

    T *release();

    void reset(T *p);

    AutoPointer(AutoPointer<T> &other);

    AutoPointer<T> &operator=(AutoPointer<T> const &other);

private:
    T *pointer;
};

template<typename T>
AutoPointer<T>::AutoPointer(T *t) {
    std::cout << "AutoPointer " << this << " constructor called." << std::endl;
    this->pointer = t;
}

template<typename T>
AutoPointer<T>::~AutoPointer() {
    std::cout << "AutoPointer " << this << " destructor called." << std::endl;
    delete this->pointer;
}

template<typename T>
T &AutoPointer<T>::operator*() {
    return *this->pointer;
}

template<typename T>
T *AutoPointer<T>::operator->() {
    return this->pointer;
}

template<typename T>
T *AutoPointer<T>::release() {
    T *new_pointer = this->pointer;
    this->pointer = nullptr;
    return new_pointer;
}

template<typename T>
void AutoPointer<T>::reset(T *p) {
    if (this->pointer != p) {
        delete this->pointer;
        this->pointer = p;
    }
}

template<typename T>
AutoPointer<T>::AutoPointer(AutoPointer<T> &other) {
    std::cout << "AutoPointer " << this << " copy constructor called." << std::endl;
    this->pointer = other.release();
}

template<typename T>
AutoPointer<T> &AutoPointer<T>::operator=(AutoPointer<T> const &other) {
    std::cout << "AutoPointer " << this << " assignment operator called." << std::endl;
    if (this->pointer != other.pointer)
        this->reset(other.release());
    return *this;
}
```

- 构造函数直接将 AutoPointer 类的 pointer 指针指向传入的参数指针所指向的地址
- 拷贝构造函数先对参数对象的指针进行 release 操作，也就是将参数对象的私有成员 pointer 指针置为 nullptr 并返回其原本指向的地址，然后将自身的 pointer 指向这个地址
- 赋值操作符先判断传入的参数是否是当前的 AutoPointer 类对象本身，如果是的话直接返回 this 指针，否则先对参数对象的指针进行 release 操作，并 delete 掉当前对象的 pointer，再将 pointer 指向参数对象的 pointer 原本指向的地址，这样的实现有效地规避了[迷途指针](https://zh.wikipedia.org/wiki/%E8%BF%B7%E9%80%94%E6%8C%87%E9%92%88)（也称悬空指针或野指针）。

## 测试

创建单个 AutoPointer 类对象时能够正常使用。

```c++
int main() {
    Object *o = new Object();
    AutoPointer<Object> a1(o);
    (*a1).Print();
    a1->Print();
    return 0;
}
/*
output:
Construct
AutoPointer 0x7fe680c02ab0 constructor called.
Print
Print
AutoPointer 0x7fe680c02ab0 destructor called.
Destruct
*/
```

创建两个 AutoPointer 类对象时如果使用同一个 Object 指针进行初始化，那么在程序退出时 Object 对象会被两个 AutoPointer 类对象各析构一次，也就是说同一块地址会被 delete 两次，造成运行时报错。

```c++
int main() {
    Object *o = new Object();
    AutoPointer<Object> a1(o);
    AutoPointer<Object> a2(o);
    return 0;
}
/*
output:
Construct
AutoPointer 0x7ffee3fa0178 constructor called.
AutoPointer 0x7ffee3fa0170 constructor called.
AutoPointer 0x7ffee3fa0170 destructor called.
Destruct
AutoPointer 0x7ffee3fa0178 destructor called.
Destruct
cpp(9015,0x1197a25c0) malloc: *** error for object 0x7fe9dec02b40: pointer being freed was not allocated
cpp(9015,0x1197a25c0) malloc: *** set a breakpoint in malloc_error_break to debug
*/
```

使用拷贝构造函数将一个 AutoPointer 类对象 a1 拷贝给另一个 AutoPointer 类对象 a2 时，Object 指针 o 原本是属于 a1 的，在 a2 调用拷贝构造函数之后，a1 的 pointer 变成了空指针，而 s2 拥有了指针 o，造成了所有权转移。

```c++
int main() {
    Object *o = new Object();
    AutoPointer<Object> a1(o);
    AutoPointer<Object> a2(a1);
    return 0;
}
/*
output:
Construct
AutoPointer 0x7fd15bc02ab0 constructor called.
AutoPointer 0x7fd15bc02ab0 copy constructor called.
AutoPointer 0x7fd15bc02ab0 destructor called.
Destruct
AutoPointer 0x0 destructor called.
*/
```

使用赋值操作符也会有所有权转移的问题。

```c++
int main() {
    Object *o = new Object();
    AutoPointer<Object> a1(o);
    AutoPointer<Object> a2 = a1;
    return 0;
}
/*
output:
Construct
AutoPointer 0x7ff5d5402ab0 constructor called.
AutoPointer 0x7ff5d5402ab0 copy constructor called.
AutoPointer 0x7ff5d5402ab0 destructor called.
Destruct
AutoPointer 0x0 destructor called.
*/
```

## 总结

AutoPointer 有效地解决了野指针问题，但又会引入一些其他的问题，例如

1. 所有权转移
    - 将 AutoPointer 作为参数进行拷贝构造或赋值操作时造成所有权转移

2. 内存泄漏
    - 在析构函数中使用了delete进行指针的销毁，但如果以数组指针进行初始化 ```AutoPointer<int> s1(new int[10])``` 会因为没有销毁数组的其它元素而造成内存泄漏

## auto_ptr源码

```c++
template<class _Tp>
class _LIBCPP_TEMPLATE_VIS auto_ptr
{
private:
    _Tp* __ptr_;
public:
    typedef _Tp element_type;

    _LIBCPP_INLINE_VISIBILITY explicit auto_ptr(_Tp* __p = 0) throw() : __ptr_(__p) {}
    _LIBCPP_INLINE_VISIBILITY auto_ptr(auto_ptr& __p) throw() : __ptr_(__p.release()) {}
    template<class _Up> _LIBCPP_INLINE_VISIBILITY auto_ptr(auto_ptr<_Up>& __p) throw()
        : __ptr_(__p.release()) {}
    _LIBCPP_INLINE_VISIBILITY auto_ptr& operator=(auto_ptr& __p) throw()
        {reset(__p.release()); return *this;}
    template<class _Up> _LIBCPP_INLINE_VISIBILITY auto_ptr& operator=(auto_ptr<_Up>& __p) throw()
        {reset(__p.release()); return *this;}
    _LIBCPP_INLINE_VISIBILITY auto_ptr& operator=(auto_ptr_ref<_Tp> __p) throw()
        {reset(__p.__ptr_); return *this;}
    _LIBCPP_INLINE_VISIBILITY ~auto_ptr() throw() {delete __ptr_;}

    _LIBCPP_INLINE_VISIBILITY _Tp& operator*() const throw()
        {return *__ptr_;}
    _LIBCPP_INLINE_VISIBILITY _Tp* operator->() const throw() {return __ptr_;}
    _LIBCPP_INLINE_VISIBILITY _Tp* get() const throw() {return __ptr_;}
    _LIBCPP_INLINE_VISIBILITY _Tp* release() throw()
    {
        _Tp* __t = __ptr_;
        __ptr_ = 0;
        return __t;
    }
    _LIBCPP_INLINE_VISIBILITY void reset(_Tp* __p = 0) throw()
    {
        if (__ptr_ != __p)
            delete __ptr_;
        __ptr_ = __p;
    }

    _LIBCPP_INLINE_VISIBILITY auto_ptr(auto_ptr_ref<_Tp> __p) throw() : __ptr_(__p.__ptr_) {}
    template<class _Up> _LIBCPP_INLINE_VISIBILITY operator auto_ptr_ref<_Up>() throw()
        {auto_ptr_ref<_Up> __t; __t.__ptr_ = release(); return __t;}
    template<class _Up> _LIBCPP_INLINE_VISIBILITY operator auto_ptr<_Up>() throw()
        {return auto_ptr<_Up>(release());}
};
```
