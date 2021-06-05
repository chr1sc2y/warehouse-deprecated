# Effective Modern C++ 读书笔记

[TOC]

## 1. 类型推导

### 1. 理解模板类型推导

```c++
template<typename T>
void f(ParamType param);

f(expr);
```

T 的类型推导同时依赖于 expr 和 ParamType 的形式，分三种情况：

1. ParamType 是指针或引用类型，但不是万能引用

    - 如果 expr 是引用类型，先将引用部分忽略掉，然后对 expr 和 ParamType 的类型执行模式匹配，来决定 T 的类型

2. ParamType 是万能引用

    - 万能引用是指在模板函数中，有一个模板参数 T，那么 T&& 就是万能引用/通用引用

3. ParamType 既非指针也非引用

    - 通过 pass-by-value 的方式处理，param 会成为一个全新的拷贝
    - const 和 volatile 等关键字都会被忽略，因为 param 是一个拷贝，而不是原本的变量 expr


### 2. 理解 auto 自动类型推导

在 C++ 11 中，decltype 最主要的用途就是用于获取函数模板返回类型。

对一个 T 类型的容器使用 operator\[\]，通常会返回一个 T& 对象，但对于 std::vector\<bool\>，operator[] 不会返回 bool&，它会返回一个有名字的对象类型。

### 3. 

### 4. 查看推导类型

运行时输出 std::type_info::name：`std::cout << typeid(x).name() << '\n';`

```c++
template<typename T>
static void PrintType(const T &t)
{
    std::cout << typeid(T).name() << std::endl;
    std::cout << typeid(t).name() << std::endl;
}

int main()
{
    PrintType(1);
}
```

```bash
[joelzychen@DevCloud ~/test]$ clang++ -std=c++11 -otest joelzychen_test.cpp 
[joelzychen@DevCloud ~/test]$ ./test 
i
i
```

结果并不可靠，T 的类型是 int，但 t 的类型应该是 const int&。



## 2. auto

### 5. 优先使用 auto 而不是显式类型声明

```
std::function 是 C++11 标准模板库中的一个模板，它泛化了函数指针的概念；与函数指针只能指向函数不同，std::function 可以指向任何可调用对象，也就是那些像函数一样能进行调用的东西。
当你声明函数指针时你必须指定函数签名，同样当你创建 std::function 对象时你也需要提供函数签名，例如
bool(const std::unique_ptr<Widget> &p1, const std::unique_ptr<Widget> &p2);
对应的 std::function 类型的签名是
std::function<bool(const std::unique_ptr<Widget> &p1,
const std::unique_ptr<Widget> &p2)> func;
```



## 3. 转向现代 C++

### 7. 创建对象时区分 () 和 {}

#### 使用 uniform initialization

使用等号进行初始化时，会调用拷贝构造函数，而不是赋值操作符：

```cpp
Widget w2 = w1;
```

不可复制的对象（如 std::atomic）可以采用大括号和小括号进行初始化，不能使用等于符号：

```cpp
std:atomic<int> a1{ 0 };	// ok
std:atomic<int> a1(0);		// ok
std:atomic<int> a1 = 0;	// error
```

非静态成员对象可以用大括号和等于符号进行初始化，不能使用小括号：

```cpp
class Widget {
private:
    int x{ 0 };	// ok
    int y = 0;	// ok
    int z(0);	// error
}
```

由此可见只有大括号初始化适用于所有场合，因此建议使用 uniform initialization，即 C++ 11 中的 braced initialization。

#### uniform initialization 的特性

1. 大括号初始化禁止内建型别进行 narrowing conversion（隐式窄化类型转换），即如果被初始化的对象和用于初始化的值无法相互转化，则会在编译时报错：
    
```cpp
   double x, y;
   int z{ x + y };	// error: narrowing conversion of ‘(x + y)’ from ‘int’ to ‘double’ inside { } [-Werror=narrowing]
   ```
   
2. 大括号初始化免疫 most vexing parse；C++ 规定任何能够解析为声明的都要解析为声明，这就导致了有时如果想以默认方式构造对象，却不小心声明了函数：

   ```cpp
class Foo {};
   Foo foo1;
   Foo foo2();
   cout << boost::typeindex::type_id_with_cvr<decltype(foo1)>().pretty_name() << endl;
cout << boost::typeindex::type_id_with_cvr<decltype(foo2)>().pretty_name() << endl;
   ```
   
   ```bash
   MostVexingParseTest()::Foo
   MostVexingParseTest()::Foo ()
   ```
   
3. 使用 `auto` 关键字声明一个用大括号初始化的变量，变量会被推导为 `std::initializer_list` 类型。

4. 只要定义了初始化列表构造函数，那么使用大括号初始化时就会优先用其进行初始化，即使其形参的构造函数无法被调用；只有在无法把大括号初始化列表中的实参转换为初始化列表中的类型时，才会尝试调用其他构造函数。



### 9.

对于通过 typedef 定义的变量名，在模板中需要用 typename 前缀来修饰，例如

```c++
template <typename T>
class Widget
{
    typename Vec<T>::type vec;
};

template <typename T>
using Widget_t = typename Widget<T>::type;
```



### 10. 优先考虑限域枚举

