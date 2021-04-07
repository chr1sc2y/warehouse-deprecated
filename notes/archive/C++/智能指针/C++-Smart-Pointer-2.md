---
title: "C++ 智能指针（2）：unique_ptr"
date: 2019-01-19T01:02:02+11:00
draft: false
categories: ["C++"]
---

# C++智能指针（2）：unique_ptr

## 分析

在使用 AutoPointer 的时候会发生所有权转移和内存泄漏的问题，所以我们可以对 AutoPointer 类稍加修改，修复这两个问题。

### 所有权转移

为了规避可能发生所有权转移的情况，我们可以直接禁止它使用拷贝构造函数和赋值操作符。

```c++
    UniquePointer(UniquePointer<T> &other) = delete;

    UniquePointer<T> &operator=(const UniquePointer<T> &other) = delete;
```

但很多时候我们都需要使用到传递指针的操作，如果只是使用 deleted 函数禁止拷贝构造函数和赋值操作符，那么这个智能指针存在的意义就不大了，我们可以通过 move 语义来实现移动构造函数和移动赋值操作符，从而在使用 UniquePointer 的时候可以在特定情况下进行所有权转移。

```c++
    UniquePointer(UniquePointer<T> &&other) noexcept;

    UniquePointer &operator=(UniquePointer &&other) noexcept;
```

### 内存泄漏

为了防止发生内存泄漏，我们可以在UniquePointer的私有成员中增加一个删除器，并根据当前指针对象的类型指定删除器，从而防止发生内存泄漏。

```c++
class Deleter {
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete p;
    }
};

template<typename T, typename D>
class UniquePointer {
  ...
private:
    T *pointer;
    Deleter deleter;
};
```

## 实现

根据unique_ptr的源码，能够大致实现UniquePointer类

```c++
template<typename T, typename D>
class UniquePointer {
public:
    explicit UniquePointer(T *t, const D &d);

    ~UniquePointer();

    T &operator*();

    T *operator->();

    T *release();

    void reset(T *p);

    UniquePointer(UniquePointer &&other) noexcept;

    UniquePointer &operator=(UniquePointer &&other) noexcept;

    UniquePointer(const UniquePointer &other) = delete;

    UniquePointer &operator=(const UniquePointer &other) = delete;

private:
    T *pointer;
    D deleter;
};

template<typename T, typename D>
UniquePointer<T, D>::UniquePointer(T *t, const D &d) {
    std::cout << "UniquePointer " << this << " constructor called." << std::endl;
    this->pointer = t;
    this->deleter = d;
}

template<typename T, typename D>
UniquePointer<T, D>::~UniquePointer() {
    std::cout << "UniquePointer " << this << " destructor called." << std::endl;
    deleter(this->pointer);
}

template<typename T, typename D>
T &UniquePointer<T, D>::operator*() {
    return *this->pointer;
}

template<typename T, typename D>
T *UniquePointer<T, D>::operator->() {
    return this->pointer;
}

template<typename T, typename D>
T *UniquePointer<T, D>::release() {
    T *new_pointer = this->pointer;
    this->pointer = nullptr;
    return new_pointer;
}

template<typename T, typename D>
void UniquePointer<T, D>::reset(T *p) {
    if (this->pointer != p) {
        deleter(this->pointer);
        this->pointer = p;
    }
}

template<typename T, typename D>
UniquePointer<T, D>::UniquePointer(UniquePointer<T, D> &&other) noexcept {
    std::cout << "UniquePointer " << this << " move constructor called." << std::endl;
    this->pointer = other.release();
    deleter(std::move(other.deleter));
}

template<typename T, typename D>
UniquePointer<T, D> &UniquePointer<T, D>::operator=(UniquePointer<T, D> &&other) noexcept {
    std::cout << "UniquePointer " << this << " assignment operator called." << std::endl;
    if (this->pointer != other.pointer) {
        reset(other.release());
        deleter = std::move(other.deleter);
    }
    return *this;
}
```

## 测试

尝试使用移动构造函数

```c++
class Deleter {
public:
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete p;
    }
};
```

```c++
int main() {
    Deleter deleter;
    Obj *o = new Obj();
    UniquePointer<Obj, Deleter> u1(o, deleter);
    UniquePointer<Obj, Deleter> u2(move(u1));
    return 0;
}
/*
output:
Construct
UniquePointer 0x7ffee7dada08 constructor called.
UniquePointer 0x7ffee7dad9f8 move constructor called.
UniquePointer 0x7ffee7dad9f8 destructor called.
Destruct
UniquePointer 0x7ffee7dada08 destructor called.
*/
```

