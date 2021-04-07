# Effective C++ 读书笔记

[TOC]

## 0 导言

### 1 构造函数

default 构造函数：可被调用而不带任何实参的构造函数，这样的构造函数要么没有参数，要么每个参数都带有默认值，例如

```c++
class Bar {
public:
    // explicit Bar(); // 是 default 构造函数
    // explicit Bar(int x = 0) // 不是 default 构造函数
    explicit Bar(int x = 0, bool b = true); // 是 default 构造函数
private:
    int x;
    bool b;
};
```

explicit 关键字：阻止执行隐式类型转换，其优点是禁止了编译器执行非预期的类型转换，例如

```c++
void Foo(Bar obj); // Foo 函数的参数是一个类型为 Bar 的对象

Bar obj_1; // 构造一个 Bar 类型的对象
Foo (obj_1); // 没问题，传递一个 Bar 类型的对象给 Foo 函数
Foo (Bar()); // 没问题，构造一个 Bar 类型的对象，并传递给 Foo 函数
Foo (2); // 如果 Bar 的构造函数没有被声明为 explicit，那么会调用 Bar 的构造函数构造一个成员变量 x = 2 的对象，也就是说发生了隐式类型转换；如果其构造函数被声明为 explicit，那么就不会构造出 Bar 类型的对象
```

copy 构造函数：用同类型的对象初始化新的对象，它定义了一个对象如何 pass by reference。

copy assignment：拷贝另一个同类型对象的值到自身，同时 "=" 也可以用来调用 copy 构造函数，例如

```c++
class Bar {
public:
    Bar(); // default 构造函数
    Bar(const Bar &rhs); // copy 构造函数
    Bar& operator=(const Bar &rhs); // copy assignment
};

Bar b1; // 调用 default 构造函数
Bar b2(b1); // 调用 copy 构造函数
b1 = b2; // 调用 copy assignment
Bar b3 = b2; // 调用 copy 构造函数
```

#### 0.2 未定义行为

```c++
int *p = nullptr;
std::cout << *p; // 对一个空指针进行取值，导致不确定行为

char name[] = "Joel"; // name 是一个长度为 5 的 char 类型数组
char c = name[10]; // 指向一个无效的数组索引，导致不确定行为
```



### 2. 尽量使用 const, enum, inline 替换#define

使用编译器替换预处理器；使用宏定义的变量在编译器处理源码之前就被替换了，从未进入记号表。

### 3. 尽可能使用 const

#### 3.1 Iterator

STL 中的 iterator 类似于一个 T* 指针，如果将 iterator 声明为 const 那么实际上是声明了一个 T* const，即 const pointer to T，指针 T 初始化后不能再指向其他的对象；而 STL 中的 const_iterator 则本身就是一个 const T*，即 pointer to const T，指向一个 const T 类型的对象，对象的值不能被修改，而其指向是可以被修改为其他对象的。

**对于 const 类型的 STL 容器，应该使用 const_iterator 来进行遍历；对于非 const 类型的 STL 容器，应该使用 iterator 来遍历。**

iterator 不用使用 reference 的形式绑定到 STL::begin() 上，因为 STL::begin() 返回的是一个临时 pointer 变量，且对于内置类型和 STL 的迭代器和函数对象，pass-by-value 会比 pass-by-reference 更高效。

使用 auto 遍历一个 STL 容器时，会直接遍历，而不是使用 iterator 指针。

#### 3.2 const 和 multable

### 4. 确保在使用前先初始化数据

#### 4.1 成员变量初始化

对于内置类型，手动进行初始化，因为 C++ 不保证会初始化它们。

对于类，在构造函数中将每一个成员变量初始化，成员变量的初始化动作发生在进入构造函数本体之前，使用 member initialization list 效率较高，且不会与 assignment 混淆；在成员初始化列表中总是列出所有成员变量，且排列次序应该和声明次序相同。

#### 4.2 static 变量

static 对象的寿命从被构造出来直到程序结束为止；决定 non-local static 对象的初始化次序非常困难，常见形式是使用 implicit template instantiations 模板隐式具现化；消除这个问题的方法是使用 local static 对象，因为 local static 对象会在函数首次被调用时被初始化。

### 5. C++ 默认编写和调用的函数

#### 5.1 类

编译器会自动为类声明 copy 构造函数和 copy assignment，它们只单纯地将来源对象的每一个 non-static 成员变量拷贝到目标对象；

编译器会自动声明 non-virtual 的析构函数；

