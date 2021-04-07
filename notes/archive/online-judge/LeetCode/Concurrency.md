---
title: "LeetCode 并发"
date: 2019-07-22T10:10:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 并发](https://leetcode-cn.com/problemset/concurrency/)

## 题目

#### [1114 按序打印](https://leetcode-cn.com/problems/print-in-order/)

##### C++ mutex

```c++
class Foo {
    mutex lock1, lock2;
public:
    Foo() {
        lock1.lock();
        lock2.lock();
    }

    void first(function<void()> printFirst) {
        printFirst();
        lock1.unlock();
    }

    void second(function<void()> printSecond) {
        lock1.lock();
        printSecond();
        lock1.unlock();
        lock2.unlock();
    }

    void third(function<void()> printThird) {
        lock2.lock();
        printThird();
        lock2.unlock();
    }
};
```

##### C++ condition_variable

```c++
class Foo {
    int i;
    mutex mut;
    condition_variable con_var1, con_var2;

public:
    Foo() : i(1) {
    }

    void first(function<void()> printFirst) {
        unique_lock<mutex> lock(mut);
        printFirst();
        ++i;
        con_var1.notify_one();
    }

    void second(function<void()> printSecond) {
        unique_lock<mutex> lock(mut);
        con_var1.wait(lock, [this]() { return i == 2; });
        printSecond();
        ++i;
        con_var2.notify_one();
    }

    void third(function<void()> printThird) {
        unique_lock<mutex> lock(mut);
        con_var2.wait(lock, [this]() { return i == 3; });
        printThird();
    }
};
```

##### C++ atomic

```c++
class Foo {
    atomic_int i;
public:
    Foo() : i(1) {
    }

    void first(function<void()> printFirst) {
        printFirst();
        ++i;
    }

    void second(function<void()> printSecond) {
        while (i != 2) {}
        printSecond();
        ++i;
    }

    void third(function<void()> printThird) {
        while (i != 3) {}
        printThird();
        i = 1;
    }
};
```

##### C++ promise

```c++
class Foo {
    promise<void> pro1, pro2;

public:
    Foo() {
    }

    void first(function<void()> printFirst) {
        printFirst();
        pro1.set_value();
    }

    void second(function<void()> printSecond) {
        pro1.get_future().wait();
        printSecond();
        pro2.set_value();
    }

    void third(function<void()> printThird) {
        pro2.get_future().wait();
        printThird();
    }
};
```

#### [1115 交替打印FooBar](https://leetcode-cn.com/problems/print-foobar-alternately/)

##### C++ mutex

```c++
class FooBar {
private:
    int n;
    mutex mut1, mut2;
public:
    FooBar(int n) {
        this->n = n;
        mut2.lock();
    }

    void foo(function<void()> printFoo) {
        for (int i = 0; i < n; i++) {
            mut1.lock();
            printFoo();
            mut2.unlock();
        }
    }

    void bar(function<void()> printBar) {
        for (int i = 0; i < n; i++) {
            mut2.lock();
            printBar();
            mut1.unlock();
        }
    }
};
```

##### C++ condition_variable

```c++
class FooBar {
private:
    int m, n;
    mutex mut;
    condition_variable con_var1, con_var2;

public:
    FooBar(int n) : n(n), m(0) {
    }

    void foo(function<void()> printFoo) {
        for (int i = 0; i < n; i++) {
            unique_lock<mutex> lock(mut);
            con_var1.wait(lock, [&]() { return m == 0; });
            printFoo();
            ++m;
            con_var2.notify_one();
        }
    }

    void bar(function<void()> printBar) {
        for (int i = 0; i < n; i++) {
            unique_lock<mutex> lock(mut);
            con_var2.wait(lock, [&]() { return m == 1; });
            printBar();
            m = 0;
            con_var1.notify_one();
        }
    }
};
```

##### C++ atomic

```c++
class FooBar {
private:
    int n;
    atomic_int m;

public:
    FooBar(int n) : n(n), m(0) {
    }

    void foo(function<void()> printFoo) {
        for (int i = 0; i < n; i++) {
            while (m != 0) {}
            printFoo();
            m = 1;
        }
    }

    void bar(function<void()> printBar) {
        for (int i = 0; i < n; i++) {
            while (m != 1) {}
            printBar();
            m = 0;
        }
    }
};
```

##### C++ promise

```c++
class FooBar {
private:
    int n;
    vector<promise<void>> pros1, pros2;

public:
    FooBar(int n) : n(n) {
        for (int i = 0; i < n; ++i) {
            pros1.push_back(promise<void>());
            pros2.push_back(promise<void>());
        }
    }

    void foo(function<void()> printFoo) {
        for (int i = 0; i < n; i++) {
            if (i != 0)
                pros1[i - 1].get_future().wait();
            printFoo();
            pros2[i].set_value();
        }
    }

    void bar(function<void()> printBar) {
        for (int i = 0; i < n; i++) {
            pros2[i].get_future().wait();
            printBar();
            pros1[i].set_value();
        }
    }
};
```

#### [1116 打印零与奇偶数](https://leetcode-cn.com/problems/print-zero-even-odd/)

##### C++ mutex

```c++
class ZeroEvenOdd {
private:
    int m, n;
    mutex mut_zero, mut_odd, mut_even;

public:
    ZeroEvenOdd(int n) {
        this->m = 1;
        this->n = n;
        mut_odd.lock();
        mut_even.lock();
    }

    void zero(function<void(int)> printNumber) {
        for (int i = 0; i < n; ++i) {
            mut_zero.lock();
            printNumber(0);
            if (this->m % 2 == 1)
                mut_odd.unlock();
            else
                mut_even.unlock();
        }
    }

    void even(function<void(int)> printNumber) {
        for (int i = 0; i < n / 2; ++i) {
            mut_even.lock();
            printNumber(this->m);
            ++this->m;
            mut_zero.unlock();
        }
    }

    void odd(function<void(int)> printNumber) {
        for (int i = 0; i < (n + 1) / 2; ++i) {
            mut_odd.lock();
            printNumber(this->m);
            ++this->m;
            mut_zero.unlock();
        }
    }
};
```

#### [1117 H2O 生成](https://leetcode-cn.com/problems/building-h2o/submissions/)

##### C++ condition variable

```c++
class H2O {
    int m = 1;
    mutex mut;
    condition_variable con_var;

public:
    void hydrogen(function<void()> releaseHydrogen) {
        unique_lock<mutex> lock(mut);
        con_var.wait(lock, [&]() { return m % 3 != 0; });
        ++m;
        releaseHydrogen();
        con_var.notify_all();
    }

    void oxygen(function<void()> releaseOxygen) {
        unique_lock<mutex> lock(mut);
        con_var.wait(lock, [&]() { return m % 3 == 0; });
        ++m;
        releaseOxygen();
        con_var.notify_all();
    }
};
```