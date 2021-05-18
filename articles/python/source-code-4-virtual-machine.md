# Python 源码学习：Python 虚拟机

[TOC]

通常我们认为 Python 是一种解释型语言，**Python 解释器（Python Interpreter）**由 **Python 编译器（Python Compiler）** 和 **Python 虚拟机（Python Virutal Machine）**组成，当我们通过 **Python** 命令执行一个 .py 文件时，Python 解释器会先通过 **Python 编译器**将 .py 文件中的代码编译为字节码（bytecode），再使用 **Python 虚拟机** 逐步执行生成的字节码。

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

在 Python 编译器将代码编译为字节码之后，Python 虚拟机会读取并逐行执行这些字节码，但代码对象本身只包含了字节码相关的信息，并不具有用于执行字节码所需要的上下文信息，因此需要引入**栈帧对象 struct PyFrameObject**，



















































