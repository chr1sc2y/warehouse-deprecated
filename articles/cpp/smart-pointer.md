# C++ 智能指针的简单实现

[TOC]

## 1 std::auto_ptr

C++ 中经常会出现因为没有 delete 指针而造成的内存泄漏，例如有一个 Object 模板类：

```cpp
template<typename T>
class Object
{
public:
    // constructor
    Object() : t_() { cout << "Object::Constructor " << this << endl; }
    Object(T t) : t_(t) { cout << "Object::Constructor " << this << endl; }

    // copy-ctor
    Object(const Object &other) { cout << "Object::Copy-ctor " << this << endl; }

    // destructor
    ~Object() { cout << "Object::Destructor " << this << endl; }

    void Set(T t) { t_ = t; }

    void Print() { cout << t_ << endl; }

private:
    T t_;
};
```



如果在堆上为类对象分配了内存，在离开其作用域的时候又没将其释放，则会造成内存泄漏：



```cpp
void AutoPointerFoo()
{
    Object<int>* o = new Object<int>(1);
    o->Print();
}
```



```shell
$ ./bin/smart-pointer 
Object::Constructor 0x7b7058 # o
1
```



为了解决这个问题，C++ 98 在标准中增加了最原始的[智能指针](https://en.wikipedia.org/wiki/Smart_pointer) `std::auto_ptr`，它利用 [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization) 的机制提供了自动内存管理的功能，即利用栈上对象来管理堆上内存，当智能指针对象离开其作用域时，默认在其析构函数中释放其管理的堆上变量；它能够在一定程度上减少内存泄露的发生，以下是参考 GCC 中的 `std::auto_ptr` 实现的 `AutoPointer` 类，做了一定程度的简化，增加了一些输出方便追踪资源分配过程：



```cpp
template<typename T>
class AutoPointer
{
public:
    // constructor
    explicit AutoPointer(T* t = nullptr) noexcept : ptr_(t) { std::cout << "AutoPointer::Constructor " << this << std::endl; }

    // copy-ctor
    AutoPointer(AutoPointer<T>& other) noexcept : ptr_(other.Release()) { std::cout << "AutoPointer::Copyctor " << this << std::endl; }

    // assignment operator
    AutoPointer<T>& operator=(AutoPointer<T>& other)
    {
        std::cout << "AutoPointer::Assignment " << this << std::endl;
        Reset(other.Release());
        return *this;
    }

    // destructor
    ~AutoPointer() noexcept
    {
        std::cout << "AutoPointer::Destructor " << this << std::endl;
        delete ptr_;
    }

    T& operator*() noexcept { return *ptr_; }

    T* operator->() const noexcept { return ptr_; }

    T* Get() const noexcept { return ptr_; }

    T* Release() noexcept
    {
        T* ptr_ret = ptr_;
        ptr_ = nullptr;
        return ptr_ret;
    }

    void Reset(T* ptr_para) noexcept
    {
        if (ptr_ != ptr_para)
        {
            delete ptr_;
            ptr_ = ptr_para;
        }
    }

private:
    T *ptr_;
};
```



在初始化时，我们需要手动在堆上分配一个对象，并将其作为参数传入；接下来就可以将智能指针对象当作普通的指针使用了，同时也并不需要关心其生命周期，并能够用使用普通指针的方法来使用智能指针：



```cpp
void AutoPointerFoo()
{
    Object<int>* o = new Object<int>(1);
    AutoPointer<Object<int>> a(o);
    (*o).Set(2);
    (*o).Print();
    o->Set(3);
    o->Print();
}
```



```shell
$ ./bin/smart-pointer 
Object::Constructor 0x48f058 # o
AutoPointer::Constructor 0xbee47668 # a
2
3
AutoPointer::Destructor 0xbee47668 # a
Object::Destructor 0x48f058 # o
```



类中最重要的两个函数是 `Release` 和 `Reset`，前者用来解除对象当前所管理的指针对象并返回，后者会释放对象当前所管理的指针对象，并将传入的指针对象置为新的管理对象，两者搭配起来实现了拷贝构造函数和赋值操作符；而这两个函数的存在则带来了第一个问题，即在进行拷贝构造或者赋值操作的时候，被操作的 `AutoPointer` 对象可能在无意识的情况下失去对其自身所管理对象的所有权，从而可能造成 `segmentation fault`：



```cpp
void Foo()
{
    AutoPointer<Object<int>> p1(new Object<int>(6));
    AutoPointer<Object<int>> p2(p1);
    cout << "p2: "; p2->Print();
    cout << "p1: "; p1->Print();
}
```



```shell
$ ./bin/smart-pointer 
Object::Constructor 0x1cbd058 # o
AutoPointer::Constructor 0xbed23668 # a1
AutoPointer::Copyctor 0xbed23664 # a2
a2: 1
Segmentation fault
```



第二个问题是 `AutoPointer` 默认只会使用 `delete` 来进行删除操作，如果一个 `AutoPointer` 对象管理了一个数组，则会在离开其作用域时发生内存泄漏，开启 AddressSanitizer 可以检查到：



```cpp
void AutoPointerFoo()
{
    int *a = new int[1000000];
    AutoPointer<int> p(a);
}
```



```shell
$ ./bin/smart-pointer 
AutoPointer::Constructor 0xbe9955e0 # new[]
AutoPointer::Destructor 0xbe9955e0 # delete
=================================================================
==2543==ERROR: AddressSanitizer: alloc-dealloc-mismatch (operator new [] vs operator delete) on 0xb412e800
# ...
```



除此之外，如果使用同一个 `Object` 指针对多个 `AutoPointer` 对象进行初始化，那么这个 `Object` 对象会被多次 `delete`，在运行时造成 `double free` 的报错：



```cpp
void AutoPointerFoo()
{
    Object<int>* o = new Object<int>(1);
    AutoPointer<Object<int>> a1(o);
    AutoPointer<Object<int>> a2(o);
}
```



```shell
$ ./bin/smart-pointer 
Object::Constructor 0x9c3058 # o
AutoPointer::Constructor 0xbee80668 # a1
AutoPointer::Constructor 0xbee80664 # a2
AutoPointer::Destructor 0xbee80664 # a2
Object::Destructor 0x9c3058 # o
AutoPointer::Destructor 0xbee80668 # a1
Object::Destructor 0x9c3058 # o
free(): double free detected in tcache 2
Aborted
```



## 2 unique_ptr

为了解决 `std::auto_ptr` 中出现的问题，C++ 11 参考了 `boost::unique_ptr` 的设计，向标准库中引入了 `std::unique_ptr`，下面是参考其实现的 `UniquePointer` 模板类，做了相当程度的简化：

```cpp
template<typename ElementType, typename DeleterType = DefaultDeleter>
class UniquePointer
{
public:
    // constructors
    UniquePointer() noexcept : ptr_(nullptr) { std::cout << "UniquePointer::Constructor " << this << std::endl; }
    explicit UniquePointer(ElementType* p) noexcept : ptr_(p) { std::cout << "UniquePointer::Constructor " << this << std::endl; }
    UniquePointer(ElementType* p, DeleterType d) noexcept : ptr_(p), deleter_(d) { std::cout << "UniquePointer::Constructor " << this << std::endl; }

    // move-ctor
    UniquePointer(UniquePointer<ElementType, DeleterType>&& other) noexcept : ptr_(other.Release()), deleter_(std::move(other.deleter_)) { std::cout << "UniquePointer::Move-ctor " << this << std::endl; }
    // move assignment operator
    UniquePointer<ElementType, DeleterType>& operator=(UniquePointer<ElementType, DeleterType>&& other) noexcept
    {
        std::cout << "UniquePointer::MoveAssignment " << this << std::endl;
        ptr_ = other.Release();
        deleter_ = std::move(other.deleter_);
        return *this;
    }

    // copy-ctor
    UniquePointer(UniquePointer<ElementType, DeleterType>& other) noexcept = delete;
    // assignment operator
    UniquePointer<ElementType, DeleterType>& operator=(UniquePointer<ElementType, DeleterType>& other) = delete;

    // destructor
    ~UniquePointer() noexcept
    {
        std::cout << "UniquePointer::Destructor " << this << std::endl;
        if (ptr_)
        {
            GetDeleter()(ptr_);
            ptr_ = nullptr;
        }
    }

    ElementType& operator*() noexcept { return *ptr_; }

    ElementType* operator->() const noexcept { return ptr_; }

    ElementType* Get() const noexcept { return ptr_; }

    const DeleterType& GetDeleter() const noexcept { return deleter_; }

    ElementType* Release() noexcept
    {
        ElementType* ret = nullptr;
        std::swap(ptr_, ret);
        return ret;
    }

    void Reset(ElementType* p) noexcept
    {
        if (ptr_ != p)
        {
            delete ptr_;
            ptr_ = p;
        }
    }

private:
    ElementType* ptr_;
    DeleterType deleter_;
};
```

相较于 `AutoPointer`，`UniquePointer` 做出的改变主要有两点：第一点是 `UniquePointer` 对其管理的指针拥有独占所有权，通过禁用拷贝构造和赋值操作的方式防止了所有权转移的发生：

```cpp
void UniquePointerFoo()
{
    Object<int>* o = new Object<int>(1);
    UniquePointer<Object<int>> u1(o);
    UniquePointer<Object<int>> u2{ u1 }; // error: use of deleted function ‘UniquePointer<ElementType, DeleterType>::UniquePointer(UniquePointer<ElementType, DeleterType>&) [with ElementType = Object<int>; DeleterType = DefaultDeleter]’
}
```

同时又增加了移动构造和移动赋值操作，通过 move 语义来让我们可以在特定情况下显式地转移指针：

```cpp
void UniquePointerFoo()
{
    Object<int>* o = new Object<int>(1);
    UniquePointer<Object<int>> u1(o);
    UniquePointer<Object<int>> u2(std::move(u1));
    UniquePointer<Object<int>> u3;
    u3 = std::move(u2);
}
```


```shell
$ ./bin/smart-pointer 
Object::Constructor 0x3af058				# o
UniquePointer::Constructor 0xbec29664		# u1
UniquePointer::Move-ctor 0xbec2965c			# u2
UniquePointer::Constructor 0xbec29654		# u3
UniquePointer::MoveAssignment 0xbec29654	# u3
UniquePointer::Destructor 0xbec29654		# u3
Object::Destructor 0x3af058					# o
UniquePointer::Destructor 0xbec2965c		# u2
UniquePointer::Destructor 0xbec29664		# u1
```

第二点是在模板参数中增加了自定义删除器，删除器是一个 functor，我们可以在其 `operator()` 操作符中自定义 `UniquePointer` 在析构时对其管理的指针进行的操作，例如使用 `delete[]` 来释放内存，或是关闭相关的 Socket 等：

```cpp
struct ArrayDeleter
{
    template<typename T>
    void operator()(T* p) const
    {
        static_assert(sizeof(p) > 0, "can't delete pointer to incomplete type");
        delete[] p;
    }
};

void UniquePointerFoo()
{
    int* int_arr = new int[1000000];
    ArrayDeleter array_deleter;
    UniquePointer<int, ArrayDeleter> u(int_arr, array_deleter);
}
```

```shell
$ ./bin/smart-pointer 
UniquePointer::Constructor 0xbe9ad660
UniquePointer::Destructor 0xbe9ad660
```

正因为 `UniquePointer` 对资源具有独占所有权，不能同时有多个 `UniquePointer` 拥有相同的资源，因此 `AutoPointer` 中的第三个问题并不能通过使用 `UniquePointer` 来解决。



## 3 shared_ptr

`std::shared_ptr` 的应用场景在于当我们需要让多个智能指针对象同时拥有同一个指针，而又希望在这些对象都退出其作用域的时候去销毁指针。它使用了一个引用计数器来记录指针在同一时间被几个智能指针对象所共享，当这个引用计数减少为 0 时，说明已经不再有对象拥有这个指针，此时则需要进行资源的销毁。

```cpp
// A smart pointer with reference-counted copy semantics.  The
// object pointed to is deleted when the last shared_ptr pointing to
// it is destroyed or reset.
template<typename _Tp, _Lock_policy _Lp> class __shared_ptr
// ...
```

由 `std::shared_ptr` 管理的指针对象（以及引用计数器）存放在堆上，如果在同一时间有两个持有相同资源但位于不同线程中的智能指针同时访问他们所持有的资源，则可能会导致线程安全问题，因此我们还需要使用一定的机制来防止线程安全问题的发生；一般来说对引用计数器的加减修改是原子的，但对于共享资源的访问则需要使用互斥锁等机制保证线程安全。

以下是参考 `std::shared_ptr` 实现的 `SharedPointer` 类：

```cpp
template<typename ElementType, typename DeleterType = DefaultDeleter>
class SharedPointer
{
public:
    // constructors
    SharedPointer() noexcept : ptr_(nullptr), ref_count_(nullptr), deleter_(nullptr), mutex_(nullptr) { std::cout << "SharedPointer::Constructor " << this << std::endl; }
    explicit SharedPointer(ElementType* p) noexcept : ptr_(p), ref_count_(new int(1)), deleter_(new DeleterType()), mutex_(new mutex()) { std::cout << "SharedPointer::Constructor " << this << std::endl; }
    SharedPointer(ElementType* p, DeleterType *d) noexcept : ptr_(p), ref_count_(new int(1)), deleter_(d), mutex_(new mutex()) { std::cout << "SharedPointer::Constructor " << this << std::endl; }
    explicit SharedPointer(const WeakPointer<ElementType, DeleterType>& wp) noexcept : ptr_(wp.ptr_), ref_count_(wp.ref_count_), deleter_(wp.deleter_), mutex_(wp.mutex_)
    {
        std::cout << "SharedPointer::Constructor " << this << std::endl;
        IncreaseReferenceCount();
    }

    // copy-ctor
    SharedPointer(const SharedPointer<ElementType>& other) noexcept : ptr_(other.ptr_), ref_count_(other.ref_count_), deleter_(other.deleter_), mutex_(other.mutex_)
    {
        std::cout << "SharedPointer::Copy-ctor " << this << std::endl;
        IncreaseReferenceCount();
    }

    // assignment operator
    SharedPointer& operator=(SharedPointer<ElementType>& other)
    {
        if (ptr_ != other.ptr_)
        {
            Release();
            ptr_ = other.ptr_;
            ref_count_ = other.ref_count_;
            mutex_ = other.mutex_;
            deleter_ = other.deleter_;
            IncreaseReferenceCount();
        }
        return *this;
    }

    // destructor
    ~SharedPointer() noexcept
    {
        std::cout << "SharedPointer::Destructor " << this << std::endl;
        Release();
    }

    void Swap(SharedPointer<ElementType, DeleterType>& other)
    {
        std::swap(ptr_, other.ptr_);
        std::swap(ref_count_, other.ref_count_);
        std::swap(deleter_, other.deleter_);
        std::swap(mutex_, other.mutex_);
    }

    void Reset() { SharedPointer().Swap(*this); }
    void Reset(ElementType* p, DeleterType* d = nullptr) { SharedPointer(p, d).Swap(*this); }

    int UseCount() { return ref_count_ ? *ref_count_ : 0; }

    ElementType& operator*() noexcept { return *ptr_; }

    ElementType* operator->() const noexcept { return ptr_; }

    ElementType* Get() const noexcept { return ptr_; }

    const DeleterType& GetDeleter() const noexcept { return *deleter_; }
    
    void Release()
    {
        if (!ptr_)
            return;
        bool delete_flag = false;
        mutex_->lock();
        if (--(*ref_count_) == 0)
        {
            GetDeleter()(ptr_);
            delete ref_count_;
            delete deleter_;
            delete_flag = true;
        }
        mutex_->unlock();
        
        if (delete_flag)
        {
            delete mutex_;
        }
        
    }

	void IncreaseReferenceCount()
	{
        if (!ptr_)
            return;
		mutex_->lock();
        ++(*ref_count_);
		mutex_->unlock();
	}
    
    ElementType* ptr_;
    int *ref_count_;
    DeleterType* deleter_;
	mutex* mutex_;
};
```

现在可以正确地让一个指针被多个智能指针对象所持有了：



```cpp
void SharedPointerFoo()
{
    Object<int>* o = new Object<int>(1);
    SharedPointer<Object<int>> s1(o);
    SharedPointer<Object<int>> s2(s1);
    s1.Reset(nullptr);
    s2.Reset(nullptr);
}
```

```shell
$ ./bin/smart-pointer 
Object::Constructor 0x1ac7058 # o1
SharedPointer::Constructor 0xbe89c65c # p1
SharedPointer::Copy-ctor 0xbe89c64c # p2
SharedPointer::Constructor 0xbe89c630 # p1.Reset
SharedPointer::Destructor 0xbe89c630 # p1.Reset
SharedPointer::Constructor 0xbe89c630 # p2.Reset
SharedPointer::Destructor 0xbe89c630 # p2.Reset
Object::Destructor 0x1ac7058 # o1
SharedPointer::Destructor 0xbe89c64c # p2
SharedPointer::Destructor 0xbe89c65c # p1
```



但又出现了[循环引用](https://www.learncpp.com/cpp-tutorial/circular-dependency-issues-with-stdshared_ptr-and-stdweak_ptr/)的问题，例如：



```cpp
void SharedPointerFoo()
{
    struct Node
    {
        int i_;
        SharedPointer<Node> prev_;
        SharedPointer<Node> next_;

        Node(int i) : i_(i) { std::cout << "Node::Constructor " << this << std::endl; }
        ~Node() { std::cout << "Node::Destructor " << this << std::endl; }
    };

    Node *n1 = new Node(1), *n2 = new Node(2);
    SharedPointer<Node> s1(n1), s2(n2);
    cout << s1.UseCount() << ' ' << s2.UseCount() << endl;
    s1->next_ = s2;
    s2->prev_ = s1;
    cout << s1.UseCount() << ' ' << s2.UseCount() << endl;
}
```



```shell
$ ./bin/smart-pointer 
SharedPointer::Constructor 0x3f905c # n1->prev_
SharedPointer::Constructor 0x3f906c # n1->next_
Node::Constructor 0x3f9058 # n1
SharedPointer::Constructor 0x3f948c # n2->prev_
SharedPointer::Constructor 0x3f949c # n2->next_
Node::Constructor 0x3f9488 # n2
SharedPointer::Constructor 0xbeca0658 # s1
SharedPointer::Constructor 0xbeca0648 # s2
1 1
2 2
SharedPointer::Destructor 0xbeca0648 # s2
SharedPointer::Destructor 0xbeca0658 # s1
```



这里虽然 `s1` 和 `s2` 两个 `SharedPointer<Node>` 在函数退出时被成功地销毁了，但它们所持有的 `n1` 和 `n2` 两个对象却没有，因为 `s1` 和 `s2` 的引用计数都没有减少为 0，只有当 `s1->next_` 不再指向 `s2`，且 `s2->prev_` 不再指向 `s1` 时，两者的引用计数才能够正确的减少为 0。



## 4 weak_ptr



`std::weak_ptr` 是一种弱引用智能指针，它和 `std::shared_ptr` 的唯一区别是它必须由 `std::shared_ptr` 或 `std::weak_ptr` 显式转换而来，而不能由使用 `new` 创建的对象进行构造，因此其管理的资源实际上是被另一个 `std::shared_ptr` 所持有的，而其本身则只是提供了对被管理资源的访问能力，同时也不对被管理资源的生命周期造成影响，即不会修改 `std::shared_ptr` 的引用计数。



下面是参考 `std::weak_ptr` 实现的 `WeakPointer` 类：



```cpp
template<typename ElementType, typename DeleterType> class SharedPointer;

template<typename ElementType, typename DeleterType = DefaultDeleter>
class WeakPointer
{
public:
    // constructors
    WeakPointer() noexcept : ptr_(nullptr), ref_count_(nullptr), deleter_(nullptr), mutex_(nullptr) { std::cout << "WeakPointer::Constructor " << this << std::endl; }
    explicit WeakPointer(SharedPointer<ElementType, DeleterType>& sp) noexcept : ptr_(sp.ptr_), ref_count_(sp.ref_count_), deleter_(sp.deleter_), mutex_(sp.mutex_) { std::cout << "WeakPointer::Constructor " << this << std::endl; }

    // copy-ctor
    WeakPointer(WeakPointer<ElementType>& other) noexcept : ptr_(other.ptr_), ref_count_(other.ref_count_), deleter_(other.deleter_), mutex_(other.mutex_)
    {
        std::cout << "WeakPointer::Copy-ctor " << this << std::endl;
    }

    // assignment operator
    WeakPointer& operator=(SharedPointer<ElementType, DeleterType>& sp)
    {
        ptr_ = sp.ptr_;
        ref_count_ = sp.ref_count_;
        mutex_ = sp.mutex_;
        deleter_ = sp.deleter_;
        return *this;
    }

    // destructor
    ~WeakPointer() noexcept { std::cout << "WeakPointer::Destructor " << this << std::endl; }

    void Swap(WeakPointer& other)
    {
        std::swap(ptr_, other.ptr_);
        std::swap(ref_count_, other.ref_count_);
        std::swap(deleter_, other.deleter_);
        std::swap(mutex_, other.mutex_);
    }

    void Reset() { WeakPointer().Swap(*this); }

    SharedPointer<ElementType, DeleterType> Lock() const
    {
        return Expired() ? SharedPointer<ElementType, DeleterType>()
                         : SharedPointer<ElementType, DeleterType>(*this);
    }

    int UseCount() { return *ref_count_; }

    const DeleterType& GetDeleter() const noexcept { return *deleter_; }

    bool Expired() const { return *ref_count_ == 0; }

    ElementType* ptr_;
    int *ref_count_;
    DeleterType* deleter_;
	mutex* mutex_;
};
```



`std::weak_ptr` 不能控制被管理资源的生命周期，因此我们在使用的时候需要先判断被管理资源是否存在，我们可以借助 `std::weak_ptr::lock` 获取一个新的 `std::shared_ptr` 对象以达到安全访问资源的目的：



```cpp
void WeakPointerFoo()
{
    Object<string> *o = new Object<string>("test");
    SharedPointer<Object<string>> s(o);
    WeakPointer<Object<string>> w(s);
    s.Reset();

    auto p = w.Lock().Get();
    cout << w.Expired() << endl;
    cout << static_cast<void*>(p) << endl;
}
```

```shell
$ ./bin/smart-pointer
Object::Constructor 0xf8c058 # o
SharedPointer::Constructor 0xbe97a630 # s
WeakPointer::Constructor 0xbe97a620 # w
SharedPointer::Constructor 0xbe97a608 # temporary variable in s.Reset()
SharedPointer::Destructor 0xbe97a608 # temporary variable in s.Reset()
Object::Destructor 0xf8c058 # o
SharedPointer::Constructor 0xbe97a65c # temporary variable in w.Lock()
SharedPointer::Destructor 0xbe97a65c # temporary variable in w.Lock()
1 # w.Expired()
0 # p
WeakPointer::Destructor 0xbe97a620 # w
SharedPointer::Destructor 0xbe97a630 # s
```



如果把上面的 `s.Reset()` 去掉，那么 `w.Lock()` 则会返回一个包含有 `Object<string>` 对象的 `SharedPointer`，上面的 `*o` 也不会在 `s.Reset()` 之后析构掉：



```cpp
void WeakPointerFoo()
{
    Object<string> *o = new Object<string>("test");
    SharedPointer<Object<string>> s(o);
    WeakPointer<Object<string>> w(s);
    // s.Reset();

    auto p = w.Lock().Get();
    cout << w.Expired() << endl;
    cout << static_cast<void*>(p) << endl;
}
```

```shell
$ ./bin/smart-pointer
Object::Constructor 0xb5c058 # o
SharedPointer::Constructor 0xbef4562c # s
WeakPointer::Constructor 0xbef4561c # w
SharedPointer::Constructor 0xbef45658 # temporary variable in s.Reset()
SharedPointer::Destructor 0xbef45658 # temporary variable in s.Reset()
0 # w.Expired()
0xb5c058 # p
WeakPointer::Destructor 0xbef4561c # temporary variable in w.Lock()
SharedPointer::Destructor 0xbef4562c # # temporary variable in w.Lock()
Object::Destructor 0xb5c058 # o
```



对于循环引用的问题，将需要互相指向的智能指针改为 `WeakPointer` 则可以成功避免指针对象不能正常析构的问题：



```cpp
void WeakPointerFoo()
{
	struct Node
    {
        int i_;
        WeakPointer<Node> prev_;
        WeakPointer<Node> next_;

        Node(int i) : i_(i) { std::cout << "Node::Constructor " << this << std::endl; }
        ~Node() { std::cout << "Node::Destructor " << this << std::endl; }
    };

    Node *n1 = new Node(1), *n2 = new Node(2);
    SharedPointer<Node> p1(n1), p2(n2);
    cout << p1.UseCount() << ' ' << p2.UseCount() << endl;
    p1->next_ = p2;
    p2->prev_ = p1;
    cout << p1.UseCount() << ' ' << p2.UseCount() << endl;
}
```

```shell
$ ./bin/smart-pointer 
WeakPointer::Constructor 0xb505c # n1->prev_
WeakPointer::Constructor 0xb506c # n1->next_
Node::Constructor 0xb5058 # n1
WeakPointer::Constructor 0xb548c # n2->prev_
WeakPointer::Constructor 0xb549c # n2->next_
Node::Constructor 0xb5488 # n2
SharedPointer::Constructor 0xbe904658 # s1
SharedPointer::Constructor 0xbe904648 # s2
1 1
1 1
SharedPointer::Destructor 0xbe904648 # s2
Node::Destructor 0xb5488 # n2
WeakPointer::Destructor 0xb549c # n2->next_
WeakPointer::Destructor 0xb548c # n2->prev_
SharedPointer::Destructor 0xbe904658 # s1
Node::Destructor 0xb5058 # n1
WeakPointer::Destructor 0xb506c # n1->next_
WeakPointer::Destructor 0xb505c # n1->prev_
```

