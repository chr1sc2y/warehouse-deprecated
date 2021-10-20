# Python 源码学习：协程

[TOC]

**协程** *coroutine* 是一种用户态的轻量级线程，它可以在函数的特定位置暂停或恢复，同时调用者可以从协程中获取状态或将状态传递给协程；Python中的**生成器** *generator* 就是一个典型的协程应用，本文简单地对 Python 中生成器的实现进行分析。

## 1 生成器

如果 Python 中的函数含有 yield 关键字，那么在调用这个函数时，它不会如同普通的函数一样运行到 `return` 语句并返回一个变量，而是会立即返回一个生成器对象；以一个斐波那契数列生成函数为例：

```python
def FibonacciSequenceGenerator():
    a, b = 0, 1
    while True:
        yield a + b
        a, b = b, a + b

if __name__ == "__main__":
    fsg = FibonacciSequenceGenerator()
    print(fsg)
    print(type(fsg))
```



```shell
$ python3 main.py
<generator object FibonacciSequenceGenerator at 0x7fb4720b1ac0>
<class 'generator'>
```

可以看到函数 `FibonacciSequenceGenerator` 返回了一个类型为 `generator` 的生成器对象 `f`；对于生成器对象，我们不能像操作普通函数一样直接进行函数调用，而是要使用 `next()` 或 `fsg.send()` 来进行函数切换，使得生成器函数开始或继续执行，直到 `yield` 所在行或是函数末尾再将执行权交还给调用方：

```python
    for i in range(100):
        print(next(fsg))
```



```shell
$ python3 main.py
1
2
3
5
# ...
218922995834555169026
354224848179261915075
573147844013817084101
```

生成器的这种行为与线程切换非常类似，它包含了执行，保存，恢复上下文的步骤，用生成器来模拟线程的行为可以避免从用户态到内核态的切换，从而提升效率。

## 2 协程

### 2.1 生成器对象

通过类型对象的 `tp_name` 变量可以找到生成器 `generator` 对象在源码中对应的结构体是 `PyGenObject`，它的类型对象是 `PyGen_Type`：

```c++
// Inlcude/genobject.h

/* _PyGenObject_HEAD defines the initial segment of generator
   and coroutine objects. */
#define _PyGenObject_HEAD(prefix)                                           \
    PyObject_HEAD                                                           \
    /* Note: gi_frame can be NULL if the generator is "finished" */         \
    PyFrameObject *prefix##_frame;                                          \
    /* True if generator is being executed. */                              \
    char prefix##_running;                                                  \
    /* The code object backing the generator */                             \
    PyObject *prefix##_code;                                                \
    /* List of weak reference. */                                           \
    PyObject *prefix##_weakreflist;                                         \
    /* Name of the generator. */                                            \
    PyObject *prefix##_name;                                                \
    /* Qualified name of the generator. */                                  \
    PyObject *prefix##_qualname;                                            \
    _PyErr_StackItem prefix##_exc_state;

typedef struct {
    /* The gi_ prefix is intended to remind of generator-iterator. */
    _PyGenObject_HEAD(gi)
} PyGenObject;

// Inlcude/genobject.c


PyTypeObject PyGen_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "generator",                                /* tp_name */
    sizeof(PyGenObject),                        /* tp_basicsize */
    0,                                          /* tp_itemsize */
    /* methods */
    (destructor)gen_dealloc,                    /* tp_dealloc */
    // ...
    (iternextfunc)gen_iternext,                 /* tp_iternext */
    _PyGen_Finalize,                            /* tp_finalize */
};

```

`PyGenObject` 结构体中的成员变量不多，分别是：

- 定长对象公共头部 `PyObject_HEAD`，其中包括引用计数 `ob_refcnt` 和类型对象指针 `ob_type`；
- 生成器是否正在运行的标志 `gi_running`；
- 生成器运行时依赖的栈帧对象 `gi_frame`；
- 生成器对应的代码对象 `gi_code`；
- 生成器的弱引用列表 `gi_weakreflist`；
- 生成器的名称 `gi_name` 和 `gi_qualname`；
- 生成器的异常状态 `gi_exec_state`；

生成器在刚被创建时，并不会立即执行，并且其栈帧对象中最后一条执行过的指令也为空：

```python
    print(fsg.gi_running)
    print(fsg.gi_frame.f_lasti)
```

```shell
$ python3 main.py 
False
-1
```

当然它的栈帧对象也是和代码对象相关联的：

```python
    print(fsg.gi_code)
    print(fsg.gi_frame)
    print(fsg.gi_frame.f_code)
```

