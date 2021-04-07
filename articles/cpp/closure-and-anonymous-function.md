# C++ 闭包和匿名函数

[TOC]

本文主要介绍了 C++ 中闭包和仿函数，以及匿名函数相关的概念。

## 1 闭包和仿函数

**闭包**（[Closure](https://en.wikipedia.org/wiki/Closure_(computer_programming))）可以被理解为一个附带数据的操作，WikiPedia 对闭包的定义是 *"In programming languages, a **closure**, also **lexical closure** or **function closure**, is a technique for implementing **lexically scoped name binding** in a language with **first-class functions**."*，其中有两层含义：

1. 词法作用域（[lexically scoped](https://en.wikipedia.org/wiki/Lexically_scoped)）的名字绑定（[name binding](https://en.wikipedia.org/wiki/Name_binding)）：在词法作用域（C++ 的词法作用域是静态绑定的，包括块、函数、类、命名空间、全局作用域等）中，变量名与其词法上下文的标识符相关联，而独立于运行时的调用栈；
2. 函数被当作头等公民（[first-class citizen](https://en.wikipedia.org/wiki/First-class_citizen)）：在运行时可以构造一个函数对象并将其作为参数传递给其他函数；

显然 C++ 98 并不符合这两点定义，因此 C++ 98 中并没有严格意义上的闭包，但我们可以用仿函数（[Functor](https://www.geeksforgeeks.org/functors-in-cpp/)）来模拟闭包的行为；仿函数即一个重载了小括号操作符的类，这个类拥有与函数相近的行为方式，它拥有自己的私有成员变量，例如：

```cpp
class Adder
{
public:
    int operator()(int num)
    {
        sum += num;
        return sum;
    }

    Adder() : sum(0) {}
    Adder(int num) : sum(num) {}
private:
    int sum;
};
   
int main()
{
    Adder adder(0);
    cout << adder(1) << endl;
    cout << adder(2) << endl;
    cout << adder(3) << endl;
}
```

```shell
$ g++ -std=c++98 -o adder adder.cpp 
$ ./adder 
1
3
6
```

相比之下 golang 中真正的闭包显得简洁很多：

```go
func adder() func(int) int {
	sum := 0
	return func(num int) int {
		sum += num
		return sum
	}
}

func main() {
	numAdder := adder()
	fmt.Println(numAdder(1))
	fmt.Println(numAdder(2))
	fmt.Println(numAdder(3))
}
```

```shell
$ go run main.go 
1
3
6
```

C++ 98 的标准库中提供了很多实用的函数，例如 `std::sort`，当我们需要定制其排序规则的时候，也可以定义一个简单的仿函数（或者普通的函数）作为参数传入 ，注意定义排序规则的时候要满足 [Strict Weak Ordering](https://www.boost.org/sgi/stl/StrictWeakOrdering.html)：

```cpp
struct Foo
{
    int a_, b_;
    Foo(int a, int b) : a_(a), b_(b) {}
};

struct FooComparatorGreater
{
    bool operator()(const Foo f1, const Foo f2)
    {
        if (f1.a_ != f2.a_)
            return f1.a_ > f2.a_;
        return f1.b_ > f2.b_;
    }
};

int main()
{
    vector<Foo> foo{ Foo(3, 6), Foo(9, 2), Foo(9, 8) };
    sort(foo.begin(), foo.end(), FooComparatorGreater());
    for (const auto& f : foo)
        cout << f.a_ << ' ' << f.b_ << endl;
    return 0;
}
```

 ```shell
$ g++ -std=c++11 -o sort-functor sort-functor.cpp 
$ ./sort-functor 
9 8
9 2
3 6
 ```

## 2 匿名函数

**匿名函数**（[Anonymous Function](https://en.wikipedia.org/wiki/Anonymous_function)）起源于第一个[函数式编程](https://en.wikipedia.org/wiki/Functional_programming)语言 [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)#:~:text=Lisp%20(historically%20LISP)%20is%20a,is%20older%2C%20by%20one%20year.)，C++ 11 标准中正式引入了匿名函数，也叫做 lambda 表达式（[Lambda Expression](https://en.cppreference.com/w/cpp/language/lambda)）；匿名函数是一种没有被绑定标识符的函数，可以用于很方便地定义一个临时的函数对象，或作为一个函数对象传递给更上层的函数（例如 [`std::for_each`](https://en.cppreference.com/w/cpp/algorithm/sort)），其在 C++ 11 的语法上表现得非常轻量级，不需要像普通的**具名函数**一样单独在头文件中作出声明，且**符合闭包的定义**。

匿名函数可以替代掉复杂且冗余的仿函数，使得代码更易于理解和维护：

```c++
sort(foo.begin(), foo.end(), [](const Foo& f1, const Foo& f2)
{
    return f1.a_ != f2.a_ ? f1.a_ > f2.a_ : f1.b_ > f2.b_;
});
```

匿名函数由以下几个部分组成，其中只有 1, 2, 6 三个部分是必须的，其余部分可以省略：

![lambda-expression-syntax](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/cpp/lambda-expression-syntax.png)

1. 捕获子句 capture clause / lambda introducer
2. 参数列表 parameter list / lambda declarator
3. 可变规格 mutable specification
   - 被 mutable 修饰的匿名函数可以修改按值捕获的变量
4. 异常规格 exception specification
5. 尾随返回类型 trailing-return-type
6. 匿名函数体 lambda body

### 2.1 捕获子句

捕获子句用于捕获外部变量，使得匿名函数体可以使用这些变量，捕获的方法分为引用捕获和值（拷贝）捕获两种，使用方法如下：

1. `[]` 不捕获任何变量；

2. `[&]` 按引用捕获所有外部变量；

3. `[=]` 按值捕获所有外部变量

4. `[&, var]` 默认按引用捕获，仅按值捕获 var；

5. `[=, &var]` 默认按值捕获，仅按引用捕获 var；

6. `[y, y]` 重复按值捕获同一个变量，没有意义，会报 warning；

7. `[&, &var]` 默认按引用捕获，并按引用捕获 var，没有意义，会报 warning；

8. `[=, this]` 默认按值捕获，并按值捕获 this 指针，没有意义，同样会报 warning；

   ```cpp
   std::function<void()> AnonyFunc = [=, this]() -> void {};//  warning: explicit by-copy capture of ‘this’ redundant with by-copy capture default
   ```

9. `[this]` 按值捕获 this 指针，this 指针虽然不能被修改，但其指向的对象可以被操作并修改，相当于按引用捕获了 this 指向的对象，即 `[&(*this)]`；

   ```cpp
   class Foo
   {
   public:
       void Func()
       {
           int y{ 0 };
           std::function<void()> AnonyFunc = [this]() -> void
           {
               x_ = 2; // ok，x_ 是类的成员变量，可以被修改
               y = 2; // error: ‘y’ is not captured，函数的局部变量并没有被捕获
               this = nullptr; // error: lvalue required as left operand of assignment，这里捕获的 this 指针是一个临时变量即右值，不能被修改
           };
           AnonyFunc();
       }
   
   private:
       int x_ = 0;
   };
   ```

10. `[*this]` 在 C++ 11 中不能按值捕获 this 指针指向的对象；

    ```cpp
    std::function<void()> AnonyFunc = [*this]() -> void {}; // error: expected identifier before ‘*’ token
    ```

在使用捕获子句的时候，需要注意一些问题：

1. 不建议使用 2，3 这两种方式进行捕获（对性能影响较大），应该明确地指出需要按引用捕获的变量；

2. 按值捕获的变量是 read-only (const) 的，只有当匿名函数的可变规格被显式声明为 `mutable` 的时候才可以修改按值捕获的变量；

   ```cpp
   int x{ 0 };
      
   auto AnonyFunc = [=]() -> void
   {
       x = 1; // error: assignment of read-only variable ‘x’
   }
      
   auto AnonyFunc = [=]() mutable -> void
   {
       x = 1; // ok
   }
   ```

3. 按值捕获的变量的值在匿名函数生成的时候就已经确定了，如果在匿名函数生成后修改外部变量的值，则不会影响到匿名函数内被捕获的变量值，因为它们是两个作用域不同的变量：

   ```cpp
   int i{ 0 };
   auto AnonyFunc = [i]() -> void
   {
       cout << i << endl;
       cout << &i << endl;
   };
   i = 1;
   cout << i << endl;
   cout << &i << endl;
   AnonyFunc();
   ```

   ```shell
   $ g++ -std=c++11 -o lambda-capture lambda-capture.cpp 
   $ ./lambda-capture 
   1
   0x7ffe31fced8c
   0
   0x7ffe31fced80
   ```

4. 对于按引用捕获的变量（或按值捕获的指针），如果该引用变量（或指针指向的对象）在外部被析构，那么匿名函数中的引用变量（或指针）则会成为**悬空引用/指针**（[Dangling Pointer](https://en.wikipedia.org/wiki/Dangling_pointer)）：

   ```cpp
   int* x = new int[1000000];
   x[0] = 0;
   auto AnonyFunc = [&x]()
   {
       x[0] = 1; // Segmentation fault
   };
   delete[] x;
   AnonyFunc();
   ```

   ```cpp
   struct Foo
   {
       int x_[1000000];
   };
      
   int main()
   {
       Foo* f = new Foo();
       f->x_[0] = 0;
       auto AnonyFunc = [f]() -> void
       {
           f->x_[0] = 1; // Segmentation fault
       };
       delete f;
       AnonyFunc();
   }
   ```

### 2.2 匿名函数和闭包

*Scott Meyers* 对 lambda 表达式（匿名函数）与闭包之间的关系的解释是 *"The distinction between a lambda and the corresponding closure is precisely equivalent to the distinction between a class and an instance of the class. A class exists only in source code; it doesn’t exist at runtime. What exists at runtime are objects of the class type. Closures are to lambdas as objects are to classes. This should not be a surprise, because each lambda expression causes a unique class to be generated (during compilation) and also causes an object of that class type–a closure–to be created (at runtime)."*；

这段解释可以拆分为两段：

1. 匿名函数和闭包的关系就如同类和类对象的关系，匿名函数和类的定义都只存在于源码（代码段）中，而闭包和类对象则是在运行时占用内存空间的实体；
2. 对匿名函数的定义会生成一个独一无二的类，并在运行时生成其类对象；

再结合 C++ 11 的标准说明：

- *"[C++11: 5.1.2/3]: The type of the lambda-expression (which is also the type of the closure object) is a unique, unnamed non-union class type — called the closure type..."*，C++ 11 中的匿名函数实际上也是用类（closure type）来实现的；
- *“[C++11: 5.1.2/5]: The closure type for a lambda-expression has a public inline function call operator (13.5.4) whose parameters and return type are described by the lambda-expression’s parameter-declaration-clause and trailing-return-type respectively. [..]”*，匿名函数生成的类中也重载了 `operator()`，其参数与匿名函数的参数列表相同，返回值与匿名函数的尾随返回类型相同；
- *"[C++11: 5.1.2/6]: The closure type for a lambda-expression with no lambda-capture has a public non-virtual non-explicit const conversion function to pointer to function having the same parameter and return types as the closure type’s function call operator. The value returned by this conversion function shall be the address of a function that, when invoked, has the same effect as invoking the closure type’s function call operator."*，如果匿名函数没有任何参数，那么将会生成一个普通的函数，而不是闭包类型；

可以知道实际上匿名函数也是用仿函数实现的，它实际上是 C++ 11 加入的语法糖，不过其语法特性是符合闭包定义的。

## 3 匿名函数在 C++ 14 及之后的变化

###  C++ 14 广义捕获

C++ 14 中引入了新的**广义 lambda 捕获**（[Generalized Lambda Captures](https://en.wikipedia.org/wiki/C%2B%2B14#Generic_lambdas)），即可以在捕获列表中以任意方式初始化匿名函数中的变量，使得某些被禁用了拷贝构造函数的类型可以通过 `std::move` 的方式被捕获到匿名函数中：

```cpp
auto ptr_0 = make_unique<int>( 0 );
auto AnonyFunc = [ptr_0 = move(ptr_0)]()
{
    *ptr_0 = 1;
    cout << *ptr_0 << endl;
};
AnonyFunc();
```

这里捕获列表中左边和右边的 `ptr_0` 不是同一个变量，它们的作用域分别是匿名函数内和匿名函数外；

除此之外广义 lambda 捕获还可以用来间接地捕获 `*this`，即在 C++ 11 中无法实现的按值捕获 this 指向的对象：

```cpp
auto AnonyFunc = [this_copy = *this]() mutable
{
    this_copy.x_ = 1;
    cout << this_copy.x_ << endl;
};
AnonyFunc();
```

### C++ 17 捕获 `*this`

在 C++ 17 中，终于可以直接捕获 `*this` 了，[提案 P0018R3](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0018r3.html) 指出捕获 `*this` 可以用于需要进行异步操作的并发应用，因为 `this` 可能失效：

```cpp
auto AnonyFunc = [*this]() mutable
{
    x_ = 1;
    cout << x_ << endl;
};
AnonyFunc();
cout << x_ << endl;
```

```shell
$ g++ -std=c++17 -o lambda lambda.cpp 
$ ./lambda 
1
0
```

