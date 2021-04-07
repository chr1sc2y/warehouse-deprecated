---
title: "C++ 智能指针（3）：shared_ptr"
date: 2019-01-25T17:47:38+11:00
draft: false
categories: ["C++"]
---

# C++智能指针（3）：shared_ptr

## 分析

UniquePointer对象只能绑定单个指针，要实现指针的自动管理和销毁需要引入计数器
```
private:
    int *counter;
    T *pointer;
    D *deleter;
```
计数器的主要作用是标识当前指针被几个智能指针对象所引用，在析构当前对象时，使其计数器自减1。如果计数器等于0，则表示已经没有其他的对象在使用当前指针，此时则可以销毁指针，计数器和删除器。
```
template<typename T, typename D>
void SharedPointer<T, D>::release() {
    if (pointer) {
        std::cout << "SharedPointer " << this << " counter remains " << *counter << std::endl;
        if (--(*counter) == 0) {
            std::cout << "SharedPointer " << this << " destructor called." << std::endl;
            (*deleter)(pointer);
            (*deleter)(counter);
            (*deleter)(deleter);
            pointer = nullptr;
            counter = nullptr;
            deleter = nullptr;
        }
    }
}
```
reset函数将指针设为other的指针
```
template<typename T, typename D>
void SharedPointer<T, D>::reset(const SharedPointer<T, D> &other) {
    pointer = other.pointer;
    counter = other.counter;
    deleter = other.deleter;
    if (pointer)
        ++(*counter);
}
```
析构函数可以直接调用release函数
```
template<typename T, typename D>
SharedPointer<T, D>::~SharedPointer() {
    release();
}
```
拷贝构造函数可以直接调用reset函数
```
template<typename T, typename D>
SharedPointer<T, D>::SharedPointer(const SharedPointer<T, D> &other) {
    std::cout << "SharedPointer " << this << " copy constructor called." << std::endl;
    reset(other);
}
```
使用赋值操作符时先调用release函数，再调用reset函数
```
template<typename T, typename D>
SharedPointer<T, D> &SharedPointer<T, D>::operator=(const SharedPointer<T, D> &other) {
    std::cout << "SharedPointer " << this << " assignment operator called." << std::endl;
    if (this != &other) {
        release();
        reset(other);
    }
    return *this;
}
```

## 实现

根据shared_ptr的源码，能够大致实现SharedPointer类
```
template<typename T, typename D>
class SharedPointer {
public:
    explicit SharedPointer(T *t = nullptr, D *d = nullptr);

    ~SharedPointer();

    T &operator*();

    T *operator->();

    void reset(const SharedPointer &other);

    void release();

    SharedPointer(const SharedPointer &other);

    SharedPointer &operator=(const SharedPointer &other);

private:

    int *counter;
    T *pointer;
    D *deleter;
};

template<typename T, typename D>
void SharedPointer<T, D>::reset(const SharedPointer<T, D> &other) {
    pointer = other.pointer;
    counter = other.counter;
    deleter = other.deleter;
    if (pointer)
        ++(*counter);
}

template<typename T, typename D>
void SharedPointer<T, D>::release() {
    if (pointer) {
        std::cout << "SharedPointer " << this << " counter remains " << *counter << std::endl;
        if (--(*counter) == 0) {
            std::cout << "SharedPointer " << this << " destructor called." << std::endl;
            (*deleter)(pointer);
            (*deleter)(counter);
            (*deleter)(deleter);
            pointer = nullptr;
            counter = nullptr;
            deleter = nullptr;
        }
    }
}

template<typename T, typename D>
SharedPointer<T, D>::SharedPointer(T *t, D *d): pointer(t), deleter(d) {
    if (pointer)
        counter = new int(1);
    else
        counter = nullptr;
    std::cout << "SharedPointer " << this << " constructor called." << std::endl;
}

template<typename T, typename D>
SharedPointer<T, D>::SharedPointer(const SharedPointer<T, D> &other) {
    std::cout << "SharedPointer " << this << " copy constructor called." << std::endl;
    reset(other);
}


template<typename T, typename D>
SharedPointer<T, D>::~SharedPointer() {
    release();
}

template<typename T, typename D>
T &SharedPointer<T, D>::operator*() {
    return *pointer;
}

template<typename T, typename D>
T *SharedPointer<T, D>::operator->() {
    return pointer;
}

template<typename T, typename D>
SharedPointer<T, D> &SharedPointer<T, D>::operator=(const SharedPointer<T, D> &other) {
    std::cout << "SharedPointer " << this << " assignment operator called." << std::endl;
    if (this != &other) {
        release();
        reset(other);
    }
    return *this;
}
```

