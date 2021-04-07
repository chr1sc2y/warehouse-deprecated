---
title: "C++ 基础知识整理"
date: 2015-07-05T08:57:52+10:00
draft: false
categories: ["C++"]
---

# C++ 基础知识

## const 相关

1. #define，typedef，const
    - \# 是宏，宏不做类型检查，只进行简单替换；在编译前被处理，编译阶段的程序是宏处理后的结果
    - typedef 用于声明自定义数据类型，简化代码
    - const 用于定义常量，有数据类型，编译器会对进行类型检查

2. const 和指针
    - const char *p: p is a pointer to const char
    - char const *p: p is a pointer to char const（同上）
    - char *const p: p is a const pointer to char

    ```c++
    int main() {
    const char *p1 = new char('a');
    char const *p2 = new char('b');
    char *const p3 = new char('c');
    *p1 = 'd'; // error: read-only variable is notassignable
    *p2 = 'e'; // error: read-only variable is notassignable
    p3 = new char('f'); // error: cannot assign tovariable 'p3' with const-qualified type 'char *const'
    p3 = nullptr;  // error: cannot assign to variable'p3' with const-qualified type 'char *const'
    }
    ```

3. const 和类
    - const 修饰类的成员函数时，该成员函数不能修改类的成员变量，不能调用类的非 const 成员函数
    - const 修饰函数参数时，在函数内部不能修改参数的值
    - const 修饰函数返回值时，接收返回值的变量也要用 const 修饰

    ```c++
    class Base {
    public:
        int i = 1;

        const int *Func(const int &j) const {
            const int *k = new int(1);
            i = 1; // error: cannot assign to non-static data member within const member function 'Func'
            j = 1; // error: cannot assign to variable 'j' with const-qualified type 'const int &'
            return k;
        }

        void Test() {
            i = 2;
        }
    };

    int main() {
        Base obj;
        int j = 1;
        int *k = obj.Func(j); // error: cannot initialize a variable of type 'int *' with an rvalue of type 'const int *'
        return 0;
    }
    ```

## static 相关

### 1. 面向过程

- 修饰全局变量时，该静态全局变量的作用域只在当前源文件
    - 全局变量的构造在 main 函数之前执行
- 修饰全局函数时，该静态全局函数的作用域只在当前源文件
- 修饰局部变量时，该静态局部变量的作用域只在当前函数中；该变量只在第一次执行到声明时被初始化，其内存被分配在全局数据区，当程序结束时才被回收

### 2. 面向对象

- 修饰类的成员变量时，该静态成员变量作用于类的所有实例对象，其内存被分配在全局数据区；该变量可以直接通过类名调用
- 修饰类的成员函数时，该静态成员函数不能操作类中的其他非静态成员函数；该函数可以直接通过类名调用

### 3. 类型转换

- static_cast
    - 不执类型检查；接近于 C 的强制类型转换，通常用于数值数据类型的转换，例如把 int 转换成 char，或把 void* 转换成其他类型的指针（不安全）
    - ，基本数据类型之间的转换，如把 int 转换成 char
    - 可以在整个类层次结构中移动指针，子类转化为父类安全（向上转换），父类转化为子类不安全（因为子类可能有不在父类的字段或方法）

- dynamic_cast
    - 运行时执行类型检查
    - 只适用于指针或引用，对不明确的指针的转换将失败（返回 nullptr），但不引发异常
    - 如果强制转换为引用类型失败，dynamic_cast 运算符会引发 bad_cast 异常
    - 用于多态类型的转换，可以在整个类层次结构中移动指针，包括向上转换、向下转换

- const_cast
    - 用于删除 const，volatile，__unaligned 特性，比如将 const int 类型转换为 int 类型

- reinterpret_cast
    - 用于位的简单重新解释
    - 允许将任何指针转换为任何其他指针类型，比如 char* 到 int*
    - 允许将任何整数类型转换为任何指针类型以及反向转换
    - 一个实际用途是在哈希函数中，通过让两个不同的值几乎不以相同的索引结尾的方式将值映射到索引

## 内存分配

1. 栈区 stack
    - 由编译器分配和释放，存储函数参数，局部变量等
    - 当系统剩余空间小于申请空间时，抛出异常提示栈溢出
    - 在一个进程中，位于用户虚拟地址空间顶部的是用户栈，编译器用它来实现函数的调用

