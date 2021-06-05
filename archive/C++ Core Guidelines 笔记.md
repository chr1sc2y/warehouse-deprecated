### P.1: 直接在代码中表达想法

**原因**：编译器和许多程序员都不会阅读你的评论，或者设计文档。代码中的内容已经定义了语义，并且可以被编译器和其他工具检测。

**示例**

```c++
class Data {
public:
    Month month() const; // 好
    int month();		 // 差
};
```



### F.15：在传递参数时使用简单且常见的方法

普通的传参方法：

![Normal parameter passing table](http://isocpp.github.io/CppCoreGuidelines/param-passing-normal.png)

进阶的传参方法：

![Advanced parameter passing table](http://isocpp.github.io/CppCoreGuidelines/param-passing-advanced.png)

总结：

1. 对于仅用作输出的参数，使用 return value 的形式
2. 对于既用作输入又用作输出的参数，使用 pass by reference 的形式
3. 对于仅用作输入的参数
   - 如果拷贝操作的开销很小（比如普通数据类型），使用 pass by value 的形式

```c++
void f1(const string& s);  // 好: pass by reference to const 的开销很小

void f2(string s);         // 差: 拷贝的开销可能很大

void f3(int x);            // 好：很好的方法

void f4(const int& x);     // 差：引用会造成额外的开销
```



### F.60：当允许参数为空指针时，用 T* 替代 T&

当我们需要传递一个可能为空的指针时，使用 T* 更好。

构造出一个 nullptr 的引用是可行的，但不合法，这种错误非常罕见。

```c++
T *p = nullptr;
T &r = static_cast<T&>(*p);
```

使用 GDB 调试可以看到

```
(gdb) p p
$6 = (int *) 0x0
(gdb) p r
$7 = (int &) @0x0: <error reading variable>
(gdb) p &r
$8 = (int *) 0x0
```

### F.42：仅在指向位置时返回 T*

```c++
Node* Traverse(Node* t, int val)
{
    if (!t) return nullptr;
    if (t->val == val) return t;
    if ((auto p = Traverse(t->left, val))) return p;
    if ((auto p = Traverse(t->right, val))) return p;
    return nullptr;
}
```

返回指向某个位置的指针并不意味着发生了所有权转移。位置也可以用 iterator，indices 和 reference 表示，如果位置不能为空的话可以使用 T& 替代 T*。

### F.43：永远不要返回一个指向局部对象的指针或引用

返回一个 T* 悬空指针或 T& 悬空引用都可能造成程序崩溃和数据损坏。但 static 变量是静态分配的，所以返回一个 static 变量是不会造成悬空的。

### F.46：int main()

这是一条 C++ 的语言规则，将 main 声明为 void 会限制其可移植性。

### F.51：当可以选择时，优先使用默认参数而不是重载

```c++
// 默认参数
void Print(const string &s, format f = {});

// 重载
void Print(const string &s);
void Print(const string &s, format f);
```



# Google 代码规范

### 4.2 隐式类型转换

对于**转换运算符**和**单参数构造函数**，使用 `explicit` 关键字。

### 4.4 struct 和 class

仅对承载数据的被动对象使用 `struct`，其它一概使用 `class`。

### 4.5 继承

C++ 实践中，继承主要用于两种场合：`实现继承`，子类继承父类的实现代码；`接口继承`，子类仅继承父类的方法名称。

**优点**

- 继承可以复用基类代码，减少了代码量
- 继承是在编译时声明，程序员和编译器都可以理解相应操作并发现错误
- 接口继承可以强制类实现特定的 API，在类没有实现这些 API 时，编译器可以发现问题

**缺点**

- 对于实现继承，子类的实现代码散布在父类和子类之间，要理解其实现变得更加困难
- 子类不能重写父类的非虚函数，也不能修改其实现
- 基类也可能定义了一些数据成员，必须区分基类的实际布局

**结论**

- 只在”是一个“的情况下使用继承，只使用 `public` 继承
- 如果类里有虚函数，那么析构函数也应该是虚函数
- 数据成员都必须是`private`
- 对于重载的虚函数，使用 `override`或`final` 关键字显式地进行标记

### 5.1 输出参数

优先使用返回值作为函数输出，输入参数在前，输出参数在后。

### 5.3 引用参数

**结论**

- 如果函数需要修改变量的值，则参数必须声明为指针
- 输入参数是值参或 `const reference` 或 `const pointer`，输出参数是`pointer`
- 

