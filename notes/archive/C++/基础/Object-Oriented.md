---
title: "C++ 面向对象"
date: 2015-07-05T08:57:00+08:00
draft: true
categories: ["C++"]
---

# C++ 面向对象

## 多态

### 多态类型

1. 重载多态（Ad-hoc Polymorphism，编译期）：函数重载、运算符重载
2. 子类型多态（Subtype Polymorphism，运行期）：虚函数
3. 参数多态性（Parametric Polymorphism，编译期）：类模板、函数模板
4. 强制多态（Coercion Polymorphism，编译期/运行期）：基本类型转换、自定义类型转换

### 类和虚函数

1. class 和 struct

    - C 中 struct 是一种复合数据类型，只能定义成员变量
    - C++ 中 struct 和 class 都拥有继承，虚函数等所有特性
    - class 的默认访问权限是 private，struct 的默认访问权限是 public

2. 访问权限

    - public：可以通过类的成员函数，子类的成员函数，友元函数，类对象访问
    - protected：可以通过类的成员函数，子类的成员函数，友元函数访问，不能通过类对象访问
    - private：只能通过类的成员函数和友元函数访问
    - 基类的 public 变量无论通过哪一种继承方式被继承后都能够被访问，其访问权限与继承方式相同
    - 基类的 private 变量无论通过哪一种继承方式被继承后都无法被访问
    - 基类的 protected 变量无论通过哪一种继承方式被继承后都能够被访问，通过 public 和 protected 方式被继承后访问权限仍为 protected，通过 private 方式被继承后访问权限为 private

3. 纯虚函数
    - 纯虚函数是在在基类中没有定义的虚函数，这样的基类叫做抽象类，如果其派生类对应的虚函数仍没有定义，那么其派生类也是抽象类；抽象类不能被实例化
    - 纯虚函数的表示需要在函数声明后加上 "=0"，例如 virtual void Func() = 0;
    - 纯虚函数的目的
        - 应用多态特性
        - 定义统一的接口
        - 实现动态单分派子类型多态（Dynamic Single-dispatch Subtype Polymorphism）

4. 虚函数

    - 对于类的普通成员函数，它在内存中的地址是唯一的，而不会被存储于每个类的实例对象中，也就相当于是带有第一个参数是 this 指针的普通函数；当类对象调用其普通成员函数时，编译器先找到该函数的地址，再将该类对象的 this 指针传递执行；这个过程叫静态联编，是在编译阶段完成的
    - 对于类的虚函数（virtual function），它的地址被存储于每个类对象的虚表（virtual table）中；当类对象调用其虚函数时，编译器先从虚表中查询到要执行的虚函数的地址，再将该类对象的 this 指针传递执行；这个过程叫动态联编，是在运行时完成的
    - 每个类都会维护一个唯一的虚表，编译器会在编译时创建虚表；当类对象被构造时，虚表的地址会被写入类对象内存地址的起始位置；每一个类都有自己独特的虚表
    - 在一个继承链上，编译器会为基类插入一个隐式的指针，指向虚表，称为 __vptr；子类继承父类时，会继承这个 __vptr. 于是，当构建具体的类对象时，若是基类对象，__vptr 就会指向基类的 vtable，若是派生类对象，__vptr 就会指向子类的 vtable
    - 同一个类的所有对象共享同一个虚函数表；基类和派生类的虚函数表不同
    - 测试
        - 有虚函数时，类对象地址的起始位置应该是其虚函数的地址

        ```c++
        class Base {
            int f = 0x2f;
            virtual void Func() {
                printf("Func\n");
            }
        };

        int main() {
            auto a = new class Base();
            printf("%x\n", *(int *) (void *) a);
            for (int i = 0; i < 10; i++) {
                auto b = new class Base();
                printf("%x\n", *(int *) (void *) b);
                delete b;
            }
            return 0;
        }

        
        // output
        ab38310
        ab38310
        ab38310
        ab38310
        ab38310
        ab38310
        ab38310
        ab38310
        ab38310
        ab38310
        ab38310

        ```

        - 没有虚函数时，类对象地址的起始位置应该是其成员变量

        ```c++
        class Base {
            int f = 0x2f;
            void Func() {
                printf("Func\n");
            }
        };

        int main() {
            auto a = new class Base();
            printf("%x\n", *(int *) (void *) a);
            for (int i = 0; i < 10; i++) {
                auto b = new class Base();
                printf("%x\n", *(int *) (void *) b);
                delete b;
            }
            return 0;
        }


        // output
        2f
        2f
        2f
        2f
        2f
        2f
        2f
        2f
        2f
        2f
        2f
        ```

