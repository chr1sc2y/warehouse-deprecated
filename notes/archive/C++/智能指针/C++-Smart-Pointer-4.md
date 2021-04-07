---
title: "C++ 智能指针（4）：weak_ptr"
date: 2019-01-26T14:33:18+11:00
draft: true
tags: ["C++", "Pointer", "C++11"]
categories: ["C++ Pointer"]
---
## 分析

在使用SharedPointer的时候发生了交叉引用的问题，我们有以下几种方法可以解决

1. 手动释放或销毁引起交叉引用的对象
2. 设置对象的生命周期，使其自动销毁
3. 使用弱引用

前两种方法都不易控制且实现较为繁琐，需要考虑多方面的因素。我们可以考虑通过引入一个弱引用的辅助智能指针来解决。

### 弱引用

强引用是指对象的生命周期完全由引用控制。只要引用存在，那么对象也存在。在SharedPointer中，私有成员```T *pointer;```是对象，而SharedPointer的实例就是引用，只要引用存在，那么它的对象指针就不会被销毁。而弱引用不控制对象的生命周期，不能修改对象本身或是对象的引用计数，仅仅用于监控他所引用的对象是否存在，以及获取对象的管理权并为其生成一个强引用。

为了维护弱引用，我们需要引入一个新的弱引用计数，用于监控对象是否存在。我们可以将原有的强引用计数和新的弱引用计数封装为一个类，在SharedPointer和WeakPointer对象中使用这个类的指针进行计数
```









## weak_ptr源码
```
class _LIBCPP_TYPE_VIS __shared_weak_count
    : private __shared_count
{
    long __shared_weak_owners_;

public:
    _LIBCPP_INLINE_VISIBILITY
    explicit __shared_weak_count(long __refs = 0) _NOEXCEPT
        : __shared_count(__refs),
          __shared_weak_owners_(__refs) {}
protected:
    virtual ~__shared_weak_count();

public:
#if defined(_LIBCPP_BUILDING_MEMORY) && \
    defined(_LIBCPP_DEPRECATED_ABI_LEGACY_LIBRARY_DEFINITIONS_FOR_INLINE_FUNCTIONS)
    void __add_shared() _NOEXCEPT;
    void __add_weak() _NOEXCEPT;
    void __release_shared() _NOEXCEPT;
#else
    _LIBCPP_INLINE_VISIBILITY
    void __add_shared() _NOEXCEPT {
      __shared_count::__add_shared();
    }
    _LIBCPP_INLINE_VISIBILITY
    void __add_weak() _NOEXCEPT {
      __libcpp_atomic_refcount_increment(__shared_weak_owners_);
    }
    _LIBCPP_INLINE_VISIBILITY
    void __release_shared() _NOEXCEPT {
      if (__shared_count::__release_shared())
        __release_weak();
    }
#endif
    void __release_weak() _NOEXCEPT;
    _LIBCPP_INLINE_VISIBILITY
    long use_count() const _NOEXCEPT {return __shared_count::use_count();}
    __shared_weak_count* lock() _NOEXCEPT;

    // Define the function out only if we build static libc++ without RTTI.
    // Otherwise we may break clients who need to compile their projects with
    // -fno-rtti and yet link against a libc++.dylib compiled
    // without -fno-rtti.
#if !defined(_LIBCPP_NO_RTTI) || !defined(_LIBCPP_BUILD_STATIC)
    virtual const void* __get_deleter(const type_info&) const _NOEXCEPT;
#endif
private:
    virtual void __on_zero_shared_weak() _NOEXCEPT = 0;
};