## 测试

尝试使用拷贝构造函数和赋值操作符使多个SharedPointer对象使用同一个指针，以及使用reset函数清空智能指针对象的指针
```
int main() {
    Deleter *deleter = new Deleter();
    Obj *o = new Obj();
    SharedPointer<Obj, Deleter> s1(o, deleter);
    SharedPointer<Obj, Deleter> s2(s1);
    SharedPointer<Obj, Deleter> s3;
    s3 = s1;
    return 0;
}
/*
output:
Construct
SharedPointer 0x7ffeeebdda00 constructor called.
SharedPointer 0x7ffeeebdd9e8 copy constructor called.
SharedPointer 0x7ffeeebdd9d0 constructor called.
SharedPointer 0x7ffeeebdd9d0 assignment operator called.
SharedPointer 0x7ffeeebdd9d0 counter remains 3
SharedPointer 0x7ffeeebdd9e8 counter remains 2
SharedPointer 0x7ffeeebdda00 counter remains 1
SharedPointer 0x7ffeeebdda00 destructor called.
Destruct
*/
```

考虑如下一个类
```
class Object : public Obj {
public:
    SharedPointer<Object, Deleter> S;
};
```
创建两个Object类型的对象
```
int main() {
    SharedPointer<Object, Deleter> s1(new Object());
    SharedPointer<Object, Deleter> s2(new Object());
    s1->S = s2;
    s2->S = s1;
    return 0;
}
/*
output:
Construct
SharedPointer 0x7f88eac02ab0 constructor called.
SharedPointer 0x7ffee0bfaa20 constructor called.
Construct
SharedPointer 0x7f88eac02ae0 constructor called.
SharedPointer 0x7ffee0bfa9f8 constructor called.
SharedPointer 0x7f88eac02ab0 assignment operator called.
SharedPointer 0x7f88eac02ae0 assignment operator called.
SharedPointer 0x7ffee0bfa9f8 counter remains 2
SharedPointer 0x7ffee0bfaa20 counter remains 2
*/
```
两个Object类型的指针都包含了一个SharedPointer类型的智能指针对象，但这两个指针又依赖于SharedPointer对象去进行销毁，导致最后s1和s2的计数器都不能减为0，从而不能正确地销毁指针，导致了内存泄漏。这种现象叫做交叉引用。

## 总结