当**没有任何构造函数**时，编译器会为类声明 default 构造函数；这些函数都是 inline public 的，并且只有当这些函数被调用的时候才会被编译器创建出来，前提是生成的代码是合法的，即对于内含 const 成员的或在基类中将 copy assignment 声明为 private 的派生类是无法自动生成对应的函数的。

### 6. 对于不想使用的编译器自动生成的函数，显式地拒绝

函数的参数名称不一定需要被写出来。

将 copy 构造函数和 copy assignment 声明为 private 并且不实现，可以防止编译器自动生成这两个函数，同时防止它们在外部被调用；但如果在 member 函数或者 friend 函数中调用了，那么连接器会报错；我们应该尽可能将连接期错误移动至编译期错误，尽早检查出错误。

可以设计一个专门为了阻止 copying 动作为存在的 base class，因为 base class 的 copy 函数和 copy assignment 是 private 的，派生类无法调用基类的私有方法，所以编译器会拒绝为子类自动生成这两个函数并报错

```c++
class Uncopyable {
protected:
    Uncopyable();
    ~Uncopyable();
private:
    Uncopyable(const Uncopyable &);
    Uncopyable & operator=(const Uncopyable &);
}

class Foo: private Uncopyable {
    ...
}
```

C++ 11 中引入了 default 和 delete 关键字，前者可以让编译器自动生成函数，后者可以直接禁止使用函数

```c++
class Foo: private Uncopyable {
public:
    A() = default;
    A(const A &) = delete;
}
```

### 7. 为多态基类声明 virtual 析构函数

当派生类对象经由基类指针被删除时，如果其析构函数不是 virtual 的，那么编译器不会去虚表上查找到派生类的析构函数正确地进行调用，而是只执行基类的部分，造成局部销毁的现象。

当不意图将一个函数作为基类实现多态时，不要将它的析构函数声明为 virtual；只有当类中至少含有一个 virtual 函数时，才为其声明 virtual 析构函数。

所有 STL 容器都没有 virtual 析构函数。

对于抽象类，将其析构函数声明为 pure virtual。

析构函数的运作方式是从最深层的派生类的析构函数依次调用到基类的析构函数，因此必须为纯虚函数的 virtual 析构函数提供一个定义。

### 8. 不要在析构函数中抛出异常

在 C++ 中，如果同时存在两个异常，程序会发生未定义行为（或结束执行）。

### 9. 不要在构造函数和析构函数内调用 virtual 函数

对于一个派生类，其对象在构造的时候会从基类的构造函数一直调用到派生类的构造函数。

### 10. 让 oprator= 返回一个 reference to *this

```c++
class Foo {
public:
    ...
    Foo &operator=(const Foo &rhs) {
        ...
        return *this;
    }
}
```

STL 中的容器均使用这个协议。

### 11. 在 operator= 中处理自我赋值

在 operator= 的开头加上 identity test，防止发生因为自我赋值产生的指针指向问题：

```c++
Foo& Foo::operator=(const Foo &rhs) {
    if (this == &rhs)
        return *this;
    delete p_bm;
    p_bm = new Bitmap(*rhs.p_bm);
    return *this;
}
```

这样的实现并不具备异常安全性，因为 new Bitmap 可能会因为内存不足或 copy 构造函数出错导致异常。

```c++
Foo& Foo::operator=(const Foo &rhs) {
    Bitmap *p_bm_temp = p_bm;
    p_bm = new Bitmap(*rhs.p_bm);
    delete p_bm_temp;
    return *this;
}
```

### 12. 拷贝对象是确保复制每一个成员

在编写一个 copying（包括 copy 构造函数和 copy assignment）函数时，确保：1. 复制所有的 local 成员变量；2. 调用所有基类对应的 copying 函数。

如果 copy 构造函数和 copy assignment 有相似的代码，可以定义一个新的 private 类型的初始化函数供其实用。

### 13. 以对象管理资源

获得资源后立刻放进管理对象，即资源取得时机便是初始化时机（Resource Acquisition Is Initialization, RAII），总是在取得资源后马上用它来初始化某个管理对象；管理对象使用析构函数确保资源被释放。例如使用智能指针。

### 14. 在资源管理类中小心 copying 行为

### 16. 成对使用 new 和 delete 的时候要采取相同形式

单一对象的内存布局不同于数组对象的内存布局，数组的内存还包含了数组大小的记录，以便 delete 知道需要调用多少次析构函数；new[] 和 delete[] 一定要成对出现。

## 4 设计和声明

### 18. 让接口容易被正确使用，不易被误用

```
不一致性对开发人员造成的心理和精神上的摩擦与争执，没有任何一个 IDE 可以完全抹除。
```