5. 构造函数
    - 构造函数不能是虚函数
        - 存储空间角度：因为虚函数是通过类对象的虚表的地址来调用的，因此在调用之前必须初始化类对象及其虚表，而类对象在被构造之前是没有实际地址的，也就没有虚函数表，也就查不到对应的函数地址
        - 使用角度：构造一个对象时，必须知道对象的实际类型，而虚函数行为是在运行期间确定的；而在构造一个对象时，由于对象还未构造成功，编译器无法知道对象的实际类型是基类还是派生类

6. 析构函数
    - 无论类的析构函数是否是虚函数，派生类指针在被析构时总是会先调用派生类的析构函数，再调用基类的析构函数
        - 测试

        ```c++
        class Base {
        public:
            Base() {
                cout << "Base Constructor" << endl;
            }
        
            ~Base() {
                cout << "Base Destructor" << endl;
            }
        };
        
        class Derivated : public Base {
        public:
            Derivated() {
                cout << "Derivated Constructor" << endl;
            }
        
            ~Derivated() {
                cout << "Derivated Destructor" << endl;
            }
        };
        
        int main() {
            Derivated *obj = new Derivated();
            delete obj;
            return 0;
        }
        
        // output
        Base Constructor
        Derivated Constructor
        Derivated Destructor
        Base Destructor
        ```

    - 如果类的析构函数是虚函数，那么基类指针在被析构时也会先调用派生类的析构函数，再调用基类的析构函数
        - 测试

        ```c++
        class Base {
        public:
            Base() {
                cout << "Base Constructor" << endl;
            }
        
            virtual ~Base() {
                cout << "Base Destructor" << endl;
            }
        };
        
        class Derivated : public Base {
        public:
            Derivated() {
                cout << "Derivated Constructor" << endl;
            }
        
            virtual ~Derivated() {
                cout << "Derivated Destructor" << endl;
            }
        };
        
        int main() {
            Base *obj = new Derivated();
            delete obj;
            return 0;
        }
        
        // output
        Base Constructor
        Derivated Constructor
        Derivated Destructor
        Base Destructor
        ```
    
    - 如果类的析构函数不是虚函数，那么基类指针在被析构时只会调用基类的析构函数，在应用多态特性时可能会造成内存泄漏
        - 测试

        ```c++
        class Base {
        public:
            Base() {
                cout << "Base Constructor" << endl;
            }
        
            ~Base() {
                cout << "Base Destructor" << endl;
            }
        };
        
        class Derivated : public Base {
        public:
            Derivated() {
                cout << "Derivated Constructor" << endl;
            }
        
            ~Derivated() {
                cout << "Derivated Destructor" << endl;
            }
        };
        
        int main() {
            Base *obj = new Derivated();
            delete obj;
            return 0;
        }
        
        // output
        Base Constructor
        Derivated Constructor
        Base Destructor
        ```

### 覆盖/重写 Override

- 基类中有非虚函数
- 派生类中重新实现这个函数

### 重载 Overload

- 同一个类中有多个函数名相同的函数，它们有不同的参数列表

### 其他

- 抽象类：含有纯虚函数的类
- 接口类：仅含有纯虚函数的抽象类
- 聚合类：用户可以直接访问其成员，并且具有特殊的初始化语法形式
    - 所有成员都是 public
    - 没有定义任何构造函数
    - 没有类内初始化
    - 没有基类，也没有 virtual 函数

### delete this

- 合法
- 但必须保证 this 对象是通过 new 分配的，而不是通过 new[]，placement new，栈上，全局，或其他对象成员分配的
- 必须保证调用 delete this 的成员函数是最后一个调用 this 的成员函数