通常来说，在大括号中声明的名字的作用域会被限制在大括号之内，而枚举类型却拥有与定义它们的 enum 相同的作用域，这叫做未限域枚举 unscoped enum。C++ 11 中新增了限域枚举 scoped enum，它不会导致枚举名泄露，例如：

```c++
enum class Color : uint8_t { // 为限域枚举指定基础类型，默认是 int
    red = 0,
    green = 1,
    blue = 0xff
};
```

使用限域枚举的好处：

1. 减少命名空间污染
2. 限域枚举的枚举名是强类型，而未限域枚举的枚举名会隐式转换为整型
3. 限域枚举可以前置

### 11. 使用 deleted 函数替代未定义的 private 成员函数

防止函数被调用的方法：
1. 将函数声明为私有成员，可以防止其被外部调用，但仍可以被其他成员函数或友元函数调用
2. 只声明而不定义函数，如果它们被调用就会在**链接时**引发缺少函数定义 missing function definitions 的错误
3. C++ 11 中，使用 = delete 将函数标记为 deleted 函数，调用 deleted 函数将不能通过**编译**

C++ 中一般有三类指针：
1. void\* 指针，无法直接解引用，或者进行自增自减操作
2. char\* 指针，一般代表 C 风格的字符串
3. 其他指针

delete 的优势：
1. 将 deleted 函数声明为 public 成员可以让编译器生成正确的编译报错，而不是产生因非 public 而导致的报错
2. 任何函数都可以被声明为 deleted 函数，而 private 只能修饰成员函数

### 12. 使用 override 修饰重载函数

### 13. 使用 const_iterator 替代 iterator

STL::const_iterator 是指向常量的迭代器指针。

### 14. 使用 noexcept 修饰不抛出异常的函数

`noexcept`保证函数不会抛出任何异常。

表明函数不抛异常的方法有两种：

```c++
void Func() throw(); // C++98 风格
void Func() noexcept; // C++11 风格，优化更好
```

### 15. 尽可能使用 constexpr

`constexpr`表示一个值不仅是常量，且它是编译期可知的。

### 16. 保证 const 成员函数线程安全

### 17. 理解特殊成员函数的生成

C++98 中特殊成员函数包含四个，这些函数仅在需要的时候被生成，它们都具有 public 访问层级，且都是 inline 的：

1. 默认构造函数
   
   - 当类里没有声明任何构造函数的时候，编译器会生成默认构造函数
2. 析构函数
   
   - 在 C++ 11 中默认是 noexcept 的
   - 当类是继承自某个含有虚函数的类时，编译器生成的析构函数也是虚函数
3. 拷贝构造函数
4. 拷贝赋值操作符

在 C++11 中新增了两个特殊成员函数：

1. 移动构造函数：使用形参 rhs 的各个非静态成员对自身的成员进行移动构造
   ```c++
   Widget(Widget &&rhs);
   ```

2. 移动赋值操作符：使用形参 rhs 的各个非静态成员对自身的成员进行移动赋值

   ```c++
   Widget &operator=(Widget &&rhs);
   ```


移动操作并不一定会真的发生，对于不支持移动操作的成员，会执行拷贝操作；一旦声明了移动操作，编译器就会删除拷贝操作。

两个拷贝操作相互独立，而若生成了两个移动操作的其中一个，编译器就会阻止另一个的生成。

Rule of Three：如果声明了**拷贝构造**函数，**拷贝赋值**操作符，和**析构函数**中的任何一个，就应该同时声明另外两个，因为如果有改写拷贝操作的需求，往往意味着该类需要进行某些资源管理，也就是说：

1. 在一种拷贝操作中进行的资源管理，很有可能在另一种拷贝操作中也需要进行
2. 析构函数也应该参与到资源管理中

标准库中用于进行内存管理的类都遵从 Rule of Three。

在 C++ 11 中，只要用户声明了**拷贝操作**或**析构函数**就会阻止**移动操作**的生成，因此，移动操作的生成条件是：

1. 类中未声明**拷贝操作**
2. 类中未声明**析构函数**
3. 类中未声明其他**移动操作**

同样的，C++ 11 中，**在已存在拷贝操作之一或析构函数的条件下，编译器自动生成其他拷贝操作**的行为已经被废弃。如果需要，可以用 ”= default“ 来显式地表达出默认实现式正确的。

成员函数模板不会抑制特殊成员函数的生成。

## 4. 智能指针

### 18. 使用 std::unique_ptr 管理具备专属所有权的资源

可以为 std::unique_ptr 设置自定义析构器；在使用默认析构器的前提下，可以认为 std::unique_ptr 和裸指针的尺寸相同；而如果析构器是函数指针，那么其尺寸会增加一到两个字长；如果析构器是函数对象，则带来的尺寸变化取决于该函数对象中存储了多少状态；无状态的函数对象（例如无捕获的 lambda 表达式）不会浪费任何存储尺寸。

std::unique_ptr 有两种形式：

1. std::unique_ptr\<T\> 单个对象，其对象种类不会产生二义性
2. std::unique_ptr<T[]> 数组，不提供提领操作符（* 和 ->）；可以使用 std::vector，std::array，std::string 替代数组智能指针

### 21. 优先使用 std::make_unique 和 std::make_shared 替代 new

## 5. 右值引用，移动语义和完美转发

### 23. 理解 std::move 和 std::forward

### 