### 19. 像设计一个新类型一样设计 class

设计一个新的 class 的时候需要考虑的因素包括但不限于：

1. 类的对象应该如何被创建和销毁 - 构造函数和析构函数
2. 初始化和赋值有什么区别 - copy 构造函数和 copy assignment 的区别
3. 对象被 pass by value 时会发生什么 - copy 构造函数的设计
4. 哪些操作是有效的 - 成员函数必须进行错误检查
5. 继承图系 inheritance graph 的约束 - virtual，override，final，析构函数相关
6. 类型转换 - 定义类型转换函数
7. 操作符的定义
8. public 和 private 函数的区分
9. 未声明接口 undeclared interface
10. 泛型的考虑

### 20. 使用 pass-by-reference-to-const 替代 pass-by-value

对象切割 slicing ：用一个派生类实参去初始化一个基类形参，执行其拷贝构造函数，导致函数内的参数实际上会执行基类的行为。

以 pass-by-reference-to-const 的方式传递参数可以防止对象的拷贝构造；也可以避免对象切割问题，因为 pass-by-reference-to-const 的方式可以防止拷贝构造的发生，从而正确地表现多态特性。

在 C++ 的底层，reference 一般是使用 pointer 实现的，因此 pass by reference 实际上传递的是指针；对于内置类型和 STL 的迭代器和函数对象，pass-by-value 会比 pass-by-reference 更高效。

### 21*. 必须返回对象时，不要返回其 reference

```
任何时候看到一个 reference 声明式，你都应该立刻问自己，它的另一个名称是什么？
```

无论是在堆还是栈上创建的对象，使用 reference 返回都会造成问题，前者会发生内存泄漏，后者会导致未定义行为。

```
绝不要返回一个指向 local stack 的 pointer 或 reference，或返回 reference 指向一个 heap-allocated 对象，或返回一个指向 local static 的 pointer 或 reference。
```

### 22*. 将成员变量声明为 private

```
将成员变量隐藏在函数接口的背后，可以为“所有可能的实现”提供弹性。例如这可以使得成员变量在被读或被写时轻松通知其他对象，可以验证 class 的约束条件以及函数的前提和事后状态，可以在多线程环境中执行同步控制等等。
```

从封装的角度出发，只有两种访问权限：private（提供封装）和其他（不提供封装）；protected 并不比 public 更有封装性，因为无论修改 protected 变量或是 public 变量，都会有大量代码受到破坏。

### 23. 尽量使用 non-member，non-friend 替换 member 函数

```
将所有遍历函数放在多个头文件内，但隶属于同一个命名空间，意味着客户可以轻松扩展这一组遍历函数，他们需要做的就是添加更多 non-member non-friend 函数到此命名空间中。
```

尽量使用 non-member，non-friend 替换 member 函数，可以增加封装性，包裹弹性 packaging flexibility，和机能扩充性。

### 24. 如果所有参数都需要类型转换，那么将这个函数设置为 non-member

### 25. 支持一个不抛出异常的 swap 函数

所有 STL 容器都提供 public swap 函数和 std::swap 的特化版本。

## 5. 实现 Implementation

```
太快定义变量可能造成效率上的拖延；过度使用转型 cast 可能导致代码变慢又难维护，又招来微妙难解的错误；返回对象内部数据的句柄 handle 可能会破坏封装性并留给客户悬空句柄 dangling handle；未考虑异常带来的冲击可能导致资源泄漏和数据腐坏；过度热心地 inlining 可能引起代码膨胀；过度耦合 coupling 可能导致冗长构建时间 build time。
```

### 26. 尽量延后变量定义地出现

延后变量的定义，直到不得不使用它的前一刻为止，否则需要承受额外的构造和析构成本，以及无意义地 default 构造行为。

对于需要在循环内反复使用的

### 27. 少使用转型

C++ 风格的类型转换有四种：

- const_cast：将对象的常量性移除
- dynamic_cast：执行安全向下转型 safe downcasting，用来决定对象是否归属继承体系中的某个类型；它无法由 C 风格的类型转换执行，但可能耗费很大成本
- reinterpret_cast：执行低级转型，其动作和结果可能取决于编译器
- static_cast：执行隐式转换，例如将 non-const 转为 const 对象，或将 int 转为 double

C++ 风格的类型转换更好，原因是：

- 更容易被辨识
- 每种转型动作有更明确的意义，易于调试

任何一个类型转换都会让编译器编译出运行期的执行码

### 28. 避免返回指向对象内部成员的 handle

对象的内部不只有成员变量，还有不被公开的成员函数，绝不应该返回一个指向“访问级别较低”的成员函数。