尝试使用移动赋值操作符
```
class Deleter {
public:
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete p;
    }
};

int main() {
    Deleter deleter;
    Obj *o = new Obj();
    UniquePointer<Obj, Deleter> u1(o, deleter);
    UniquePointer<Obj, Deleter> u2(nullptr, deleter);
    u2 = move(u1);
    return 0;
}
/*
output:
Construct
UniquePointer 0x7ffee915da08 constructor called.
UniquePointer 0x7ffee915d9f8 constructor called.
UniquePointer 0x7ffee915d9f8 assignment operator called.
UniquePointer 0x7ffee915d9f8 destructor called.
Destruct
UniquePointer 0x7ffee915da08 destructor called.
*/
```

定义一个数组删除器，尝试以数组指针初始化UniquePointer类对象
```
class ArrayDeleter {
public:
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete[] p;
    }
};

int main() {
    ArrayDeleter array_deleter;
    Obj *o = new Obj[3];
    UniquePointer<Obj, ArrayDeleter> u(o, array_deleter);
    return 0;
}
/*
output:
Construct
Construct
Construct
UniquePointer 0x7ffeed926a08 constructor called.
UniquePointer 0x7ffeed926a08 destructor called.
Destruct
Destruct
Destruct
*/
```
作为对比，如果使用默认删除器作为数组指针的删除器
```
class Deleter {
public:
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete p;
    }
};

int main() {
    Deleter deleter;
    Obj *o = new Obj[3];
    UniquePointer<Obj, Deleter> u(o, deleter);
    return 0;
}
/*
output:
Construct
Construct
Construct
UniquePointer 0x7ffee8f85a10 constructor called.
UniquePointer 0x7ffee8f85a10 destructor called.
Destruct
*/
```
说明删除器能够正确地修复内存泄漏的问题。


尝试将两个UniquePointer的对象指向同一个指针
```
int main() {
    Deleter deleter;
    Obj *o = new Obj();
    UniquePointer<Obj, Deleter> u1(o, deleter);
    UniquePointer<Obj, Deleter> u2(o, deleter);
    return 0;
}
/*
output:
(19576,0x10e00a5c0) malloc: *** error for object 0x7fcfe8c02ab0: pointer being freed was not allocated
(19576,0x10e00a5c0) malloc: *** set a breakpoint in malloc_error_break to debug
Construct
UniquePointer 0x7ffee28a9a10 constructor called.
UniquePointer 0x7ffee28a9a00 constructor called.
UniquePointer 0x7ffee28a9a00 destructor called.
Destruct
UniquePointer 0x7ffee28a9a10 destructor called.
Destruct
*/
```
还是产生调用两次析构函数的错误。

## 总结

UniquePointer成功地解决了所有权转移和内存泄漏的问题，但还有诸如重复析构的问题存在。


## unique_ptr源码