2. 堆区 heap
    - 由程序分配和释放，例如使用 malloc, new
    - 堆向高地址扩展，是不连续的内存区域，大小可以灵活调整
    - 若程序不进行释放，则在程序结束时被回收
    - 系统维护一个记录空闲内存地址的链表；申请内存时，系统遍历该链表，寻找第一个空间大于所申请内存的堆结点，将该结点从链表中删除，并分配此节点的空间；这块内存的首地址会记录本次分配的大小，执行 delete 语句的时候根据记录释放内存空间；因为堆结点空间一般会大于申请的空间，系统会将多余的空间放入空闲的链表中

3. 自由存储区，存储由 malloc 等分配的内存，用 free 来回收

4. 全局/静态存储区，存储全局变量和静态变量
    - 在 C 语言中，全局变量分为初始化的和未初始化的，未被初始化的对象存储区可以通过 void* 来访问
    - 在 C++ 中它们共同占用同一块内存区域

5. 常量存储区，存储常量

6. new 分配失败
    - int* p = new (std::nothrow) int(1); 返回空指针
    - 用 try {} catch (const bad_alloc &b) {} 捕捉异常

### 内存分配方式

1. malloc：申请指定字节数的内存。申请到的内存中的初始值不确定
2. calloc：为指定长度的对象，分配能容纳其指定个数的内存。申请到的内存的每一位（bit）都初始化为 0
3. realloc：更改以前分配的内存长度（增加或减少）。当增加长度时，可能需将以前分配区的内容移到另一个足够大的区域，而新增区域内的初始值则不确定。
4. alloca：在栈上申请内存。程序在出栈的时候，会自动释放内存；alloca 不具可移植性, 而且在没有传统堆栈的机器上很难实现。alloca 不宜使用在必须广泛移植的程序中。

## 指针和引用

1. void *
    - void * 是一个指针，它指向了内存里的某个区域，但是它所指向的内存区域没有任何类型信息或辅助信息，所以它可以随意地解读其指向的内存区域内的数据，它可以按 int 类型来解读，也可以按照 double 类型来解读；在使用前需要告知系统从其指向的内存区域取出多少个字节
    - 不能对void * 进行解引用操作

2. 区分指针类型
    - int *p[10]
        - int *p[10] 表示指针数组，强调数组概念，是一个数组变量，数组大小为 10，数组内每个元素都是指向 int 类型的指针变量
    - int (*p)[10]
        - int (*p)[10] 表示数组指针，强调是指针，只有一个变量，是指针类型，不过指向的是一个 int 类型的数组，这个数组大小是 10
    - int *p(int)
        - int *p(int) 是函数声明，函数名是 p，参数是 int 类型的，返回值是 int * 类型的
    - int (*p)(int)
        - int (*p)(int) 是函数指针，强调是指针，该指针指向的函数具有 int 类型参数，并且返回值是 int 类型的

3. 数组和指针

    ```
    int a[10];
    int (*p)[10] = &a;
    ```
    
    - a是数组名，也是数组首元素的地址，+1 表示地址值加上一个int类型的大小，如果 a 的值是 0x00000001，加 1 操作后变为 0x00000005，*(a + 1) = a[1]
    - &a 是数组的地址，其类型为int (*)[10]，+1 表示数组首地址加上整个数组的偏移（10个int型变量），值为数组a尾元素后一个元素的地址
    - 若 (int *)p ，此时输出 *p 时，其值为 a[0] 的值，因为被转为int * 类型，解引用时按照int类型大小来读取

4. 数组名和指向数组首元素的指针的区别

    - 均可通过增减偏移量来访问数组中的元素
    - 数组名不是真正意义上的指针，可以理解为常指针，所以数组名没有自增、自减等操作
    - 当数组名当做形参传递给调用函数后，就失去了原有特性，退化成一般指针，多了自增、自减操作，但sizeof运算符不能再得到原数组的大小了

5. 野指针

    - 空悬指针，是指向垃圾内存的指针
    - 产生原因
        - 指针变量未初始化
        - 指针 free 或 delete 之后没有置空

6. 常引用
    - 常引用类似于常量指针，const typename &refname = varname
    - 使用常引用时，原变量的值不会被常引用所修改
    - 常引用通常用作只读变量别名或是形参传递