避免返回指向对象内部成员的 handle，可以增加封装性，使得 const 成员函数的行为是真正的 const，也能降低出现悬空指针的可能性。

### 29. 努力写出异常安全的代码

当异常被抛出时，带有异常安全性的函数会：

- 不泄露任何资源，即不因为流程阻塞而导致其后的资源未被释放
- 不允许数据败坏，即不允许悬空指针的出现

异常安全函数提供以下三个保证之一：

1. 基本保证：异常抛出时，没有对象或数据结构被破坏，所有对象都处于前后一致的状态
2. 强烈保证：如果抛出异常，程序状态不改变；如果程序失败，程序会恢复到调用函数前的状态
3. 不抛异常保证：程序保证不抛出异常，因为它总能够完成原先承诺的功能

### 30. 理解 inlining

```
在一台内存有限的机器上，过度热衷于 inlining 会导致程序体积过大；即使拥有虚拟内存，inlining 造成的代码膨胀也会导致额外的换页 paging 行为，降低指令高速缓存装置的击中率 instruction cache hit rate，以及伴随而来的效率损失
```

inline 可以隐喻提出，例如将函数定义于 class 的定义内部。

inline 函数一般被放在头文件内，因为 inlining 在大多数 C+ 程序中是编译期行为，而编译器为了将函数调用替换为被调用函数的本体，编译器需要知道函数的具体实现。template 也是如此。

将大多数 inlining 限制在小型且被频繁调用的函数上，可以使得调试和二进制升级更容易，也可以使得潜在的代码膨胀问题最小化，并提升程序运行速度。

### 31. 将文件间的编译依存关系降至最低

`#include` 在定义文件和包含文件之间形成了编译依存关系 compilation dependency，如果头文件中的任何一个被改变，或头文件所依赖的其他头文件有改变，那么每一个包含有该头文件的文件都会重新编译，任何使用该文件中定义的类和函数的文件也需要重新编译，这被叫做连串编译依存关系 cascading compilation dependencies。

当编译器看到定义时，它必须知道要分配多少内存。

## 6. 继承和面向对象设计

### 32. 确保 public 继承是 `is-a` 关系

适用于 base class 的每一个函数和对象也一定适用于 derived class。

### 33. 避免遮挡继承得来的名称

内层作用域的名称会遮挡外部作用域的名称，当编译器处于一个作用域内的时候，会先在 local 作用域内查找是否有对应的变量名称，如果没有才会去其他作用域找。

可以使用 using 关键字显式地声明其他作用域的变量和函数，使得它们在当前作用域可见。

### 34. 区分接口继承和实现继承

不同类型成员函数的目的：

- 纯虚函数：让派生类继承函数的接口
- 虚函数：让派生类继承函数的接口和默认实现
- 非虚函数：让派生类继承函数的接口和强制性实现

如果成员函数是个非虚函数，意味着它并不打算在派生类中被 override；非虚函数代表其不变性 invariant 凌驾于特异性 specialization 之上。

```
一个典型的程序员有 80% 的执行时间花在 20% 的代码上；它意味着，平均而言函数调用中可以有 80% 是 virtual 而不冲击程序的大体效率，所以在担心 virtual 函数的成本之前，先将心力放在 20% 的代码上。
```

### 35. 考虑 virtual 函数以外的其他选择

1. 通过 non-virtual interface 实现模板方法模式

非虚接口 non-virtual interface (NVI)：使客户通过 public non-virtual 成员函数间接调用 private virtual 函数，它是所谓的模板方法模式的一个独特表现形式，把这个 public non-virtual 函数称为 virtual 函数的包装器 wrapper。

2. 通过 function pointer 实现策略模式

### 36. 绝不重新定义通过继承得来的非虚函数

非虚函数是静态绑定的，被调用函数与指针本身类型对应；虚函数是动态绑定的，被调用函数与指针所指对象类型对应。

### 37. 绝不重新定义通过继承得来的默认参数值

- 静态类型：在程序中被声明时所采用的类型
- 动态类型：指针实际所指向的类型；动态类型可以表现出一个对象将有什么行为，且动态类型在执行过程中可以改变

```c++
Shape *p_s;
Shape *p_c = new Circle();
Shape *p_r = new Rectangle();
```

此处 p_s，p_c，p_r 三者的静态类型都是 Shape，而它们的动态类型分别是 Shape，Circle，Rectangle

在继承一个带有默认参数值的虚函数时，不应重新定义其默认参数，因为虚函数是动态绑定的，而默认参数是静态绑定的，默认参数只与静态类型有关。

