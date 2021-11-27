## 协程和线程的对比

协程

- 本质是用户态的线程
- 由程序实现调度，由于多个协程不涉及用户态和内核态的切换，因此运行开销远小于线程
- 执行过程中，可以调度别的协程，自己中途退出
- 缺点是无法利用多核资源，协程是在单个线程上进行调度的，需要配合多进程

线程

- 上下文存储在内核上，切换时需要进入内核态，会造成调度开销

## 有栈协程

协程切换时需要保存上下文环境，包括寄存器值（例如 ebp 和 esp 寄存器指向的被调函数的头尾地址），和栈帧里的内容（参数，局部变量）

有栈协程分为独立栈和共享栈两种

- 独立栈是指协程运行的过程中，它们用的栈帧是自己的，这块栈地址的内容不会让其他协程进行读写；优点是切换时不需要对栈内容进行拷贝，缺点是更浪费内存，因为需要预分配栈空间，且不一定会被完全使用
- 共享栈指所有协程运行时使用的是同一个栈；优点是省内存；缺点是切换协程时需要将协程的内容拷贝进/出这个共享栈



## libco

### 协程的数据结构

```cpp
struct stCoRoutine_t
{
    stCoRoutineEnv_t *env; // 协程执行的环境，libco协程一旦创建便跟对应线程绑定了，不支持在不同线程间迁移，这里env即同属于一个线程所有协程的执行环境，包括了当前运行协程、嵌套调用的协程栈，和一个epoll的封装结构。这个结构是跟运行的线程绑定了的，运行在同一个线程上的各协程是共享该结构的，是个全局性的资源。
    pfn_co_routine_t pfn; // 实际等待执行的协程函数
    void *arg; // 上面协程函数的参数
    coctx_t ctx; // 上下文，即ESP、EBP、EIP和其他通用寄存器的值

    // 一些状态和标志变量
    char cStart;
    char cEnd;
    char cIsMain;
    char cEnableSysHook;
    char cIsShareStack;

    void *pvEnv; // 保存程序系统环境变量的指针

    //char sRunStack[ 1024 * 128 ];
    stStackMem_t* stack_mem; // 协程运行时的栈内存，这个栈内存是固定的 128KB 的大小。


    //save stack buffer while confilct on same stack_buffer;
    // 共享栈模式中使用
    char* stack_sp; 
    unsigned int save_size;
    char* save_buffer;

    stCoSpec_t aSpec[1024];
};
```



### 协程环境

libco 中用于管理协程的结构体为 `stCoRoutineEnv_t`，它在第一个协程创建的时候初始化，每个线程中都只有一个 `stCoRoutineEnv_t` 实例。一个线程对应一个 `stCoRoutineEnv_t` 结构，一个 `stCoRoutineEnv_t` 结构对应多个协程。

线程可以通过该 `stCoRoutineEnv_t` 实例了解现在有哪些协程，哪个协程正在运行，以及下一个运行的协程是哪个。

```CPP
struct stCoRoutineEnv_t
{
	stCoRoutine_t *pCallStack[ 128 ];
	int iCallStackSize;
	stCoEpoll_t *pEpoll;

	//for copy stack log lastco and nextco
	stCoRoutine_t* pending_co;
	stCoRoutine_t* occupy_co;
};
```

### 协程切换

协程上下文结构：

```cpp
// 栈切换的时候需要保存的寄存器空间
struct coctx_t
{
#if defined(__i386__)
    void *regs[ 8 ];
#else
    void *regs[ 14 ];
#endif
    size_t ss_size;
    char *ss_sp;    
};
```



协程上下文 `coctx_t` 的初始化如下：

```
int coctx_make( coctx_t *ctx,coctx_pfn_t pfn,const void *s,const void *s1 )
{
    //make room for coctx_param
    // 获取(栈顶 - param size)的指针，栈顶和sp指针之间用于保存函数参数
    char *sp = ctx->ss_sp + ctx->ss_size - sizeof(coctx_param_t);
    sp = (char*)((unsigned long)sp & -16L); // 用于16位对齐
 
    // 将参数填入到param中
    coctx_param_t* param = (coctx_param_t*)sp ;
    param->s1 = s;
    param->s2 = s1;

    memset(ctx->regs, 0, sizeof(ctx->regs));
    // 为什么要 - sizeof(void*)呢？ 用于保存返回地址
    ctx->regs[ kESP ] = (char*)(sp) - sizeof(void*);
    ctx->regs[ kEIP ] = (char*)pfn;

    return 0;
}
```

流程如下：

1. 给 `coctx_pfn_t` 函数预留2个参数的大小，并4位地址对齐
2. 将参数填入到预存的参数中
3. `regs[kEIP]`中保存了`pfn`的地址，`regs[kESP]`中则保存了栈顶指针 - 4个字节的大小的地址。这预留的4个字节用于保存`return address`。

### 协程管理

协程的管理层可以自由选择管理方式，除了直接使用共享栈和独立栈的方式以外，甚至可以通过判断栈空间的大小来比较灵活地自动切换两种模式。

































