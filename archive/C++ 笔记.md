## 宏

#：将其后的宏参数进行字符串化操作（Stringfication），即在其左右各加上一个双引号

##：连接符（concatenator），将两个Token连接为一个Token



## 编译

.h 文件中变量要用extern 修饰，否则在被同一个 .cpp 文件多次引用的时候会 multiple definition 报错。

.a 文件：静态库 static libraries

.so 文件：动态库 dynamic libraries

先编写 foo.h 和 foo.cc 文件

### 静态库编译步骤

2. 使用 g++ -std=c++11 -c foo.cc 生成 .o 机器码 obj 文件
3. 使用 ar -crv libfoo.a foo.o 生成 .a 库文件
4. 编写 bar.cc 文件，使用相对目录 include foo.h 文件，注意要把 foo.h 头文件和 libfoo.a 库文件放在相同目录
5. 使用 g++ -std=c++11 -o foo foo.cc -L./lib -ladd 生成 foo 可执行文件，其中 -L 指定静态库目录，-l 指定头文件，注意 -L 和 -l 参数一定要在被编译文件之后

### 动态库编译步骤

1. 使用 g++ foo.cc -c -fPIC 生成位置无关代码 .o 机器码 obj文件
2. 使用 g++ foo.o -shared -o libfoo.so 生成 .so 动态库文件
3. 或者去掉中间生成 obj 文件的步骤，直接使用 g++ foo.cc -shared -fPIC -o libfoo.so 生成 .so 动态库文件
4. 省略掉了 -g -Wall -std=c++11 参数

## 链接

`gcc -s`：去掉符号表

strip xxx.bin：删除二进制文件的符号表

## 工具

GNU Binary Utilities：编程语言工具程序，用来处理目标文件，通常搭配 GCC，Make 等使用。

其包括：addr2line、ar、gprof、nm、objcopy、objdump、ranlib、size、strings、strip。其中 ar 用于建立、修改、提取档案文件 archive，archive 是一个包含多个被包含文件的单一文件（即库文件），其结构保证了可以从中检索并得到原始的被包含文件（称之为 archive 中的 member），member的原始文件内容、权限、时间戳、所有者和组等属性都被保存在 archive 中。

## 指针

std::shared_ptr<int> sum = std::make_shared<int>(0); // C++ 11

std::unique_ptr<Obj> sum(new Obj(1)); // C++ 11

std::unique_ptr<obj> sum = std::make_unique<int>(1)); C++ 14



## 内存

memcpy：直接从头开始拷贝，不做校验

memmove：检查是否有重合，有的话反转拷贝顺序

#pragma pack(n)：将struct按指定n字节数对齐



## Namespace

避免 name collisions



## 异常

### std::exception::what

Returns the explanatory string.

virtual const char* what() const noexcept; (since C++11) 



## 仿函数

仿函数（Functor）又称为函数对象（Function Object），是一个能行使函数功能的类，其必须重载 operator() 运算符

```c++
// functor
class Functor
{
private:
    int num_;
public:
    void operator() (int i)
    {
        num_ += i;
    }
}

// function
void Func(int i)
{
    cout << i << endl;
}
```



## lambda

在lambda中递归调用自身时不能使用auto自动推导，需要显示指明



## Lock

### std::Mutex



mutex::lock：阻塞等待锁

mutex::try_lock：如果锁不可用则直接返回false



lock_guard<_Mutex> 只有构造（直接lock和adopt_lock_t）和析构函数，传入mutex 对象进行构造，退出作用域时执行析构，解锁mutex；会在函数抛出异常时，自动调用作用域内的变量的析构函数



// 用于偏特化

 struct defer_lock_t { };	不获取锁的所有权

 struct try_to_lock_t { };	尝试获取锁的所有权

 struct adopt_lock_t { };	已经获取了所有权



unique_lock<_Mutex> 有两个成员变量：   mutex_type* _M_device 和 bool  _M_owns; 构造函数一定获取锁_M_device，_M_owns标记锁状态

unique_lock<_Mutex>::lock：lock mutex，如果没有锁对象（用constructor with 0para 构造）或已经上锁则throw



### std::shared_mutex

读写锁



## std::thread

- 问题：runtime error: `terminate called without an active exception Aborted`
  - When a thread object goes out of scope and it is in joinable state, the program is terminated. 
  - 使用 join（调用线程阻塞）或 detach（非阻塞，将被调用线程驻留到后台，放弃控制权）



## std::ref

使用 thread 和 bind 的时候需要显示使用 std::ref 来传递引用



## std::future

- 访问异步操作的结果的机制



## std::vector

```cpp
vector<int>().swap(res);	// ok
res.swap(vector<int>());	// error
```

参数必须传左值引用

```cpp
void swap(vector& __x)
#if __cplusplus >= 201103L
			noexcept(_Alloc_traits::_S_nothrow_swap())
#endif
      {
	this->_M_impl._M_swap_data(__x._M_impl);
	_Alloc_traits::_S_on_swap(_M_get_Tp_allocator(),
	                          __x._M_get_Tp_allocator());
      }

```

# struct

offsetof