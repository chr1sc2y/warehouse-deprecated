Linux:

我平时的学习基本上都在Linux下完成的，我现在有两台电脑，一台iMac，一台笔记本，笔记本上就装的Linux，然后每天都会用很多指令，
比如说用pipeline管道输出后用grep做正则搜索，用ifconfig和netstat查看网络状态，用top看进程资源，用iostat看cpu和硬盘状态
还有就是最基础的cd ls cp rm mkdir find whereis，还有远程的scp
还有一个查看程序运行时的系统调用，在Linux下是strace，Mac下是dtruss，有些命令在两个平台不一样
Linux下对文件操作有两种方式：系统调用（底层调用，面向硬件），和库函数调用（应用）。系统调用：open, close, read, write, ioctl等，用于底层文件访问

调试：
用vim编辑（会一些vim的快捷键，比如dd删除一行，gg定位到头部，6$定位到某一行），用gdb调试，打log出来看，用过有个叫log4cplus的框架，有内存错误或者内存泄漏的问题就用valgrind调试。还有用GNU profiler来查看一些函数的调用次数，调用点相关的信息（编译器的话在Linux上GCC，在MAC上用自带的clang，单元测试用gtest。）

C++ 内存分配
1. 栈区：参数，局部变量 2.堆区：new，向高地址扩展，不连续 3.自由存储区，malloc 4.全局/静态存储区，存储全局变量和静态变量 5.常量存储区，存储常量
new 分配失败： 1. int* p = new (std::nothrow) int(1); 返回空指针  2. 用 try {} catch (const bad_alloc &b) {} 捕捉异常

auto_ptr：有拷贝构造和赋值操作，所有权转移
unique_ptr：独占式拥有，保证同一时间内只有一个智能指针可以指向特定指针，同时销毁，可以移交拥有权
shared_ptr：多个智能指针指向相同指针对象，在最后一个shared_ptr被销毁时对象被释放；搭配 weak_ptr、bad_weak_ptr等辅助类来实现，可以定制型删除器，因为默认只会把指针删掉，如果是个数组不会删后面的
循环引用就是说有两个类对象的智能指针，他们分别有一个成员变量是对方的类型，并且指向了对方的智能指针，形成了一个环状，那么他们就无法被正确的释放掉
weak_ptr：只引用，不计数，如果一个指针被一些shared_ptr和weak_ptr同时引用，当这些shared_ptr都释放掉，不管还有没有weak_ptr引用该内存，内存都会被释放，解决了循环引用

线程通信：锁（互斥锁，读写所，自旋锁，条件变量，原子变量，临界区），信号量，信号，屏障