## 智能指针

### unique_ptr

- unique_ptr 实现独占式拥有（exclusive ownership）或严格拥有（strict ownership）概念，保证同一时间内只有一个智能指针可以指向该对象
- 一旦拥有者被销毁，或拥有了另一个对象，之前拥有的那个指针对象就会被销毁，相应的资源会被释放
- 可以移交拥有权
- 用于避免内存泄漏（resource leak），比如 new 后忘记 delete

### shared_ptr

- shared_ptr 实现共享式拥有（shared ownership）概念
- 多个智能指针指向相同对象，该对象和其相关资源会在最后一个 reference 被销毁时被释放；需要使用 weak_ptr、bad_weak_ptr 和 enable_shared_from_this 等辅助类来实现
- 支持定制型删除器（custom deleter）
- 可防范 Cross-DLL 问题（对象在动态链接库 DLL 中被 new 创建，却在另一个 DLL 内被 delete 销毁），自动解除互斥锁

### weak_ptr

- weak_ptr 允许共享但不拥有某对象
- 一旦最末一个拥有该对象的智能指针失去了所有权，任何 weak_ptr 都会自动成空（empty）
- 因此，在 default 和 copy 构造函数之外，weak_ptr 只提供 接受一个 shared_ptr 的构造函数
- 可打破环状引用（cycles of references，两个其实已经没有被使用的对象彼此互指，使之看似还在 “被使用” 的状态）的问题

### auto_ptr

- 缺乏语言特性，比如针对构造和赋值的 std::move 语义
- auto_ptr 与 unique_ptr 对比
    - auto_ptr 可以赋值拷贝，复制拷贝后所有权转移；unqiue_ptr 无拷贝赋值语义，但实现了move 语义
    - auto_ptr 对象不能管理数组（析构调用 delete），unique_ptr 可以管理数组（析构调用 delete[] ）

## Lambda 表达式

### 1. Lambda 表达式的形式

```[capture list] (params list) mutable exception-> return type { function body }```

- capture list：捕获外部变量列表
- params list：形参列表
- mutable：指示符，用来指明是否可以修改捕获的变量
- exception：异常设定
- return type：返回类型
- function body：函数体
- 只有捕获外部变量列表和函数题是必须有的

### 2. 捕获外部变量列表

- Lambda 表达式可以使用其可见范围内的外部变量，但必须明确声明哪些外部变量可以被该 Lambda 表达式使用
- Lambda 表达式通过在最前面的方括号 [] 来指明其内部可以访问的外部变量，这一过程也称为 Lambda 表达式捕获了外部变量，类似于参数传递
- 外部变量的捕获方式有三种
    - 值捕获
        - 值捕获和参数传递中的值传递类似，被捕获的变量的值在 Lambda 表达式创建时通过值拷贝的方式传入，在函数体内对该变量的修改不会影响外部的值
    - 引用捕获
        - 使用 &a 的方式传递引用，值捕获和引用捕获都要显式地列出外部变量
    - 隐式捕获
        - 让编译器根据函数体中的代码来推断需要捕获哪些变量，这种方式称之为隐式捕获
        - 隐式捕获有两种方式，分别是 [=] 和 [&]；[=] 表示以值捕获的方式捕获外部变量，[&] 表示以引用捕获的方式捕获外部变量。
- 修改捕获变量
    - 如果以传值方式捕获外部变量，在函数体中将不能修改该外部变量，否则会引发编译错误
    - 使用 mutable 关键字可以修改值捕获的变量

### 3. 形参列表

- Lambda 表达式中传递参数时有一些限制
    - 参数列表中不能有默认参数
    - 不支持可变参数
    - 所有参数必须有参数名

## RTTI

- Runtime Type Identification 运行时类型识别

### 1. 目的

- 让程序在运行时根据基类的指针或引用来获得该指针或引用所指的对象的实际类型
- 通过 typeid 操作符识别出所有的基本类型的变量对应的类型

### 2. 使用

- typeid 运算符，该运算符返回其表达式或类型名的实际类型
- dynamic_cast 运算符，该运算符将基类的指针或引用安全地转换为派生类类型的指针或引用



## 模板

- 实现范型编程
    ```template <class type> ret-type func-name(parameter list) { }```
    