SharedPointer利用[Reference counting](https://en.wikipedia.org/wiki/Reference_counting)（计数引用）解决了多个对象使用同一个指针时自动销毁指针的问题，但又会发生交叉引用时不能够正确销毁指针的问题。

## shared_pointer源码
```
template<class _Tp>
class _LIBCPP_TEMPLATE_VIS shared_ptr
{
public:
    typedef _Tp element_type;

#if _LIBCPP_STD_VER > 14
    typedef weak_ptr<_Tp> weak_type;
#endif
private:
    element_type*      __ptr_;
    __shared_weak_count* __cntrl_;

    struct __nat {int __for_bool_;};
public:
    _LIBCPP_INLINE_VISIBILITY
    _LIBCPP_CONSTEXPR shared_ptr() _NOEXCEPT;
    _LIBCPP_INLINE_VISIBILITY
    _LIBCPP_CONSTEXPR shared_ptr(nullptr_t) _NOEXCEPT;
    template<class _Yp>
        explicit shared_ptr(_Yp* __p,
                            typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat());
    template<class _Yp, class _Dp>
        shared_ptr(_Yp* __p, _Dp __d,
                   typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat());
    template<class _Yp, class _Dp, class _Alloc>
        shared_ptr(_Yp* __p, _Dp __d, _Alloc __a,
                   typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat());
    template <class _Dp> shared_ptr(nullptr_t __p, _Dp __d);
    template <class _Dp, class _Alloc> shared_ptr(nullptr_t __p, _Dp __d, _Alloc __a);
    template<class _Yp> _LIBCPP_INLINE_VISIBILITY shared_ptr(const shared_ptr<_Yp>& __r, element_type* __p) _NOEXCEPT;
    _LIBCPP_INLINE_VISIBILITY
    shared_ptr(const shared_ptr& __r) _NOEXCEPT;
    template<class _Yp>
        _LIBCPP_INLINE_VISIBILITY
        shared_ptr(const shared_ptr<_Yp>& __r,
                   typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat())
                       _NOEXCEPT;
#ifndef _LIBCPP_HAS_NO_RVALUE_REFERENCES
    _LIBCPP_INLINE_VISIBILITY
    shared_ptr(shared_ptr&& __r) _NOEXCEPT;
    template<class _Yp> _LIBCPP_INLINE_VISIBILITY  shared_ptr(shared_ptr<_Yp>&& __r,
                   typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat())
                       _NOEXCEPT;
#endif  // _LIBCPP_HAS_NO_RVALUE_REFERENCES
    template<class _Yp> explicit shared_ptr(const weak_ptr<_Yp>& __r,
                   typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type= __nat());
#if _LIBCPP_STD_VER <= 14 || defined(_LIBCPP_ENABLE_CXX17_REMOVED_AUTO_PTR)
#ifndef _LIBCPP_HAS_NO_RVALUE_REFERENCES
    template<class _Yp>
        shared_ptr(auto_ptr<_Yp>&& __r,
                   typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat());
#else
    template<class _Yp>
        shared_ptr(auto_ptr<_Yp> __r,
                   typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat());
#endif
#endif
#ifndef _LIBCPP_HAS_NO_RVALUE_REFERENCES
    template <class _Yp, class _Dp>
        shared_ptr(unique_ptr<_Yp, _Dp>&&,
                   typename enable_if
                   <
                       !is_lvalue_reference<_Dp>::value &&
                       !is_array<_Yp>::value &&
                       is_convertible<typename unique_ptr<_Yp, _Dp>::pointer, element_type*>::value,
                       __nat
                   >::type = __nat());
    template <class _Yp, class _Dp>
        shared_ptr(unique_ptr<_Yp, _Dp>&&,
                   typename enable_if
                   <
                       is_lvalue_reference<_Dp>::value &&
                       !is_array<_Yp>::value &&
                       is_convertible<typename unique_ptr<_Yp, _Dp>::pointer, element_type*>::value,
                       __nat
                   >::type = __nat());
#else  // _LIBCPP_HAS_NO_RVALUE_REFERENCES
    template <class _Yp, class _Dp>
        shared_ptr(unique_ptr<_Yp, _Dp>,
                   typename enable_if
                   <
                       !is_lvalue_reference<_Dp>::value &&
                       !is_array<_Yp>::value &&
                       is_convertible<typename unique_ptr<_Yp, _Dp>::pointer, element_type*>::value,
                       __nat
                   >::type = __nat());
    template <class _Yp, class _Dp>
        shared_ptr(unique_ptr<_Yp, _Dp>,
                   typename enable_if
                   <
                       is_lvalue_reference<_Dp>::value &&
                       !is_array<_Yp>::value &&
                       is_convertible<typename unique_ptr<_Yp, _Dp>::pointer, element_type*>::value,
                       __nat
                   >::type = __nat());
#endif  // _LIBCPP_HAS_NO_RVALUE_REFERENCES

    ~shared_ptr();

    _LIBCPP_INLINE_VISIBILITY
    shared_ptr& operator=(const shared_ptr& __r) _NOEXCEPT;
    template<class _Yp>
        typename enable_if
        <
            is_convertible<_Yp*, element_type*>::value,
            shared_ptr&
        >::type
        _LIBCPP_INLINE_VISIBILITY
        operator=(const shared_ptr<_Yp>& __r) _NOEXCEPT;
#ifndef _LIBCPP_HAS_NO_RVALUE_REFERENCES
    _LIBCPP_INLINE_VISIBILITY
    shared_ptr& operator=(shared_ptr&& __r) _NOEXCEPT;
    template<class _Yp>
        typename enable_if
        <
            is_convertible<_Yp*, element_type*>::value,
            shared_ptr<_Tp>&
        >::type
        _LIBCPP_INLINE_VISIBILITY
        operator=(shared_ptr<_Yp>&& __r);
#if _LIBCPP_STD_VER <= 14 || defined(_LIBCPP_ENABLE_CXX17_REMOVED_AUTO_PTR)
    template<class _Yp>
        _LIBCPP_INLINE_VISIBILITY
        typename enable_if
        <
            !is_array<_Yp>::value &&
            is_convertible<_Yp*, element_type*>::value,
            shared_ptr
        >::type&
        operator=(auto_ptr<_Yp>&& __r);
#endif
#else  // _LIBCPP_HAS_NO_RVALUE_REFERENCES
#if _LIBCPP_STD_VER <= 14 || defined(_LIBCPP_ENABLE_CXX17_REMOVED_AUTO_PTR)
    template<class _Yp>
        _LIBCPP_INLINE_VISIBILITY
        typename enable_if
        <
            !is_array<_Yp>::value &&
            is_convertible<_Yp*, element_type*>::value,
            shared_ptr&
        >::type
        operator=(auto_ptr<_Yp> __r);
#endif
#endif
    template <class _Yp, class _Dp>
        typename enable_if
        <
            !is_array<_Yp>::value &&
            is_convertible<typename unique_ptr<_Yp, _Dp>::pointer, element_type*>::value,
            shared_ptr&
        >::type
#ifndef _LIBCPP_HAS_NO_RVALUE_REFERENCES
        _LIBCPP_INLINE_VISIBILITY
        operator=(unique_ptr<_Yp, _Dp>&& __r);
#else  // _LIBCPP_HAS_NO_RVALUE_REFERENCES
        _LIBCPP_INLINE_VISIBILITY
        operator=(unique_ptr<_Yp, _Dp> __r);
#endif

    _LIBCPP_INLINE_VISIBILITY
    void swap(shared_ptr& __r) _NOEXCEPT;
    _LIBCPP_INLINE_VISIBILITY
    void reset() _NOEXCEPT;
    template<class _Yp>
        typename enable_if
        <
            is_convertible<_Yp*, element_type*>::value,
            void
        >::type
        _LIBCPP_INLINE_VISIBILITY
        reset(_Yp* __p);
    template<class _Yp, class _Dp>
        typename enable_if
        <
            is_convertible<_Yp*, element_type*>::value,
            void
        >::type
        _LIBCPP_INLINE_VISIBILITY
        reset(_Yp* __p, _Dp __d);
    template<class _Yp, class _Dp, class _Alloc>
        typename enable_if
        <
            is_convertible<_Yp*, element_type*>::value,
            void
        >::type
        _LIBCPP_INLINE_VISIBILITY
        reset(_Yp* __p, _Dp __d, _Alloc __a);

    _LIBCPP_INLINE_VISIBILITY
    element_type* get() const _NOEXCEPT {return __ptr_;}
    _LIBCPP_INLINE_VISIBILITY
    typename add_lvalue_reference<element_type>::type operator*() const _NOEXCEPT
        {return *__ptr_;}
    _LIBCPP_INLINE_VISIBILITY
    element_type* operator->() const _NOEXCEPT {return __ptr_;}
    _LIBCPP_INLINE_VISIBILITY
    long use_count() const _NOEXCEPT {return __cntrl_ ? __cntrl_->use_count() : 0;}
    _LIBCPP_INLINE_VISIBILITY
    bool unique() const _NOEXCEPT {return use_count() == 1;}
    _LIBCPP_INLINE_VISIBILITY
    _LIBCPP_EXPLICIT operator bool() const _NOEXCEPT {return get() != 0;}
    template <class _Up>
        _LIBCPP_INLINE_VISIBILITY
        bool owner_before(shared_ptr<_Up> const& __p) const _NOEXCEPT
        {return __cntrl_ < __p.__cntrl_;}
    template <class _Up>
        _LIBCPP_INLINE_VISIBILITY
        bool owner_before(weak_ptr<_Up> const& __p) const _NOEXCEPT
        {return __cntrl_ < __p.__cntrl_;}
    _LIBCPP_INLINE_VISIBILITY
    bool
    __owner_equivalent(const shared_ptr& __p) const
        {return __cntrl_ == __p.__cntrl_;}

#ifndef _LIBCPP_NO_RTTI
    template <class _Dp>
        _LIBCPP_INLINE_VISIBILITY
        _Dp* __get_deleter() const _NOEXCEPT
            {return static_cast<_Dp*>(__cntrl_
                    ? const_cast<void *>(__cntrl_->__get_deleter(typeid(_Dp)))
                      : nullptr);}
#endif  // _LIBCPP_NO_RTTI

#ifndef _LIBCPP_HAS_NO_VARIADICS

    template<class ..._Args>
        static
        shared_ptr<_Tp>
        make_shared(_Args&& ...__args);

    template<class _Alloc, class ..._Args>
        static
        shared_ptr<_Tp>
        allocate_shared(const _Alloc& __a, _Args&& ...__args);

#else  // _LIBCPP_HAS_NO_VARIADICS

    static shared_ptr<_Tp> make_shared();

    template<class _A0>
        static shared_ptr<_Tp> make_shared(_A0&);

    template<class _A0, class _A1>
        static shared_ptr<_Tp> make_shared(_A0&, _A1&);

    template<class _A0, class _A1, class _A2>
        static shared_ptr<_Tp> make_shared(_A0&, _A1&, _A2&);

    template<class _Alloc>
        static shared_ptr<_Tp>
        allocate_shared(const _Alloc& __a);

    template<class _Alloc, class _A0>
        static shared_ptr<_Tp>
        allocate_shared(const _Alloc& __a, _A0& __a0);

    template<class _Alloc, class _A0, class _A1>
        static shared_ptr<_Tp>
        allocate_shared(const _Alloc& __a, _A0& __a0, _A1& __a1);

    template<class _Alloc, class _A0, class _A1, class _A2>
        static shared_ptr<_Tp>
        allocate_shared(const _Alloc& __a, _A0& __a0, _A1& __a1, _A2& __a2);

#endif  // _LIBCPP_HAS_NO_VARIADICS

private:
    template <class _Yp, bool = is_function<_Yp>::value>
        struct __shared_ptr_default_allocator
        {
            typedef allocator<_Yp> type;
        };

    template <class _Yp>
        struct __shared_ptr_default_allocator<_Yp, true>
        {
            typedef allocator<__shared_ptr_dummy_rebind_allocator_type> type;
        };

    template <class _Yp, class _OrigPtr>
        _LIBCPP_INLINE_VISIBILITY
        typename enable_if<is_convertible<_OrigPtr*,
                                          const enable_shared_from_this<_Yp>*
        >::value,
            void>::type
        __enable_weak_this(const enable_shared_from_this<_Yp>* __e,
                           _OrigPtr* __ptr) _NOEXCEPT
        {
            typedef typename remove_cv<_Yp>::type _RawYp;
            if (__e && __e->__weak_this_.expired())
            {
                __e->__weak_this_ = shared_ptr<_RawYp>(*this,
                    const_cast<_RawYp*>(static_cast<const _Yp*>(__ptr)));
            }
        }

    _LIBCPP_INLINE_VISIBILITY void __enable_weak_this(...) _NOEXCEPT {}

    template <class _Up> friend class _LIBCPP_TEMPLATE_VIS shared_ptr;
    template <class _Up> friend class _LIBCPP_TEMPLATE_VIS weak_ptr;
};
```