```shell
$ python3 main.py 
<code object FibonacciSequenceGenerator at 0x7f8283e49c90, file "main.py", line 89>
<frame at 0x7f8283f17610, file 'main.py', line 89, code FibonacciSequenceGenerator>
<code object FibonacciSequenceGenerator at 0x7f8283e49c90, file "main.py", line 89>
```

### 2.2 执行

#### next

`next()` 函数是 Python 的内置函数，用于驱动生成器的执行，或者说将程序的调用栈从当前函数切换到生成器中；其源码如下：

```cpp
// Python/bltinmodule.c
builtin_next(PyObject *self, PyObject *const *args, Py_ssize_t nargs)
{
    PyObject *it, *res;

    if (!_PyArg_CheckPositional("next", nargs, 1, 2))
        return NULL;

    it = args[0];
    if (!PyIter_Check(it)) {
        PyErr_Format(PyExc_TypeError,
            "'%.200s' object is not an iterator",
            Py_TYPE(it)->tp_name);
        return NULL;
    }

    res = (*Py_TYPE(it)->tp_iternext)(it);
    if (res != NULL) {
        return res;
    } else if (nargs > 1) {
        PyObject *def = args[1];
        if (PyErr_Occurred()) {
            if(!PyErr_ExceptionMatches(PyExc_StopIteration))
                return NULL;
            PyErr_Clear();
        }
        Py_INCREF(def);
        return def;
    } else if (PyErr_Occurred()) {
        return NULL;
    } else {
        PyErr_SetNone(PyExc_StopIteration);
        return NULL;
    }
}

```

除去前后两大块类型检查，最核心的部分是 `res = (*it->ob_type->tp_iternext)(it)`，即获取参数中 `args[0]` 的类型对象，并调用其 `tp_iternext` 函数指针，再检查返回的结果；因此当我们对生成器对象 `fsg` 调用 `next` 函数时，实际上是调用了生成器类型对象 `PyGen_Type` 的 `gen_iternext` 函数，来驱动生成器开始或继续运行，而这个 `gen_iternext` 函数则调用了 `gen_send_ex` 函数：

```cpp
// Objects/genobject.c
static PyObject *
gen_iternext(PyGenObject *gen)
{
    return gen_send_ex(gen, NULL, 0, 0);
}

static PyObject *
gen_send_ex(PyGenObject *gen, PyObject *arg, int exc, int closing)
{
    PyFrameObject *f = gen->gi_frame;
    // ...
    
    /* Generators always return to their most recent caller, not
     * necessarily their creator. */
    Py_XINCREF(tstate->frame);
    assert(f->f_back == NULL);
    f->f_back = tstate->frame;
    
    gen->gi_running = 1;
    gen->gi_exc_state.previous_item = tstate->exc_info;
    tstate->exc_info = &gen->gi_exc_state;

    if (exc) {
        assert(_PyErr_Occurred(tstate));
        _PyErr_ChainStackItem(NULL);
    }

    result = _PyEval_EvalFrame(tstate, f, exc);
    tstate->exc_info = gen->gi_exc_state.previous_item;
    gen->gi_exc_state.previous_item = NULL;
    gen->gi_running = 0;

    // ...
}
```

`gen_send_ex` 函数很长，但其中最关键的只有上面截取的部分，这部分代码首先将生成器对象的栈帧挂到当前调用链上 `f->f_back = tstate->frame;`，并修改生成器对象的运行状态 `gen->gi_running = 1`，接下来再通过调用 `_PyEval_EvalFrame` 来执行生成器栈帧，关于 `_PyEval_EvalFrame` 函数的流程已经在前文讨论过，简单地来说就是通过在一个循环中通过 `switch case` 不断地执行生成器对象对应的字节码。

#### send

和 `next` 函数类似，`send` 函数也可以用来驱动生成器执行，从调用形式上来看它们之间唯一的区别就是 `send` 函数会额外的传入一个参数，而从 `send` 函数的源码中可以看到它同样地也会调用 `gen_send_ex` 函数，不过其第二个参数是一个对象指针（而不像 `gen_iternext` 函数中一样传入了一个 `NULL`）：

```cpp
// Objects/genobject.c
static PyMethodDef gen_methods[] = {
    {"send",(PyCFunction)_PyGen_Send, METH_O, send_doc},
    // ...
};

PyObject *
_PyGen_Send(PyGenObject *gen, PyObject *arg)
{
    return gen_send_ex(gen, arg, 0, 0);
}
```

查看一下两者对应的字节码指令中的区别：