- 模板类中可以使用虚函数
- 模板类的成员函数不能是虚函数

## 关键字

1. volatile
    - 用 volatile 关键字声明的变量可能会被某些未知的因素更改
    - volatile 变量在被访问时，编译器都会从内存地址中取出它的值；非 volatile 修饰的变量由于编译器优化，可以直接从 CPU 寄存器中取值
    - 没有用 volatile 关键字声明的变量在被访问时，编译器可能直接从 CPU 的寄存器中取值（因为变量之前被访问过，之前从内存中值保存在某个寄存器中）

2. inline
    - 内联函数在编译时被展开，不执行进入函数的步骤，直接执行函数体
    - 编译器一般不内联包含循环、递归、switch 等复杂操作的内联函数
    - 在类声明中定义的函数，除了虚函数的其他函数都会自动隐式地当成内联函数
    - 优点
        - 内联函数在调用处进行展开，省去了参数压栈、栈帧开辟与回收，结果返回等
        - 内联函数安全检查或自动类型转换，而宏不会
        - 在类中声明并定义的成员函数，会被隐式地转化为内联函数
    - 缺点
        - 代码膨胀；内联是以代码膨胀（复制）为代价，将使程序的总代码量增大，消耗更多的内存空间
        - inline 函数无法随着函数库升级而升级。inline函数的改变需要重新编译，不像 non-inline 可以直接链接

3. friend 友元
    - 友元类和友元函数能访问其他类地私有成员
    - 破坏封装性，不可传递，单向性

4. decltype
    - 获取变量类型
    - 检查实体的声明类型或表达式的类型及值分类

## C++ 编译

- 编译预处理（.i）
- 编译优化（.s）
- 汇编程序（.obj、.o）
- 链接（可执行文件）

### 1. 编译：把文本形式的源代码翻译成机器语言，并形成目标文件

1. 编译预
    - 编译器执行预处理指令（以#开头，例如#include），包括拷贝 #include 包含的文件代码，进行 #define 宏定义的替换，处理条件编译指令（#ifndef #ifdef #endif）等
    - 生成 .i 文件

2. 编译
    - 语法分析，词法分析，语义分析，中间代码生成，代码优化，代码生成等
    - 翻译成汇编代码
    - 生成 .s 文件

3. 汇编
    - 翻译成机器指令
    - 生成 .o 目标文件
    - 目标文件通常至少有两个段
        - 代码段：包换主要程序的指令。该段是可读和可执行的，一般不可写
        - 数据段：存放程序用到的全局变量或静态数据。可读、可写、可执行

### 2. 链接 ：把目标文件和操作系统的启动代码和库文件组织起来形成可执行程序

- 将有关的目标文件连接起来
- 原因
    - 在程序中调用了某个库函数
    - 某个源文件调用了另一个源文件中的函数或常量
- 生成可执行文件

## C++ 函数调用

1. 参数入栈
    - 把参数从右往左 push 进入栈中
2. 保存现场（返回地址入栈）
    - 将当前代码区调用指令的下一条指令地址 push 入栈中，供函数返回时继续执行
3. 执行子函数
    - 代码区跳转：处理器从当前代码区跳转到被调用函数的入口处
    - 栈帧调整，包括
        - 保存当前栈帧状态值，已备后面恢复本栈帧时使用（EBP入栈）
        - 将当前栈帧切换到新栈帧（将ESP值装入EBP，更新栈帧底部）
        - 给新栈帧分配空间（把ESP减去所需空间的大小，抬高栈顶）
4. 恢复现场

## STL 容器

### 所有容器

|容器|实现|查询|插入删除|特点|
|---|---|---|---|---|
|array|数组|O(1)|O(1)|大小固定|
|vector|数组|O(1)|尾部 O(1)，其他 O(n)|大小可变，扩容耗时|
|deque|双端队列|O(n)|头尾 O(1)，其他 O(n)|一个中央控制器，多个缓冲区|
|list|双向链表|O(n)|O(1)| |
|forward_list|单向链表|O(n)|O(1)| |
|stack|deque / list| / |O(1)|先进后出|
|queue|deque / list| / |O(1)|先进先出|
|priority_queue|vector| / |O(logn)|堆，完全二叉树|
|set|红黑树|O(logn)|O(logn)| |
|multiset|红黑树|O(logn)|O(logn)| |
|map|红黑树|O(logn)|O(logn)| |
|multimap|红黑树|O(logn)|O(logn)| |
|unordered_set|哈希表|平均 O(1)|平均 O(1)| |
|unordered_multiset|哈希表|平均 O(1)|平均 O(1)| |
|unordered_map|哈希表|平均 O(1)|平均 O(1)| |
|unordered_multimap|哈希表|平均 O(1)|平均 O(1)| |