template<class _Tp>
class _LIBCPP_TEMPLATE_VIS weak_ptr
{
public:
    typedef _Tp element_type;
private:
    element_type*        __ptr_;
    __shared_weak_count* __cntrl_;

public:
    _LIBCPP_INLINE_VISIBILITY
    _LIBCPP_CONSTEXPR weak_ptr() _NOEXCEPT;
    template<class _Yp> _LIBCPP_INLINE_VISIBILITY weak_ptr(shared_ptr<_Yp> const& __r,
                   typename enable_if<is_convertible<_Yp*, _Tp*>::value, __nat*>::type = 0)
                        _NOEXCEPT;
    _LIBCPP_INLINE_VISIBILITY
    weak_ptr(weak_ptr const& __r) _NOEXCEPT;
    template<class _Yp> _LIBCPP_INLINE_VISIBILITY weak_ptr(weak_ptr<_Yp> const& __r,
                   typename enable_if<is_convertible<_Yp*, _Tp*>::value, __nat*>::type = 0)
                         _NOEXCEPT;

#ifndef _LIBCPP_HAS_NO_RVALUE_REFERENCES
    _LIBCPP_INLINE_VISIBILITY
    weak_ptr(weak_ptr&& __r) _NOEXCEPT;
    template<class _Yp> _LIBCPP_INLINE_VISIBILITY weak_ptr(weak_ptr<_Yp>&& __r,
                   typename enable_if<is_convertible<_Yp*, _Tp*>::value, __nat*>::type = 0)
                         _NOEXCEPT;
#endif  // _LIBCPP_HAS_NO_RVALUE_REFERENCES
    ~weak_ptr();

    _LIBCPP_INLINE_VISIBILITY
    weak_ptr& operator=(weak_ptr const& __r) _NOEXCEPT;
    template<class _Yp>
        typename enable_if
        <
            is_convertible<_Yp*, element_type*>::value,
            weak_ptr&
        >::type
        _LIBCPP_INLINE_VISIBILITY
        operator=(weak_ptr<_Yp> const& __r) _NOEXCEPT;

#ifndef _LIBCPP_HAS_NO_RVALUE_REFERENCES

    _LIBCPP_INLINE_VISIBILITY
    weak_ptr& operator=(weak_ptr&& __r) _NOEXCEPT;
    template<class _Yp>
        typename enable_if
        <
            is_convertible<_Yp*, element_type*>::value,
            weak_ptr&
        >::type
        _LIBCPP_INLINE_VISIBILITY
        operator=(weak_ptr<_Yp>&& __r) _NOEXCEPT;

#endif  // _LIBCPP_HAS_NO_RVALUE_REFERENCES

    template<class _Yp>
        typename enable_if
        <
            is_convertible<_Yp*, element_type*>::value,
            weak_ptr&
        >::type
        _LIBCPP_INLINE_VISIBILITY
        operator=(shared_ptr<_Yp> const& __r) _NOEXCEPT;

    _LIBCPP_INLINE_VISIBILITY
    void swap(weak_ptr& __r) _NOEXCEPT;
    _LIBCPP_INLINE_VISIBILITY
    void reset() _NOEXCEPT;

    _LIBCPP_INLINE_VISIBILITY
    long use_count() const _NOEXCEPT
        {return __cntrl_ ? __cntrl_->use_count() : 0;}
    _LIBCPP_INLINE_VISIBILITY
    bool expired() const _NOEXCEPT
        {return __cntrl_ == 0 || __cntrl_->use_count() == 0;}
    shared_ptr<_Tp> lock() const _NOEXCEPT;
    template<class _Up>
        _LIBCPP_INLINE_VISIBILITY
        bool owner_before(const shared_ptr<_Up>& __r) const _NOEXCEPT
        {return __cntrl_ < __r.__cntrl_;}
    template<class _Up>
        _LIBCPP_INLINE_VISIBILITY
        bool owner_before(const weak_ptr<_Up>& __r) const _NOEXCEPT
        {return __cntrl_ < __r.__cntrl_;}

    template <class _Up> friend class _LIBCPP_TEMPLATE_VIS weak_ptr;
    template <class _Up> friend class _LIBCPP_TEMPLATE_VIS shared_ptr;
};
```