```python
import os, sys

def FibonacciSequenceGenerator():
    a, b = 0, 1
    while True:
        yield a + b
        a, b = b, a + b

def DriveCo(co):
    r = next(fsg)
    r = fsg.send(1)

if __name__ == "__main__":
    fsg = FibonacciSequenceGenerator()
    DriveCo(fsg)
    import dis
    dis.dis(DriveCo)
```

```shell
$ python3 main.py 
 10           0 LOAD_GLOBAL              0 (next)
              2 LOAD_GLOBAL              1 (fsg)
              4 CALL_FUNCTION            1
              6 STORE_FAST               1 (r)

 11           8 LOAD_GLOBAL              1 (fsg)
             10 LOAD_METHOD              2 (send)
             12 LOAD_CONST               1 (1)
             14 CALL_METHOD              1
             16 STORE_FAST               1 (r)
             18 LOAD_CONST               0 (None)
             20 RETURN_VALUE
```

可以看到相比于 `next` 函数的字节码指令，`send` 函数只是在调用函数（`CALL_FUNCTION` / `CALL_METHOD`）前加上了 `LOAD_CONST` 指令来将参数置入栈顶。

### 2.3 暂停

在 Python 中可以使用内置的 `yield` 函数来暂停生成器对象的执行并返回一个值，我们可以研究一下 `FibonacciSequenceGenerator` 函数以及 `yield` 语句对应的字节码指令：

```python
    import dis
    dis.dis(fsg)
```

```shell
$ python3 main.py 
  2           0 LOAD_CONST               1 ((0, 1))
              2 UNPACK_SEQUENCE          2
              4 STORE_FAST               0 (a)
              6 STORE_FAST               1 (b)

  4     >>    8 LOAD_FAST                0 (a)
             10 LOAD_FAST                1 (b)
             12 BINARY_ADD
             14 YIELD_VALUE
             16 POP_TOP

  5          18 LOAD_FAST                1 (b)
             20 LOAD_FAST                0 (a)
             22 LOAD_FAST                1 (b)
             24 BINARY_ADD
             26 ROT_TWO
             28 STORE_FAST               0 (a)
             30 STORE_FAST               1 (b)
             32 JUMP_ABSOLUTE            8
             34 LOAD_CONST               0 (None)
             36 RETURN_VALUE
```

其中第 2 行（`a, b = 0, 1`）和第 5 行（`a, b = b, a + b`）都是在进行 a, b 变量的赋值操作，而在第 4 行则可以看到在执行 `yield a + b` 操作时，实际上是先通过 `LOAD_FAST` 和 `BINARY_ADD` 将新计算出的值保存在栈顶，再进行了 `YIELD_VALUE` 和 `POP_TOP` 指令操作，查看一下这两个指令的源码：

```cpp
// Python/ceval.c

PyObject* _Py_HOT_FUNCTION
_PyEval_EvalFrameDefault(PyThreadState *tstate, PyFrameObject *f, int throwflag)
{
    // ...
        case TARGET(POP_TOP): {
            PyObject *value = POP();
            Py_DECREF(value);
            FAST_DISPATCH();
        }
    // ...
        case TARGET(YIELD_VALUE): {
            retval = POP();

            if (co->co_flags & CO_ASYNC_GENERATOR) {
                PyObject *w = _PyAsyncGenValueWrapperNew(retval);
                Py_DECREF(retval);
                if (w == NULL) {
                    retval = NULL;
                    goto error;
                }
                retval = w;
            }

            f->f_stacktop = stack_pointer;
            goto exiting;
        }
    // ...
    return _Py_CheckFunctionResult(tstate, NULL, retval, __func__);
}
```

可以看到 `YIELD_VALUE`  指令的操作非常简单，它会先取出栈顶的数据作为 yield 语句的返回值，再修改指向当前栈帧顶部的指针所指向的地址，最后通过 `goto` 语句结束执行；随后，`_PyEval_EvalFrameDefault` 函数会将生成器对象的栈帧从当前调用链中移除，并返回 `retval`。



## 3 总结

至此可以发现 Python 中的生成器是完全符合协程的定义（可以在用户态中断和恢复）的；同时由于 Python 虚拟机本身的实现是 **基于栈的**（*Stack-Based*），因此生成器对象在字节码层面的实现也是非常简洁的。而协程可以在线程的基础之上，借助基于 epoll 等 IO 多路复用的事件循环，来驱动不同的子程序（*subroutine*）和上下文的执行，实现形似阻塞式但实际是异步的编程模型，适用于 IO 密集型场景。

### References

`[1]` CPython: *https://github.com/python/cpython*







































































































