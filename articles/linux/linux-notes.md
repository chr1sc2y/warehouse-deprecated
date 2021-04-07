# Linux 笔记

## 概念

程序在用户态下执行，当程序需要使用操作系统提供的服务（打开设备、读写文件）时，向操作系统发出系统调用。

进程发出系统调用之后，它所处的运行状态会由用户态变成核心态。

当进程发出系统调用申请的时候，会产生一个软件中断，系统处理这个中断时就处于核心态。

用户态和核心态之间的区别：

1. 用户态的进程只能存取它自己的指令和数据；而核心态下的进程能够存取内核和用户（其他进程）地址
2. 某些机器指令是特权指令，在用户态下执行特权指令会引起错误

对此要理解的一个是，在系统中内核并不是作为一个与用户进程平行的估计的进程的集合，内核是为用户进程运行的。



## 指令

### 压缩

- 压缩 tar cvf file.tar.gz file1 file2 ...
- 解压 tar xvf file.tar.gz

### 进程

top: 查看进程

top -H -p x：查看x进程的所有线程

gstack y：查看y线程的当前堆栈



### cpu

#### lscpu

lscpu: 显示 cpu 架构相关的信息

![lscpu](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resource/linux/lscpu.png)

CPU(s): 逻辑 cpu 数量，等于

Thread(s) per core: 每个 cpu 上能运行的线程数量（是否有 hyper-threading）

Core(s) per socket: 每个 socket 上拥有的物理 cpu 数量

Socket(s): 占据主板上一个 socket 的物理 cpu 包

#### cache

- cache 是在快设备访问慢设备时预留的高速存储区，目的是在掩盖访问延时的同时，提高数据传输率，弥补两者的速度差距
- cpu 中一般有多个级别的缓存，其中 L1 cache 和 L2 cache 是每个 cpu 各有一个，L3 cache 是多个核心共用一个，L1 cache 分为 L1i 和 L1d，分别用来存储指令和数据，L2 cache 和 L3 cache 不区分指令和数据，TLB cache 主要用来缓存 MMU（内存管理单元） 使用的页表



### 内存

#### top

top: 查看进程信息

![top](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resource/linux/top.png)

第一行：当前系统时间，系统已运行时间，在线用户数量，1、5、15分钟内的系统负载平均值

第二行：总进程数，运行进程数，睡眠线程数，停止进程数，僵尸进程数

第三行：cpu 占用率

- us: user space 用户态进程占用率
- sy: kernel space 系统态进程占用率
- ni: nice 用户态进程中修改过优先级的进程占用率
- id: idle 空闲率
- wa:  wait 等待 io 的 cpu 时间比
- hi: hardware interrupts 硬中断的 cpu 时间比
- si: software interrupst 软中断的 cpu 时间比
- st: steal time 运行虚拟器失去的时间比

第四行：内存总量 bytes，空闲量，使用量，缓存量

第五行：交换区总量，空闲量，使用量

第六行：内存列表

- PID: process id 进程 ID
- USER: 进程拥有者
- PR: priority 进程优先级，值越低，优先级越高，rt real-time 实时优先级
- NI: nice 值，值越低，优先级越高
- VIRT: virtual memory 虚拟内存使用量
- RES: resident memory 未被换出的物理内存量
- SHR: share memory 共享内存量
- S: 进程状态，D 不可中断的睡眠，R 运行，S 睡眠，T 追踪停止，Z 僵尸
- %CPU: 占用 cpu 百分比
- %MEM: 占用内存百分比
- TIME+: 运行时间
- COMMAND: 名称

##### 快捷键

P: 按照 cpu 占用率排序

M: 按照内存占用率排序

T: 按照 cpu 占用时间排序

q: 退出

s: 修改刷新时间

k: kill 进程，默认使用信号 15

##### 线程

top -H -p pid 查看进程内的所有线程的信息

ps -T -p pid 查看进程内所有线程

#### gstack

gstack: 打印一个进程的所有线程，或单个线程的调用栈





## 其他

- `echo $?` will return the exit status of last command