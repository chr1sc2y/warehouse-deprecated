# C++ 并发入门：以 LeetCode 1114 为例

[toc]

## 题目

直接做题：[1114 按序打印](https://leetcode-cn.com/problems/print-in-order/)

## 解法

### 1. std::mutex

如果你对 c++ 11 略为熟悉的话，应该能够想到用 [`std::mutex`](https://en.cppreference.com/w/cpp/thread/mutex) 来解这道题，在函数构造时（主线程）对 `std::mutex` 进行 `lock`，然后在各个线程调用的函数中依次对 `std::mutex` 对象进行 `unlock`：

```cpp
class Foo {
    mutex mtx1, mtx2;
public:
    Foo() {
        mtx1.lock(), mtx2.lock();
    }

    void first(function<void()> printFirst) {
        printFirst();
        mtx1.unlock();
    }

    void second(function<void()> printSecond) {
        mtx1.lock();
        printSecond();
        mtx1.unlock();
        mtx2.unlock();
    }

    void third(function<void()> printThird) {
        mtx2.lock();
        printThird();
        mtx2.unlock();
    }
};
```

Mutex 即 **mutual exclusion**，是用来防止多个线程同时访问共享资源对象的机制，在同一时间只有一个线程可以拥有一个 `mutex` 对象，其他线程调用 `std::mutex::lock` 函数时会阻塞直到其获取锁资源。

这段代码能够 ac，但实际上这种使用 `mutex` 的方法是**错误**的，因为根据 c++ 标准，在一个线程尝试对一个 `mutex` 对象进行 `unlock` 操作时，`mutex` 对象的所有权必须在这个线程上；也就是说，应该**由同一个线程来对一个 `mutex` 对象进行 `lock` 和 `unlock` 操作**，否则会产生未定义行为。题目中提到了 `first`, `second`, `third` 三个函数分别是由三个不同的线程来调用的，但我们是在 `Foo` 对象构造时（可以是在 create 这几个线程的主线程中，也可以是在三个线程中的任意一个）对两个 `mutex` 对象进行 `lock` 操作的，因此，调用 `first` 和 `second` 函数的两个线程中至少有一个在尝试获取其他线程所拥有的 `mutex` 对象的所有权。

另外，如果非要讨论这个解法有什么优化的余地的话，因为 `mutex` 对象本身是不保护任何数据的，我们只是通过 `mutex` 的机制来保护数据被同时访问，所以最好**使用 `lock_guard` 或者 `unique_lock` 提供的 [RAII](https://en.cppreference.com/w/cpp/language/raii) 机制来管理 `mutex` 对象**，而不是直接操作 `mutex` 对象；其中 [`lock_guard`](https://en.cppreference.com/w/cpp/thread/lock_guard) 只拥有构造和析构函数，用来实现 RAII 机制，而 [`unique_lock`](https://en.cppreference.com/w/cpp/thread/unique_lock) 是一个完整的 `mutex` 所有权包装器，封装了所有 `mutex` 的函数：

```cpp
class Foo {
    mutex mtx_1, mtx_2;
    unique_lock<mutex> lock_1, lock_2;
public:
    Foo() : lock_1(mtx_1, try_to_lock), lock_2(mtx_2, try_to_lock) {
    }

    void first(function<void()> printFirst) {
        printFirst();
        lock_1.unlock();
    }

    void second(function<void()> printSecond) {
        lock_guard<mutex> guard(mtx_1);
        printSecond();
        lock_2.unlock();
    }

    void third(function<void()> printThird) {
        lock_guard<mutex> guard(mtx_2);
        printThird();
    }
};
```

### 2. std::condition_variable

[`std::condition_variable`](https://en.cppreference.com/w/cpp/thread/condition_variable) 是一种用来同时阻塞多个线程的**同步原语**（synchronization primitive），**`std::condition_variable` 必须和 `std::unique_lock` 搭配使用**：

```cpp
class Foo {
    condition_variable cv;
    mutex mtx;
    int k = 0;
public:
    void first(function<void()> printFirst) {
        printFirst();
        k = 1;
        cv.notify_all();														// 通知其他所有在等待唤醒队列中的线程
    }

    void second(function<void()> printSecond) {
        unique_lock<mutex> lock(mtx);								// lock mtx
        cv.wait(lock, [this](){ return k == 1; });	// unlock mtx，并阻塞等待唤醒通知，需要满足 k == 1 才能继续运行
        printSecond();
        k = 2;
        cv.notify_one();														// 随机通知一个（unspecified）在等待唤醒队列中的线程
    }

    void third(function<void()> printThird) {
        unique_lock<mutex> lock(mtx);								// lock mtx
        cv.wait(lock, [this](){ return k == 2; });	// unlock mtx，并阻塞等待唤醒通知，需要满足 k == 2 才能继续运行
        printThird();
    }
};

```

`std::condition_variable::wait` 函数会执行三个操作：先将当前线程加入到等待唤醒队列，然后 `unlock` `mutex` 对象，最后阻塞当前线程；它有两种重载形式，第一种只接收一个 `std::mutex` 对象，此时线程一旦接受到唤醒信号（通过 `std::condition_variable::notify_one` 或 `std::condition_variable::notify_all` 进行唤醒），则无条件立即被唤醒，并重新 `lock` `mutex`；第二种重载形式还会接收一个条件（一般是 variable 或者 `std::function`），即只有当满足这个条件时，当前线程才能被唤醒，它在 gcc 中的实现也很简单，只是在第一种重载形式之外加了一个 `while` 循环来保证只有在满足给定条件后才被唤醒，否则重新调用 `wait` 函数：

```cpp
template<typename _Predicate>
void wait(unique_lock<mutex>& __lock, _Predicate __p)
{
    while (!__p())
        wait(__lock);
}
```

**条件变量** `std::condition_variable` 的机制和**信号量** semaphore 比较类似，它们都是建立在 mutex 的基础之上，用于实现对共享资源的同步访问，然而可惜的是 c++ 标准库中并没有信号量的实现和封装，但我们仍然可以使用 c 语言提供的 `<sempahore.h>` 库来解题 ：

```cpp
#include <semaphore.h>

class Foo {
private:
    sem_t sem_1, sem_2;

public:
    Foo() {
        sem_init(&sem_1, 0, 0), sem_init(&sem_2, 0, 0);
    }

    void first(function<void()> printFirst) {
        printFirst();
        sem_post(&sem_1);
    }

    void second(function<void()> printSecond) {
        sem_wait(&sem_1);
        printSecond();
        sem_post(&sem_2);
    }

    void third(function<void()> printThird) {
        sem_wait(&sem_2);
        printThird();
    }
};

```

### 3. std::future

**`std::future` 是用来获取异步操作结果的模板类**；[`std::packaged_task`](https://en.cppreference.com/w/cpp/thread/packaged_task), [`std::promise`](https://en.cppreference.com/w/cpp/thread/promise),  [`std::async`](https://en.cppreference.com/w/cpp/thread/async) 都可以进行异步操作，并拥有一个 `std::future` 对象，用来存储它们所进行的异步操作返回或设置的值（或异常），这个值会在将来的某一个时间点，通过某种机制被修改后，保存在其对应的 `std::future` 对象中：

对于 `std::promise`，可以通过调用 `std::promise::set_value` 来设置值并通知 `std::future` 对象：

```c++
class Foo {
    promise<void> pro1, pro2;

public:
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

`std::future<T>::wait` 和 `std::future<T>::get` 都会阻塞地等待拥有它的 `promise` 对象返回其所存储的值，后者还会获取 `T` 类型的对象；这道题只需要利用到异步通信的机制，所以并没有返回任何实际的值。

`std::packaged_task` 是一个拥有 `std::future` 对象的 functor，将一系列操作进行了封装，在运行结束之后会将返回值保存在其所拥有的 `std::future<T>` 对象中；同样地，在这道题中只需要利用到其函数运行结束之后通知 `std::future` 对象的机制：

```cpp
class Foo {
    function<void()> task = []() {};
    packaged_task<void()> pt_1{ task }, pt_2{ task };

public:
    void first(function<void()> printFirst) {
        printFirst();
        pt_1();
    }

    void second(function<void()> printSecond) {
        pt_1.get_future().wait();
        printSecond();
        pt_2();
    }

    void third(function<void()> printThird) {
        pt_2.get_future().wait();
        printThird();
    }
};
```

### 4. std::atomic

我们平时进行的数据修改都是非原子操作，如果多个线程同时以非原子操作的方式修改同一个对象可能会发生数据争用，从而导致未定义行为；而**原子操作能够保证多个线程顺序访问，不会导致数据争用**，其执行时没有任何其它线程能够修改相同的原子对象。c++ 11 提供了 `std::atomic<T>` 模板类来构造原子对象：

```c++
class Foo {
    std::atomic<bool> a{ false };
    std::atomic<bool> b{ false };
public:
    void first(function<void()> printFirst) {
        printFirst();
        a = true;
    }

    void second(function<void()> printSecond) {
        while (!a)
            this_thread::sleep_for(chrono::milliseconds(1));
        printSecond();
        b = true;
    }

    void third(function<void()> printThird) {
        while (!b)
            this_thread::sleep_for(chrono::milliseconds(1));
        printThird();
    }
};
```

值得注意的是，原子操作的实现跟处理器和操作系统内核相关，因此 c++ 标准并没有规定 `atomic` 的实现是否是无锁的（lock-free），只规定了需要提供一个 `is_lock_free()` 来查询当前编译器对 `atomic` 的实现是否是无锁的。