### 38. 通过组合 composition 构建出 `has-a` 或 `由其实现` 的关系

在应用域内，复合意味着 `has-a`；在实现域内，复合意味着 `is-implemented-in-terms-of`。

### 39. 明智而审慎地使用 private 继承

private 继承意味着由其实现 implemented-in-terms-of，通过 private 继承得到的基类中的 public 和 protected 函数和对象都是 private 的。

尽可能使用组合，必要时才使用 private 继承。

### 40. 明智而审慎地使用多重继承

多重继承会导致比较多的歧义，例如

- 多重继承的两个基类都有相同签名的函数：他们具有相同的匹配程度而没有最佳匹配，为了解决这个起义，必须明确指出要调用的是哪一个基类的函数
- 菱形继承：某个基类到某个派生类之间有一条以上的通路，那么派生类默认对多份基类数据都执行拷贝，而如果使用虚继承，那么派生类只会保留一份拷贝；标准库中的 basic_ios，basic_istream，basic_ostream 和 basic_iostream 也是菱形继承体系，它们采用的是虚继承

一般来说，public 继承都应该是 virtual 继承

## 7. 模板和泛型编程



### 41. 了解隐式接口和编译器多态

- 面向对象编程总是以显式接口 explicit interface 和运行期多态 runtime polymorphism 解决问题，显式接口由函数的签名构成
- template 参数具有隐式接口 implicit interface，隐式接口由有效表达式组成，也就是对于模板变量来说，它必须提供所需的成员函数和操作符；template 的多态是通过函数重载解析在编译期完成的

### 42. 了解 typename 的双重意义

在使用 template 时，typename 和 class 意义完全相同。

template 中，在参数 class 内嵌套的变量称为嵌套从属名称 nested dependent name，不依赖任何参数的变量叫做非从属名称 non-dependent name。

嵌套从属名称可能导致解析困难，编译器在解析时，如果遇到 template 中有一个嵌套从属名称，会便假设这个名称不是一个类型，例如：

```c++
template <typename T>
void Foo(const T &t) {
    T::const_iterator iter(t.begin()); // Warning: Missing 'typename' prior to dependent type name 'T::const_iterator'
}
```

编译器会假设 T::const_iterator 不是一个类型，所以会发出 warning，只需要在其前面加上关键字 typename 即可；但 typename 不能出现在基类列表中，也不能在成员初始列 member initialization list 中作为基类修饰符，例如：

```c++
template <typename T>
class Derived: public Base<T>::Nested { // 基类列表中不嫩使用 typename
public:
    explicit Derived(int x):Base<T>::Nested(x) { // 成员初始列中不能使用 typename
        typename Base<T>::Nested temp; // 在普通的嵌套从属名称前可以使用 typename
    }
};
```

### 43. 学会处理模板化基类内的名称

### 44. 将与参数无关的代码抽离 templates

## 8. 自定义 new 和 delete

### 49. 了解 new 的行为

当 operator new 无法满足内存分配的需求时，它会抛出异常；

在 operator new 抛出异常之前，它会先调用错误处理函数 `new-handler`，可以使用 `std::set_new_handler` 来设置这个函数。

| c++ 11        | operator new                                                 |
| ------------- | ------------------------------------------------------------ |
| throwing (1)  | void* operator new (std::size_t size);                       |
| nothrow (2)   | void* operator new (std::size_t size, const std::nothrow_t& nothrow_value) noexcept; |
| placement (3) | void* operator new (std::size_t size, void* ptr) noexcept;   |

一个设计良好的 new-handler 应该完成：

1. 可分配更多内存；否则调用另一个 new-handler，或抛出异常
2. 捕获 bad_alloc，调用 abort 或 exit

### 50. 了解 new 和 delete 的合理替换时间

替换默认 operator new 和 operator delete 的理由：

1. 检测运行时错误
2. 优化性能：提高分配和归还的速度，降低额外的空间开销，弥补非最佳对齐（suboptimal alignment）
3. 统计数据

### 51. 编写 new 和 delete 的时候需要遵守规约

## 9. 杂项 Miscellany

### 53. 关注编译器警告

在编译时使用 `-Wall -Wextra -Werror`。

### 54. 熟悉 [TR1](https://en.wikipedia.org/wiki/C%2B%2B_Technical_Report_1) 和标准库内容

tr1 是 2007 年提出的对标准库的补充，包括了 shared_ptr, function, bind, unordered_map, unordered_set, regex, tuple, array, mem_fn, reference_wrapper 等函数和类，以及 type traits, result_of 等模板。

### 55. 熟悉 boost