### vector

1. vector 的内存释放

    - 对于vector，string 等容器，执行 clear() 函数后只会将其 size 置为 0，而不会改变它们的 capacity
    - 可以用 swap() 函数来清理 vector 和 string 容器的内存，用一个右值 vector 来替换需要清理的容器
    - 其它的 stl 容器调用 clear() 时会清空内存
    - 测试

    ```c++
    vector<int> v;
    printf("v.size() = %d, v.capacity() = %d\n", v.size(), v.capacity());
    for (int i = 0; i < 100000; ++i)
        v.push_back(i);
    printf("v.size() = %d, v.capacity() = %d\n", v.size(), v.capacity());
    vector<int>().swap(v);
    printf("v.size() = %d, v.capacity() = %d\n", v.size(), v.capacity());

    /*
    output:
    v.size() = 0, v.capacity() = 0
    v.size() = 100000, v.capacity() = 131072
    v.size() = 0, v.capacity() = 0
    */
    ```

2. resize 报错
    - vector 的类型是自定义类或结构体的时候，在执行 resize() 的时候如果参数大于当前的 size，编译器会将超过当前 size 的部分初始化；如果自定义的类没有构造函数那么编译器会报错

### 红黑树

1. 特征
    - 节点是红色或黑色
    - 根是黑色
    - 所有叶子（空节点）都是黑色
    - 每个红色节点必须有两个黑色的子节点；从每个叶子到根的所有路径上不能有两个连续的红色节点
    - 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点

2. 非严格平衡，通过变色，左旋，右旋来调整

## 左值和右值

1. 左值可以取地址；右值是将要销毁的临时变量

2. 右值的生命周期在表达式结束时结束，常量左值引用会延长临时变量的生命

3. 左值引用
    - 普通引用，一般表示对象的身份/别名

4. 右值引用
    - 必须绑定到右值的引用，一般表示对象的值
    - 右值引用可实现转移语义（Move Sementics）和精确传递（Perfect Forwarding）
    - 主要目的是消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率
    - 能够更简洁明确地定义泛型函数

## 排序

|排序算法|平均时间复杂度|最差时间复杂度|空间复杂度|数据对象稳定性|
|---|---|---|---|---|
|冒泡排序|O(n2)|O(n2)|O(1)|稳定|
|选择排序|O(n2)|O(n2)|O(1)|数组实现不稳定|
|插入排序|O(n2)|O(n2)|O(1)|稳定|
|快速排序|O(n*log2n)|O(n2)|O(log2n)|不稳定|
|堆排序|O(n*log2n)|O(n*log2n)|O(1)|不稳定|
|归并排序|O(n*log2n)|O(n*log2n)|O(n)|稳定|
|希尔排序|O(n*log2n)|O(n2)|O(1)|不稳定|
|计数排序|O(n+m)|O(n+m)|O(n+m)|稳定|
|桶排序|O(n)|O(n)|O(m)|稳定|
|基数排序|O(k*n)|O(n2)| |稳定|

## 其他

### 重载操作符

1. new

    ```c++
    void *operator new(size_t size)
    {
        cout << "in threee_d new\n";
        return malloc(size);
    }
    ```
    - throw(std::bad_alloc)
    - const std::nothrow_t& 不抛出异常

## 参考

- 《C++ Primer》
- 《Effective C++》
- 《More Effective C++》
- 《深度探索 C++ 对象模型》
- 《深入理解 C++11》
- 《STL 源码剖析》

- 《剑指 Offer》
- 《编程珠玑》
- 《程序员面试宝典》

- 《深入理解计算机系统》
- 《Windows 核心编程》
- 《Unix 环境高级编程》

- 《Unix 网络编程》
- 《TCP/IP 详解》

- 《程序员的自我修养》