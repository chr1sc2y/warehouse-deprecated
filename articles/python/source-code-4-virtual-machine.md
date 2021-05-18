# Python 源码学习：Python 虚拟机

[TOC]

通常我们认为 Python 是一种解释型语言，**Python 解释器**（*Python Interpreter*）由 **Python 编译器**（*Python Compiler*）和 **Python 虚拟机**（*Python Virutal Machine*）两部分组成。当我们通过 Python 命令执行一个 .py 文件时，Python 编译器会将 .py 文件中的 Python 代码编译为 **Python 字节码**（*[bytecode](https://www.quora.com/What-is-the-difference-between-byte-code-and-machine-code-and-what-are-its-advantages)*）；随后 Python 虚拟机会读取并逐步执行这些字节码。

## 1 编译

Python 提供了内置函数 `compile`，可以编译 Python 代码并生成一个包含字节码信息的对象，举例如下：

```python
code = '''
i = 1
print(i)
'''

bytecode = compile(code, '', 'exec')
print(bytecode)
print(type(bytecode))

exec(bytecode)
```

```shell
$ python3 main.py 
<code object <module> at 0x7fe5ca552a80, file "", line 2>
<class 'code'>
1
```

可以看到生成的 `bytecode` 对象的类型是 `class 'code'`，它在源码中对应的结构体是**代码对象 struct PyCodeObject**；代码对象是后续步骤中 Python 虚拟机的核心操作对象，它将字节码相关的参数、名称、指令序列、栈空间等信息包装成了一个结构体：

```c
// Include/cpython/code.h

/* Bytecode object */
struct PyCodeObject {
    PyObject_HEAD
    int co_argcount;            /* #arguments, except *args */
    int co_posonlyargcount;     /* #positional only arguments */
    int co_kwonlyargcount;      /* #keyword only arguments */
    int co_nlocals;             /* #local variables */
    int co_stacksize;           /* #entries needed for evaluation stack */
    int co_flags;               /* CO_..., see below */
    int co_firstlineno;         /* first source line number */
    PyObject *co_code;          /* instruction opcodes */
    PyObject *co_consts;        /* list (constants used) */
    PyObject *co_names;         /* list of strings (names used) */
    PyObject *co_varnames;      /* tuple of strings (local variable names) */
    PyObject *co_freevars;      /* tuple of strings (free variable names) */
    PyObject *co_cellvars;      /* tuple of strings (cell variable names) */
    /* The rest aren't used in either hash or comparisons, except for co_name,
       used in both. This is done to preserve the name and line number
       for tracebacks and debuggers; otherwise, constant de-duplication
       would collapse identical functions/lambdas defined on different lines.
    */
    Py_ssize_t *co_cell2arg;    /* Maps cell vars which are arguments. */
    PyObject *co_filename;      /* unicode (where it was loaded from) */
    PyObject *co_name;          /* unicode (name, for reference) */
    PyObject *co_lnotab;        /* string (encoding addr<->lineno mapping) See
                                   Objects/lnotab_notes.txt for details. */
    void *co_zombieframe;       /* for optimization only (see frameobject.c) */
    PyObject *co_weakreflist;   /* to support weakrefs to code objects */
    /* Scratch space for extra data relating to the code object.
       Type is a void* to keep the format private in codeobject.c to force
       people to go through the proper APIs. */
    void *co_extra;

    /* Per opcodes just-in-time cache
     *
     * To reduce cache size, we use indirect mapping from opcode index to
     * cache object:
     *   cache = co_opcache[co_opcache_map[next_instr - first_instr] - 1]
     */

    // co_opcache_map is indexed by (next_instr - first_instr).
    //  * 0 means there is no cache for this opcode.
    //  * n > 0 means there is cache in co_opcache[n-1].
    unsigned char *co_opcache_map;
    _PyOpcache *co_opcache;
    int co_opcache_flag;  // used to determine when create a cache.
    unsigned char co_opcache_size;  // length of co_opcache.
};
```

### 1.1 字节码

在所有的这些成员变量中，`PyObject *co_code` 存储了编译后生成的字节码，它是以字节的方式存储的：

```python
code = bytecode.co_code
print(code)
```

```shell
b'd\x00Z\x00e\x01e\x00\x83\x01\x01\x00d\x01S\x00'
```

我们可以使用 Python 内置模块 dis 来将这些字节码反编译成类似于汇编语言的指令：

```python
import dis
dis.dis(bytecode)
```

```shell
          0 LOAD_CONST               0 (0)
          2 STORE_NAME               0 (0)
          4 LOAD_NAME                1 (1)
          6 LOAD_NAME                0 (0)
          8 CALL_FUNCTION            1
         10 POP_TOP
         12 LOAD_CONST               1 (1)
         14 RETURN_VALUE
```

在反编译后的输出结果中，第一列代表字节码中每一条指令的**偏移量 offset**；第二列代表各条**助记符 mnemonics**的名称，这些助记符可以很方便地帮助我们理解在后续的步骤中 Python 虚拟机要执行的事件；第三列则是每条指令的**操作数 opargs**。

同时，在字节码对应的十六进制表示中，每一位数字也分别代表了不同的助记符和操作数，我们可以直接通过打印出字节码的十六进制以查看其内容：

```python
print(bytecode.hex())
```

```shell
64005a00650165008301010064015300
```

以第一行为例，在 offset == 0 的地方可以找到数字 64，即 LOAD_CONST 加载常量助记符对应的**操作码 opcode**，其后紧跟着的是它的操作数 opargs == 0；第二行对应的 offset == 2，即保存变量助记符对应的操作码 opcode == 5a 及其操作数 opargs == 0；以此类推。

Python 的 opcode 模块提供了关于 Python 虚拟机中助记符和操作码的相关信息，也可以在源码的 Include/opcode.h 中找到相关定义：

```python
import opcode
print(opcode.opname[0x64])
print(opcode.opname[0x5a])
print(opcode.opmap['LOAD_NAME'])
print(opcode.opmap['RETURN_VALUE'])
```

```shell
LOAD_CONST
STORE_NAME
101
83
```

<!-- 回到刚才的例子中，我们可以将反编译生成的指令分为两部分，分别对应用于反编译变量 `code` 中的两行代码；第一部分 `i = 1`

```shell
          0 LOAD_CONST               0 (0)
          2 STORE_NAME               0 (0)
          4 LOAD_NAME                1 (1)
          6 LOAD_NAME                0 (0)
          8 CALL_FUNCTION            1
         10 POP_TOP
         12 LOAD_CONST               1 (1)
         14 RETURN_VALUE
``` -->

## 2 运行

类似于 x86-64, arm 平台和 Java 虚拟机，Python 虚拟机也是 **基于栈的**（*Stack-Based*），它的函数调用都是通过**栈** *stack* 和**栈帧** *stackframe* 来实现的。

### 2.1 栈

栈是 CPU 寄存器中的一块内存区域，它是一种 FILO 的数据结构，可以进行插入或删除操作的一边称为栈顶，另一边则称为栈底；以常用的 x86-64 架构为例，它拥有 16 个**通用寄存器**（*general-purpose registers*），寄存器被集成在 CPU 芯片上，其中 rsp 寄存器保存栈顶（运行时栈的结束位置）的地址，rbp 寄存器保存当前栈帧底部（运行时栈的起始位置）的位置，其他通用寄存器的功能如下。

![x86-64-registers](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/python/x86-64-registers.png)

栈相关的最常见操作有 push 和 pop，对于栈地址空间向下增长的 x86-64 架构来说，push 操作会取出 rsp 寄存器的值，对其进行减 8 的操作，再将操作数 opargs 写入这个地址中；而 pop 则正好相反，它先从 rsp 寄存器指向的地址取出数据，再对其地址加 8。

以一个简单的函数调用为例：

```cpp
// main.cpp
#include <iostream>
#include <stdint.h>

using namespace std;

int Calc(char a, uint16_t b, int64_t c)
{
    int d = a + b;
    int e = d + c;
    return e + 1;
}

int main()
{
    int sum = Calc(1, 2, 3);
    cout << sum << endl;
}
```

用 gdb 打开并在 main 处断点：

```shell
$ g++ -g -o main main.cpp
$ gdb main
(gdb) b main
(gdb) r
(gdb) layout asm

> 0x4007fa <main()+23>          callq      0x4007ad <Calc(char, unsigned short, long)>
```

逐步执行汇编指令，在 callq 指令处进入到 Calc 函数中：

```shell
> 0x4007ad <Calc(char, unsigned short, long)>          push     %rbp

(gdb) stepi

> 0x4007ad <Calc(char, unsigned short, long)>            push     %rbp
  0x4007ae <Calc(char, unsigned short, long)+1>          mov      %rsp,%rbp
```

此时 rsb 和 rbp 指针应该还分别在 main 函数栈帧的顶部和底部，并且能够发现栈地址空间的确是向下增长的：

```shell
(gdb) info reg

rbp            0x7fffffffe110   0x7fffffffe110
rsp            0x7fffffffe0f8   0x7fffffffe0f8
```

执行接下来的 `push %rbp` 汇编指令，查看 rbp 指针的位置：

```shell
(gdb) nexti
(gdb) info reg

rbp            0x7fffffffe0f0   0x7fffffffe0f0
rsp            0x7fffffffe0f8   0x7fffffffe0f8
```

可以看到 rbp == rsp - 8，符合 push 操作的定义；再不断向后执行，直到即将执行 pop 指令：

```shell
> 0x4007e1 <Calc(char, unsigned short, long)+52>            pop      %rbp

(gdb) info reg
```



用 `info frame` 可以看到当前栈帧的相关信息，包括：

- `Stack level 0` backtrace 的栈帧编号 0
- 当前的栈帧地址 `frame at 0x7fffffffdbe0`
- rip 是下一条将要执行的指令地址，即第 8 行的指令 `int d = a + b;` 的地址 `0x4007c0`
- saved rip 是调用方的函数地址，也是下一调指令的返回地址
- 调用方的栈帧地址 `called by frame at 0x7fffffffdc00`
- 当前使用的高级语言 `source language c++.`
- 参数的起始地址 `Arglist at 0x7fffffffdbd0`，以及参数值 `args: a=1 '\001', b=2, c=3`
- 局部变量的地址 `Locals at 0x7fffffffdbd0`
- 调用当前函数的调用方的地址 `Previous frame's sp is 0x7fffffffdbe0`，也是当前函数的起始地址
查看当前栈的起始位置 $rbp，结束位置 $rsp，参数 $rdi $rsi $rdx， 局部变量

```shell
(gdb) info frame
Stack level 0, frame at 0x7fffffffdbe0:
 rip = 0x4007c0 in Calc (main.cpp:8); saved rip 0x4007ff
 called by frame at 0x7fffffffdc00
 source language c++.
 Arglist at 0x7fffffffdbd0, args: a=1 '\001', b=2, c=3
 Locals at 0x7fffffffdbd0, Previous frame's sp is 0x7fffffffdbe0
 Saved registers:
  rbp at 0x7fffffffdbd0, rip at 0x7fffffffdbd8
```

###

代码对象本身只包含了字节码相关的信息，并不具有用于执行字节码所需要的上下文信息，因此需要引入**栈帧对象 struct PyFrameObject**，














































