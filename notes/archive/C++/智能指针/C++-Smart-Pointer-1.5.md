---
title: "C++ 智能指针（1.5）：move 语义"
date: 2019-01-02T09:55:05+11:00
draft: false
categories: ["C++"]
---

# C++智能指针（1.5）：move语义

## move语义

### 定义

右值引用（Rvalue Referene）是 C++ 11中引入的新特性，它实现了转移语义（Move Sementics）和精确传递（Perfect Forwarding），其主要目的有

- 消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。
- 能够更简洁明确地定义泛型函数。

### 实现

move 语义的实现非常简单，它将传入的参数 _Tp&& __t 使用静态类型转换 static_cast<_Up&&>(__t) 转变成了成了对应类型的右值，也就是说使用 move 语义之后，编译器窃取（一般会在移动构造函数和移动赋值操作符里将原有对象指向 nullptr）了原有对象的右值，并延长了这个右值的生命周期并将其用来赋值给其他的对象，而没有对右值做任何拷贝操作。

```c++
template <class _Tp>
typename remove_reference<_Tp>::type&&
move(_Tp&& __t) _NOEXCEPT
{
    typedef typename remove_reference<_Tp>::type _Up;
    return static_cast<_Up&&>(__t);
}
```

### 测试

定义一个 Object 类和一个 MoveObject 函数使用 move 语义返回一个 Object 的类对象，可以看到在 MoveObject 函数返回右值后，obj 对象调用了移动构造函数。

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

```c++
Object MoveObject() {
    Object obj;
    return move(obj);
}

int main() {
    Object obj = MoveObject();
    return 0;
}
/*
output:
Construct
Move
Destruct
Destruct
*/
```

### 返回值优化（RVO，Return value optimisation）

返回值优化是一种编译器优化技术，允许编译器在调用点（[call site](https://en.wikipedia.org/wiki/Call_site)）直接构造函数的返回值。

定义一个 CopyObject 函数返回一个 Object 的类对象，原本 Function 函数在返回时应该会进行一次拷贝，然而调试结果却显示 obj 对象只在 Function 函数中被构造了一次，在程序结束时被析构了一次。这是因为编译器使用了 RVO 机制，这里 Function 函数返回的是一个左值，所以又称命名返回值优化（NRVO），在 C++ 11 里被称为拷贝省略（Copy Elision）。

```c++
Object CopyObject() {
    Object obj;
    return obj; // NRVO (named return value optimisation)
}

int main() {
    Obj obj = Function();
    return 0;
}
/*
output:
Construct
Destruct
 */
```

如果在 MoveObject 函数中使用move语义进行返回，函数实际返回的是一个右值引用（Obj&&），而不是函数定义中的对象（Obj），没有触发 RVO。也就是说，要触发 RVO 机制，必须保证函数实际的返回值类型和函数定义中的返回值类型一致。

```c++
Object MoveObject() {
    Object obj;
    return move(obj);
}

int main() {
    Object obj = MoveObject();
    return 0;
}
/*
output:
Construct
Move
Destruct
Destruct
*/
```

如果把函数返回值类型也改为右值引用，那么 main 函数中的 obj 对象也会使用移动构造函数，触发 RVO 机制。

```c++
Object &&MoveObject() {
    Object obj;
    return move(obj);
}

int main() {
    Object obj = MoveObject();
    return 0;
}
/*
output:
Construct
Destruct
Move
Destruct
*/
```

在 CopyObject 函数返回时增加判断条件，会发现其返回时也会调用移动构造函数，而没有触发 RVO 机制。没有触发 RVO 机制是因为编译器会使用父堆栈帧（parent stack frame）来避免返回值拷贝，但如果在返回时使用了判断语句，编译器在编译时将不能确定将哪一个作为返回值，因此不会触发 RVO 机制；而调用移动构造函数是因为在使用左值返回时编译器会优先使用移动构造函数，不支持移动构造时才调用拷贝构造函数。

```c++
Object CopyObject(bool flag) {
    Object obj1, obj2;
    if (flag)
        return obj1;
    return obj2;
}

int main() {
    Object obj = CopyObject(true);
    return 0;
}
/*
output:
Construct
Construct
Move
Destruct
Destruct
Destruct
*/
```

## 参考

[RVO V.S. std::move](https://www.ibm.com/developerworks/community/blogs/5894415f-be62-4bc0-81c5-3956e82276f3/entry/RVO_V_S_std_move?lang=en)

[右值引用与转移语义](https://www.ibm.com/developerworks/cn/aix/library/1307_lisl_c11/index.html)