```
template <class _Tp, class _Dp = default_delete<_Tp> >
class _LIBCPP_TEMPLATE_VIS unique_ptr {
public:
  typedef _Tp element_type;
  typedef _Dp deleter_type;
  typedef typename __pointer_type<_Tp, deleter_type>::type pointer;

  static_assert(!is_rvalue_reference<deleter_type>::value,
                "the specified deleter type cannot be an rvalue reference");

private:
  __compressed_pair<pointer, deleter_type> __ptr_;

  struct __nat { int __for_bool_; };

#ifndef _LIBCPP_CXX03_LANG
  typedef __unique_ptr_deleter_sfinae<_Dp> _DeleterSFINAE;

  template <bool _Dummy>
  using _LValRefType =
      typename __dependent_type<_DeleterSFINAE, _Dummy>::__lval_ref_type;

  template <bool _Dummy>
  using _GoodRValRefType =
      typename __dependent_type<_DeleterSFINAE, _Dummy>::__good_rval_ref_type;

  template <bool _Dummy>
  using _BadRValRefType =
      typename __dependent_type<_DeleterSFINAE, _Dummy>::__bad_rval_ref_type;

  template <bool _Dummy, class _Deleter = typename __dependent_type<
                             __identity<deleter_type>, _Dummy>::type>
  using _EnableIfDeleterDefaultConstructible =
      typename enable_if<is_default_constructible<_Deleter>::value &&
                         !is_pointer<_Deleter>::value>::type;

  template <class _ArgType>
  using _EnableIfDeleterConstructible =
      typename enable_if<is_constructible<deleter_type, _ArgType>::value>::type;

  template <class _UPtr, class _Up>
  using _EnableIfMoveConvertible = typename enable_if<
      is_convertible<typename _UPtr::pointer, pointer>::value &&
      !is_array<_Up>::value
  >::type;

  template <class _UDel>
  using _EnableIfDeleterConvertible = typename enable_if<
      (is_reference<_Dp>::value && is_same<_Dp, _UDel>::value) ||
      (!is_reference<_Dp>::value && is_convertible<_UDel, _Dp>::value)
    >::type;

  template <class _UDel>
  using _EnableIfDeleterAssignable = typename enable_if<
      is_assignable<_Dp&, _UDel&&>::value
    >::type;

public:
  template <bool _Dummy = true,
            class = _EnableIfDeleterDefaultConstructible<_Dummy>>
  _LIBCPP_INLINE_VISIBILITY
  constexpr unique_ptr() noexcept : __ptr_(pointer()) {}

  template <bool _Dummy = true,
            class = _EnableIfDeleterDefaultConstructible<_Dummy>>
  _LIBCPP_INLINE_VISIBILITY
  constexpr unique_ptr(nullptr_t) noexcept : __ptr_(pointer()) {}

  template <bool _Dummy = true,
            class = _EnableIfDeleterDefaultConstructible<_Dummy>>
  _LIBCPP_INLINE_VISIBILITY
  explicit unique_ptr(pointer __p) noexcept : __ptr_(__p) {}

  template <bool _Dummy = true,
            class = _EnableIfDeleterConstructible<_LValRefType<_Dummy>>>
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(pointer __p, _LValRefType<_Dummy> __d) noexcept
      : __ptr_(__p, __d) {}

  template <bool _Dummy = true,
            class = _EnableIfDeleterConstructible<_GoodRValRefType<_Dummy>>>
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(pointer __p, _GoodRValRefType<_Dummy> __d) noexcept
      : __ptr_(__p, _VSTD::move(__d)) {
    static_assert(!is_reference<deleter_type>::value,
                  "rvalue deleter bound to reference");
  }

  template <bool _Dummy = true,
            class = _EnableIfDeleterConstructible<_BadRValRefType<_Dummy>>>
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(pointer __p, _BadRValRefType<_Dummy> __d) = delete;

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(unique_ptr&& __u) noexcept
      : __ptr_(__u.release(), _VSTD::forward<deleter_type>(__u.get_deleter())) {
  }

  template <class _Up, class _Ep,
      class = _EnableIfMoveConvertible<unique_ptr<_Up, _Ep>, _Up>,
      class = _EnableIfDeleterConvertible<_Ep>
  >
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(unique_ptr<_Up, _Ep>&& __u) _NOEXCEPT
      : __ptr_(__u.release(), _VSTD::forward<_Ep>(__u.get_deleter())) {}

#if _LIBCPP_STD_VER <= 14 || defined(_LIBCPP_ENABLE_CXX17_REMOVED_AUTO_PTR)
  template <class _Up>
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(auto_ptr<_Up>&& __p,
             typename enable_if<is_convertible<_Up*, _Tp*>::value &&
                                    is_same<_Dp, default_delete<_Tp>>::value,
                                __nat>::type = __nat()) _NOEXCEPT
      : __ptr_(__p.release()) {}
#endif

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr& operator=(unique_ptr&& __u) _NOEXCEPT {
    reset(__u.release());
    __ptr_.second() = _VSTD::forward<deleter_type>(__u.get_deleter());
    return *this;
  }

  template <class _Up, class _Ep,
      class = _EnableIfMoveConvertible<unique_ptr<_Up, _Ep>, _Up>,
      class = _EnableIfDeleterAssignable<_Ep>
  >
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr& operator=(unique_ptr<_Up, _Ep>&& __u) _NOEXCEPT {
    reset(__u.release());
    __ptr_.second() = _VSTD::forward<_Ep>(__u.get_deleter());
    return *this;
  }

#else  // _LIBCPP_CXX03_LANG
private:
  unique_ptr(unique_ptr&);
  template <class _Up, class _Ep> unique_ptr(unique_ptr<_Up, _Ep>&);

  unique_ptr& operator=(unique_ptr&);
  template <class _Up, class _Ep> unique_ptr& operator=(unique_ptr<_Up, _Ep>&);

public:
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr() : __ptr_(pointer())
  {
    static_assert(!is_pointer<deleter_type>::value,
                  "unique_ptr constructed with null function pointer deleter");
    static_assert(is_default_constructible<deleter_type>::value,
                  "unique_ptr::deleter_type is not default constructible");
  }
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(nullptr_t) : __ptr_(pointer())
  {
    static_assert(!is_pointer<deleter_type>::value,
                  "unique_ptr constructed with null function pointer deleter");
  }
  _LIBCPP_INLINE_VISIBILITY
  explicit unique_ptr(pointer __p)
      : __ptr_(_VSTD::move(__p)) {
    static_assert(!is_pointer<deleter_type>::value,
                  "unique_ptr constructed with null function pointer deleter");
  }

  _LIBCPP_INLINE_VISIBILITY
  operator __rv<unique_ptr>() {
    return __rv<unique_ptr>(*this);
  }

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(__rv<unique_ptr> __u)
      : __ptr_(__u->release(),
               _VSTD::forward<deleter_type>(__u->get_deleter())) {}

  template <class _Up, class _Ep>
  _LIBCPP_INLINE_VISIBILITY
  typename enable_if<
      !is_array<_Up>::value &&
          is_convertible<typename unique_ptr<_Up, _Ep>::pointer,
                         pointer>::value &&
          is_assignable<deleter_type&, _Ep&>::value,
      unique_ptr&>::type
  operator=(unique_ptr<_Up, _Ep> __u) {
    reset(__u.release());
    __ptr_.second() = _VSTD::forward<_Ep>(__u.get_deleter());
    return *this;
  }

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(pointer __p, deleter_type __d)
      : __ptr_(_VSTD::move(__p), _VSTD::move(__d)) {}
#endif // _LIBCPP_CXX03_LANG

#if _LIBCPP_STD_VER <= 14 || defined(_LIBCPP_ENABLE_CXX17_REMOVED_AUTO_PTR)
  template <class _Up>
  _LIBCPP_INLINE_VISIBILITY
      typename enable_if<is_convertible<_Up*, _Tp*>::value &&
                             is_same<_Dp, default_delete<_Tp> >::value,
                         unique_ptr&>::type
      operator=(auto_ptr<_Up> __p) {
    reset(__p.release());
    return *this;
  }
#endif

  _LIBCPP_INLINE_VISIBILITY
  ~unique_ptr() { reset(); }

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr& operator=(nullptr_t) _NOEXCEPT {
    reset();
    return *this;
  }

  _LIBCPP_INLINE_VISIBILITY
  typename add_lvalue_reference<_Tp>::type
  operator*() const {
    return *__ptr_.first();
  }
  _LIBCPP_INLINE_VISIBILITY
  pointer operator->() const _NOEXCEPT {
    return __ptr_.first();
  }
  _LIBCPP_INLINE_VISIBILITY
  pointer get() const _NOEXCEPT {
    return __ptr_.first();
  }
  _LIBCPP_INLINE_VISIBILITY
  deleter_type& get_deleter() _NOEXCEPT {
    return __ptr_.second();
  }
  _LIBCPP_INLINE_VISIBILITY
  const deleter_type& get_deleter() const _NOEXCEPT {
    return __ptr_.second();
  }
  _LIBCPP_INLINE_VISIBILITY
  _LIBCPP_EXPLICIT operator bool() const _NOEXCEPT {
    return __ptr_.first() != nullptr;
  }

  _LIBCPP_INLINE_VISIBILITY
  pointer release() _NOEXCEPT {
    pointer __t = __ptr_.first();
    __ptr_.first() = pointer();
    return __t;
  }

  _LIBCPP_INLINE_VISIBILITY
  void reset(pointer __p = pointer()) _NOEXCEPT {
    pointer __tmp = __ptr_.first();
    __ptr_.first() = __p;
    if (__tmp)
      __ptr_.second()(__tmp);
  }

  _LIBCPP_INLINE_VISIBILITY
  void swap(unique_ptr& __u) _NOEXCEPT {
    __ptr_.swap(__u.__ptr_);
  }
};
```