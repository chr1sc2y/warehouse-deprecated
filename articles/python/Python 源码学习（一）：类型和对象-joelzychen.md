# Python 源码学习（一）：类型和对象

[TOC]

Python 是一门解释型，动态类型，多范式的编程语言，当我们从 [python.org](https://www.python.org/) 下载并安装运行 Python 的某个分发版本时，我们实际上是在运行由 C 语言编写的 [CPython](https://github.com/python/cpython)，除此之外 Python 的运行时还有 Jython, PyPy, Cython 等；CPython 的源码中有一系列的库，组件和工具：

```shell
$ git clone https://github.com/python/cpython
$ tree -d -L 2 .
.
`-- cpython
    |-- Doc			# 文档
    |-- Grammar
    |-- Include 	# C 头文件
    |-- Lib			# 用 Python 写的库文件
    |-- Mac			# 用于在 macOS 上构建的文件
    |-- Misc		# 杂项
    |-- Modules		# 用 C 写的库文件
    |-- Objects 	# 核心类型，以及对象模型的定义
    |-- PC			# 用于在 Windows 上构建的文件
    |-- PCbuild 	# 用于在老版本的 Windows 上构建的文件
    |-- Parser		# Python 解析器源码
    |-- Programs	# Python 可执行文件和其他
    |-- Python		# CPython 编译器源码
    |-- Tools		# 构建时的工具
    `-- m4

16 directories
```

本文主要以阅读和分析 CPython 源码的方式，以 int 和 list 类型的部分函数为例，学习 Python 的类型和对象模块。

## 1 对象模型

Python 是一门面向对象的语言，我们可以使用 Python 中的 `type()` 函数查看一个对象所属的类：

```shell
>>> type(1)
<class 'int'>
>>> type(True)
<class 'bool'>
```

可以看到整数对象和布尔值对象的类型分别是 `<class 'int'>` 和 `<class 'bool'>`；

而实际上，在 Python 中无论是整数，布尔值还是基本数据类型，甚至自定义的 class，都是对象：

```python
class Foo:
    pass

print(type(int))
print(type(Foo))
```



```shell
<class 'type'>
<class 'type'>
```

可以看到 `int` 类型和自定义类 `Foo` 的类型都是 `<class 'type'>`，它们是 `type` 这个类的实例对象；`type` 类型是专门用于定义类型的类型，也称为**元类型**；实际上， `type` 这个类型本身也是一个对象，它所属的类也是 `type`：

```shell
>>> type(type) 
<class 'type'>
```

同时，Python 中的所有类型，无论是 `int`, `type`, 还是自定义类 `Foo` 都是继承自一个叫 `object` 的基类，而 `object` 则是继承链的终点：

```shell
>>> int.__base__
<class 'object'>
>>> type.__base__
<class 'object'>
>>> print(Foo.__base__)
<class 'object'>
>>> print(object.__base__)
None
```

而 `object` 基类也是一个 `type` 类型的对象：

```shell
>>> type(object)
<class 'type'>
```

上面的关系用图表达出来则是：

![process](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/python/type-0.png)

可以看到，所有类型的基类都是 `object`，所有类型的类型都是 `type`，这就是 Python 的**对象模型**（object model），也是 Objects/ 目录下源码所包含的内容。

## 2 核心类型与对象

虽然在 Python 的语法层面有非常多所谓的类型（包括 `int`, `type`, `Foo` 等），但实际上它们在源码（C 语言）层面上都是结构体对象。

### 2.1 对象

#### PyObject

Python 中所有的类型都由 **`PyObject`** 结构体扩展而来，这个结构体中有以下几个成员变量：

1. `Py_ssize_t ob_refcnt` 用于保存对象的**引用计数**；
2. `PyTypeObject *ob_type` 指向对象的**类型对象**，用来标识对象属于的类型，并存储类型的**元数据**；
3. `_PyObject_HEAD_EXTRA` 宏代表了两个 `PyObject*` 双向链表的指针，用于把堆上的所有对象链接起来，只会在开启了 `Py_TRACE_REFS` 宏的时候进行构造，方便调试；

```c
// Include/object.h

/* Define pointers to support a doubly-linked list of all live heap objects. */
#define _PyObject_HEAD_EXTRA            \
    struct _object *_ob_next;           \
    struct _object *_ob_prev;

/* Nothing is actually declared to be a PyObject, but every pointer to
 * a Python object can be cast to a PyObject*.  This is inheritance built
 * by hand.  Similarly every pointer to a variable-size Python object can,
 * in addition, be cast to PyVarObject*.
 */
typedef struct _object {
    _PyObject_HEAD_EXTRA	// 双向链表，用于追踪堆中所有对象，在开启了 Py_TRACE_REFS 宏的时候有用
    Py_ssize_t ob_refcnt;	// 引用计数，用于垃圾回收
    PyTypeObject *ob_type;	// 指针，指向当前对象的类型对象，用于查询对象的类型
} PyObject;
```

#### PyVarObject

Python 中有可以自由修改长度的 `PyVarObject` 变长对象，它由一个 `PyObject` 对象和一个存储变长部分的长度（元素个数）的变量 `ob_size` 组成：

```cpp
typedef struct {
    PyObject ob_base;
    Py_ssize_t ob_size; /* Number of items in variable part */
} PyVarObject;
```

`PyObject` 和 `PyVarObject` 一般是作为头部被包含在一个变量结构体中的，根据该变量大小是否固定来选择使用哪一种：

```cpp
// Include/object.h
/* PyObject_HEAD defines the initial segment of every PyObject. */
#define PyObject_HEAD          PyObject ob_base;

/* PyObject_VAR_HEAD defines the initial segment of all variable-size
 * container objects.  These end with a declaration of an array with 1
 * element, but enough space is malloc'ed so that the array actually
 * has room for ob_size elements.  Note that ob_size is an element count,
 * not necessarily a byte count.
 */
#define PyObject_VAR_HEAD      PyVarObject ob_base;
```

Python 中最典型的变长对象就是列表 List，它和 `std::vector` 比较类似，列表对象里有三个成员变量，包括：

- 基础的变长对象 `PyVarObject ob_base`，其中 `ob_base.ob_size` 用于表示列表当前的元素个数；
- 指向动态数组的指针 `PyObject **ob_item`；
- 动态数组当前的容量 `Py_ssize_t allocated`：

```cpp
// Inlucde/cpython/listobject.h

typedef struct {
    PyObject_VAR_HEAD
    /* Vector of pointers to list elements.  list[0] is ob_item[0], etc. */
    PyObject **ob_item;

    /* ob_item contains space for 'allocated' elements.  The number
     * currently in use is ob_size.
     * Invariants:
     *     0 <= ob_size <= allocated
     *     len(list) == ob_size
     *     ob_item == NULL implies ob_size == allocated == 0
     * list.sort() temporarily sets allocated to -1 to detect mutations.
     *
     * Items must normally not be NULL, except during construction when
     * the list is not yet visible outside the function that builds it.
     */
    Py_ssize_t allocated;
} PyListObject;
```

### 2.2 类型

#### PyTypeObject

`PyObject` 类中的 `PyTypeObject *ob_type` 是一个指向**对象类型**的指针，它是**类**在 Python 中的表现形式；`PyTypeObject` 不仅决定了 `PyObject` 对象属于什么类型，还包含了非常多的**元数据**，例如：

1. `PyObject_VAR_HEAD` 表示 `PyTypeObject` 本身是一个**变长对象**；
2. `const char *tp_name` 表示类型的名字；
3. `struct _typeobject *tp_base` 是指向基类的指针，保存类型的继承信息；
4. `Py_ssize_t tp_basicsize, tp_itemsize` 表示创建实力对象时分配的内存大小；
5. `setattrfunc tp_setattr` 设置值，`getattrfunc tp_getattr` 获取值，`destructor tp_dealloc` 析构，`hashfunc tp_hash` 哈希等函数指针表示该类型所支持的标准操作；

```cpp
// Include/object.h

/* PyTypeObject structure is defined in cpython/object.h.
   In Py_LIMITED_API, PyTypeObject is an opaque structure. */
typedef struct _typeobject PyTypeObject;

// Include/cpython/object.h
struct _typeobject {
    PyObject_VAR_HEAD // 即 PyVarObject ob_base;
    const char *tp_name; /* For printing, in format "<module>.<name>" */
    Py_ssize_t tp_basicsize, tp_itemsize; /* For allocation */

    /* Methods to implement standard operations */

    destructor tp_dealloc;
    Py_ssize_t tp_vectorcall_offset;
    getattrfunc tp_getattr;
    setattrfunc tp_setattr;
    PyAsyncMethods *tp_as_async; /* formerly known as tp_compare (Python 2)
                                    or tp_reserved (Python 3) */

    // Strong reference on a heap type, borrowed reference on a static type
    struct _typeobject *tp_base;
    
    /* More standard operations (here for binary compatibility) */
    // ...
};
```

Python 中的每一种**类型对象**都是**全局唯一**的，他们在源码中以**全局变量**的形式存在，例如 `int` 类型：

```cpp
// Objects/longobject.c
PyTypeObject PyLong_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "int",                                      /* tp_name */
    offsetof(PyLongObject, ob_digit),           /* tp_basicsize */
    sizeof(digit),                              /* tp_itemsize */
    0,                                          /* tp_dealloc */
    0,                                          /* tp_vectorcall_offset */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    0,                                          /* tp_as_async */
    long_to_decimal_string,                     /* tp_repr */
    &long_as_number,                            /* tp_as_number */
    // ...
};
```

`PyVarObject_HEAD_INIT` 宏用于初始化 `PyVarObject` 中的 `ob_refcnt`, `ob_type` 和 `ob_size`：

```cpp
#define PyObject_HEAD_INIT(type)        \
    { 1, type },

#define PyVarObject_HEAD_INIT(type, size)       \
    { PyObject_HEAD_INIT(type) size },

```

可以看到在 `PyLong_Type` 中，`ob_type` 被初始化为 `&PyType_Type`，它是专门用于**定义类型对象的类型**，抑或叫做**类型的类型**或**元类型**。

#### PyType_Type

在前面已经了解到，`type` **类型对象**所属的类也是 `type`，那么**类型对象** `PyTypeObject` 本身也是一个类对象，它也拥有指向其**类型对象**的指针 `PyTypeObject *ob_type`；对于**类型对象**本身来说，它的**类型对象**都基于一个叫做 `PyType_Type`（即**元类型**）的 `PyTypeObject` 类对象：

```cpp
// Objects/typeobject.c
PyTypeObject PyType_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)		// PyType_Type 在初始化的时候将指向自身的指针传递并用于构造了一个 PyVarObject 类型的对象 ob_base，其中 ob_base->ob_type = &PyType_Type，参考附录 1
    "type",                                     /* tp_name */
    sizeof(PyHeapTypeObject),                   /* tp_basicsize */
    sizeof(PyMemberDef),                        /* tp_itemsize */
    (destructor)type_dealloc,                   /* tp_dealloc */
    offsetof(PyTypeObject, tp_vectorcall),      /* tp_vectorcall_offset */
    // ...
};
```

Python 中**类型对象**都定义在了 Objects/ 目录下，例如 `bool` 类型：

```cpp
// Objects/boolobject.c

/* The type object for bool.  Note that this cannot be subclassed! */

PyTypeObject PyBool_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)		// 同 PyType_Type，ob_base->ob_type = &PyType_Type
    "bool",										// tp_name = "bool"
    sizeof(struct _longobject),					/* tp_basicsize */
    0,											/* tp_itemsize */
    0,                                          /* tp_dealloc */
    0,                                          /* tp_vectorcall_offset */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    0,                                          /* tp_as_async */
    bool_repr,                                  /* tp_repr */
    &bool_as_number,                            /* tp_as_number */
    // ...
    0,                                          /* tp_base */
    // ...
};
```

在 Python 中，无论是内建类型（`int`, `bool` 等），还是自定义类型（`Foo`），都是通过 `PyTypeObject` 这个结构体构造的，且一定满足 `ob_base->ob_type = &PyType_Type`。

前文提到 `tp_base` 是指向基类的指针，保存类型的继承信息，但实际上在定义 `PyBool_Type` 的时候可以看到 `tp_base = 0`，实际上这个 `tp_base` 是在 `PyType_Ready` 函数中被赋值为 `PyBaseObject_Type` 的：

```cpp
// Objects/typeobject.c

int
PyType_Ready(PyTypeObject *type)
{
    PyTypeObject *base;
    // ...
	/* Initialize tp_base (defaults to BaseObject unless that's us) */
    base = type->tp_base;
    if (base == NULL && type != &PyBaseObject_Type) {
        base = &PyBaseObject_Type;
        if (type->tp_flags & Py_TPFLAGS_HEAPTYPE) {
            type->tp_base = (PyTypeObject*)Py_NewRef((PyObject*)base);
        }
        else {
            type->tp_base = base;
        }
    }
    // ...
}
```

这里 `PyTypeObject *base` 被赋值为的 `PyBaseObject_Type` 就是前文提到的**基类型**。

#### PyBaseObject_Type

`PyBaseObject_Type` 定义在 typeobject.c 文件中：

```cpp
// Objects/typeobject.c
PyTypeObject PyBaseObject_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)		// 同 PyType_Type，ob_base->ob_type = &PyType_Type
    "object",                                   /* tp_name */
    sizeof(PyObject),                           /* tp_basicsize */
    0,                                          /* tp_itemsize */
    object_dealloc,                             /* tp_dealloc */
    // ...
    object_repr,                                /* tp_repr */
	// ...
    0,                                          /* tp_base */
    // ...
};
```

可以看到无论是 `PyBool_Type`，`PyType_Type` 还是 `PyBaseObject_Type`，都有两个共同点：

1. 它们是以同样的方式 `PyVarObject_HEAD_INIT(&PyType_Type, 0)` 定义其类型的，因此它们的类型都是 `PyTypeObject`；
2. 它们指向基类的指针 `tp_base` 初始化时都是 NULL；

不同的地方是，在使用 `PyType_Ready` 函数为它们的基类指针 `tp_base` 赋值时，只有 `PyBaseObject_Type.tp_base` 不会被赋值，其他的 `tp_base` 则都会被赋值为 `PyBaseObject_Type`，这也印证了 `PyBaseObject_Type` 是继承链的终点。

我们可以将上面的关系整理为一个图：

![process](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/python/type-1.png)



## 3 int 类型

Python 中的标准数据类型有六种，分别是 number, string, list, tuple, set, dictionary，前文已经阐述过它们的对象类型都是继承了 `PyBaseObject_Type` 类型的 `PyType_Type` 类型的实例对象，本文则主要探究 Python 中 int 类型的实现。

不同于 C 和 C++ 中的 `int` 类型，Python 中的 `int` 类型最大的特点是它一般是**不会溢出**的，对比用 C 和 Python 分别输出两个一百万相乘的结果：

```python
>>> x = 10000000000
>>> print(x)
10000000000
```

在 C 语言中会发生溢出：

```C++
printf("%d\n", 1000000 * 1000000);
printf("%u\n", 1000000 * 1000000);
```

```shell
-727379968
3567587328
```

### 3.1 int 类型在内存中的存储方式

#### 3.1.1 内存结构

Python 中的 `int` 整数类型实际上是一个名为 `PyLongObject`  的结构体，定义在 `longintrepr.h` 文件中：

```cpp
// Include/object.h
#define PyObject_VAR_HEAD      PyVarObject ob_base;

// Objects/longobject.h
#if PYLONG_BITS_IN_DIGIT == 30
typedef uint32_t digit;
// ...
#elif PYLONG_BITS_IN_DIGIT == 15
typedef unsigned short digit;
// ...
#endif
typedef struct _longobject PyLongObject; /* Revealed in longintrepr.h */

// Include/longintrepr.h
struct _longobject {
    PyObject_VAR_HEAD
    digit ob_digit[1];
};
```

它由两部分组成，分别是：

1. 一个变长对象 `PyVarObject ob_base`，其中包括引用计数 `Py_ssize_t ob_refcnt`、类型指针 `PyTypeObject *ob_type`、变长部分的长度 `Py_ssize_t ob_size`，表明 `PyLongObject` 也是一个**变长对象**；

2. 一个 `digit` 类型的数组 `ob_digit` ，用于存储整数值，数组长度默认为 1，在初始化时如果长度不够则会被扩大； `digit` 是一个被 `PYLONG_BITS_IN_DIGIT ` 宏控制的类型，在编译 Python 解释器时可以通过修改这个宏来指定其类型；如果没有指定 `PYLONG_BITS_IN_DIGIT ` 宏的值，则默认会根据操作系统的类型来决定，当指针占用 8 字节以上空间时（64 位以上操作系统），`PYLONG_BITS_IN_DIGIT = 30`，`digit` 即为 `uint32_t`，否则 `PYLONG_BITS_IN_DIGIT = 15`，`digit` 则是 `unsigned short`：

```cpp
#ifndef PYLONG_BITS_IN_DIGIT
#if SIZEOF_VOID_P >= 8
#define PYLONG_BITS_IN_DIGIT 30
#else
#define PYLONG_BITS_IN_DIGIT 15
#endif
#endif
```

`PyLongObject` 的内存结构大致如图：

![PyLongObject](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/python/PyLongObject.png)

#### 3.1.2 数据表示

在 `ob_digit` 数组中，数据的表示遵循两个原则：

1. `ob_size` 的绝对值表示 `ob_digit` 数组的长度，`ob_size = 0` 表示 `PyLongObject` 的数值等于 0；数据的正负由 `ob_size` 的正负来标识，`ob_size > 0` 表示 `PyLongObject > 0`，`ob_size < 0` 表示 `PyLongObject < 0`；
2. `ob_digit` 数组的每一个元素都是一个最大为 `2^30`（假设 `PYLONG_BITS_IN_DIGIT == 30`）的整数，如果整数超过了这个值，则会清零并使其后一位自增 1，假设 `ob_size = n`，那么数据的绝对值则等于 `ob_digit[0] + ob_digit[1] * 2^30 + ob_digit[2] * 2^60 + ... + ob_digit[n-1] * 2^(30 * (n-1))`；

例如对于整数 4294967297，可以被表示为 `1 + 4 * 2^30`，因此其 `ob_size = 2`, `ob_digit[0] = 1`, `ob_digit[1] = 4`，其内存结构大致如图：

![PyLongObject-1](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/python/PyLongObject-1.png)

通过这种大数存储方式，Python 从语言层面解决了 `2^(30*2147483648) - 1` 以下（`ob_size` 的类型 `Py_ssize_t` 是通过 `typedef long int Py_ssize_t` 定义的）的大数的溢出问题。

#### 3.1.3 创建对象

在 Python 中， `PyLongObject` 对象一般是通过 `_PyLong_New` 函数创建出来的：

```cpp
/* Allocate a new int object with size digits.
   Return NULL and set exception if we run out of memory. */

#define MAX_LONG_DIGITS \
    ((PY_SSIZE_T_MAX - offsetof(PyLongObject, ob_digit))/sizeof(digit))

PyLongObject *
_PyLong_New(Py_ssize_t size)
{
    PyLongObject *result;
    /* Number of bytes needed is: offsetof(PyLongObject, ob_digit) +
       sizeof(digit)*size.  Previous incarnations of this code used
       sizeof(PyVarObject) instead of the offsetof, but this risks being
       incorrect in the presence of padding between the PyVarObject header
       and the digits. */
    if (size > (Py_ssize_t)MAX_LONG_DIGITS) {
        PyErr_SetString(PyExc_OverflowError,
                        "too many digits in integer");
        return NULL;
    }
    result = PyObject_MALLOC(offsetof(PyLongObject, ob_digit) +
                             size*sizeof(digit));
    if (!result) {
        PyErr_NoMemory();
        return NULL;
    }
    _PyObject_InitVar((PyVarObject*)result, &PyLong_Type, size);
    return result;
}
```

这个函数非常简单，主要是做了两件事：

1. 内存分配前后的检查，包括参数 `size` 不能超过 `MAX_LONG_DIGITS`，也就是说 `PyLongObject` 所表示的整数大小不能超过 `2^(30*2147483648) - 1`，以及生成使用 `malloc` 分配内存失败后的报错信息；
2. 为 `PyLongObject` 对象申请内存，其大小分为两部分，第一部分是 `PyVarObject` 在内存对齐后所占用的空间，即 `offsetof(PyLongObject, ob_digit)`；第二部分是 `ob_digit` 数组所占用的空间，其中参数 `size` 是 `ob_digit` 数组的长度。

#### 3.1.4 数据转化

每一个 `PyLongObject` 对象都拥有不同的内存地址，我们可以通过 Python 中的 `id` 函数来查看一个变量的标识，这个标识会因内存地址的不同而改变：

```python
for i in range(5):
	print(id(i))
```



```shell
$ python3 main.py 
139748219328384
139748219328416
139748219328448
139748219328480
139748219328512
```

可以看出 0 到 4 这 5 个数的标识里每两个都相差了 32，刚好符合每一个 `PyLongObject` 对象所占用的空间 32 字节，而不是 C 语言里一个 `long` 类型变量通常所占用的 4 字节或 8 字节，这是因为所有原始的数据都会被转化为 `PyLongObject` 对象。

数据转化的方法有很多，以 `PyLong_FromLong` 为例，它会将一个 `long` 类型的整数转化为 `PyLongObject` 对象：

```cpp
// Objects/longobject.c
/* interpreter state */
#define _PY_NSMALLPOSINTS           257
#define _PY_NSMALLNEGINTS           5

#define NSMALLNEGINTS           _PY_NSMALLNEGINTS
#define NSMALLPOSINTS           _PY_NSMALLPOSINTS

#define IS_SMALL_INT(ival) (-NSMALLNEGINTS <= (ival) && (ival) < NSMALLPOSINTS)

PyObject *
PyLong_FromLong(long ival)
{
    PyLongObject *v;
    unsigned long abs_ival;
    unsigned long t;  /* unsigned so >> doesn't propagate sign bit */
    int ndigits = 0;
    int sign;

    if (IS_SMALL_INT(ival)) {
        return get_small_int((sdigit)ival);
    }

    if (ival < 0) {
        /* negate: can't write this as abs_ival = -ival since that
           invokes undefined behaviour when ival is LONG_MIN */
        abs_ival = 0U-(unsigned long)ival;
        sign = -1;
    }
    else {
        abs_ival = (unsigned long)ival;
        sign = ival == 0 ? 0 : 1;
    }

    /* Fast path for single-digit ints */
    if (!(abs_ival >> PyLong_SHIFT)) {
        v = _PyLong_New(1);
        if (v) {
            Py_SET_SIZE(v, sign);
            v->ob_digit[0] = Py_SAFE_DOWNCAST(
                abs_ival, unsigned long, digit);
        }
        return (PyObject*)v;
    }

#if PyLong_SHIFT==15
    /* 2 digits */
    if (!(abs_ival >> 2*PyLong_SHIFT)) {
        v = _PyLong_New(2);
        if (v) {
            Py_SET_SIZE(v, 2 * sign);
            v->ob_digit[0] = Py_SAFE_DOWNCAST(
                abs_ival & PyLong_MASK, unsigned long, digit);
            v->ob_digit[1] = Py_SAFE_DOWNCAST(
                  abs_ival >> PyLong_SHIFT, unsigned long, digit);
        }
        return (PyObject*)v;
    }
#endif

    /* Larger numbers: loop to determine number of digits */
    t = abs_ival;
    while (t) {
        ++ndigits;
        t >>= PyLong_SHIFT;
    }
    v = _PyLong_New(ndigits);
    if (v != NULL) {
        digit *p = v->ob_digit;
        Py_SET_SIZE(v, ndigits * sign);
        t = abs_ival;
        while (t) {
            *p++ = Py_SAFE_DOWNCAST(
                t & PyLong_MASK, unsigned long, digit);
            t >>= PyLong_SHIFT;
        }
    }
    return (PyObject *)v;
}
```

虽然看起来比较长，但其实思路非常简单：

1. 创建用于存储返回值的指针 `PyLongObject *z`，保存数据绝对值的变量 `unsigned long abs_ival, t`，标识数组长度的 `int ndigits` 和标识数据正负的 `int sign`；
2. 如果数据范围在 [-5, 257) 内，则通过 `get_small_int` 函数返回结果；
3. 获取数据的绝对值和正负符号；
4. 如果数据绝对值没有超过 `ob_digit` 数组的单个元素所能表示的大小，则通过一个快速路径返回结果；
5. 对于较大的数据，确定其 `ob_digit` 数组的长度，逐位置入。

可以注意到，第 2 步针对 [-5, 257) 区间内的小整数做了特殊处理，最终在调用到 `__PyLong_GetSmallInt_internal` 函数时，会通过 `tstate->interp->small_ints[index]` 缓存数组获取小整数对应的指针对象并将其返回，这里的 `small_ints` 数组是一个全局变量，一般称为**小整数对象池**，是针对常用的小整数做的一个优化：

```cpp
// Objects/longobject.c
static inline PyObject* __PyLong_GetSmallInt_internal(int value)
{
    PyThreadState *tstate = _PyThreadState_GET();
#ifdef Py_DEBUG
    _Py_EnsureTstateNotNULL(tstate);
#endif
    assert(-_PY_NSMALLNEGINTS <= value && value < _PY_NSMALLPOSINTS);
    size_t index = _PY_NSMALLNEGINTS + value;
    PyObject *obj = (PyObject*)tstate->interp->small_ints[index];
    // _PyLong_GetZero() and _PyLong_GetOne() must not be called
    // before _PyLong_Init() nor after _PyLong_Fini()
    assert(obj != NULL);
    return obj;
}
```

### 3.2 数学运算

`PyLongObject` 的类型对象是 `PyLong_Type`，`PyLong_Type` 的成员变量  `PyNumberMethods *tp_as_number` 由 `static PyNumberMethods long_as_number*` 结构体指针初始化，其中包含了许多数学运算的函数指针，当我们对 `PyLong_Type` 进行数学运算时，实际上会调用这些函数：

```cpp
// Objects/longobject.c
PyTypeObject PyLong_Type = {
    // ...
    &long_as_number,                            /* tp_as_number */
    // ...
};

static PyNumberMethods long_as_number = {
    (binaryfunc)long_add,       /*nb_add*/
    (binaryfunc)long_sub,       /*nb_subtract*/
    (binaryfunc)long_mul,       /*nb_multiply*/
    long_mod,                   /*nb_remainder*/
    long_divmod,                /*nb_divmod*/
    long_pow,                   /*nb_power*/
    // ...
};
```

#### 3.2.1 加法

`PyLong_Type`  的加法运算对应的函数是 `long_add`，其实现和相关的宏定义如下：

```cpp
// Objects/longobject.c
#define CHECK_BINOP(v,w)                                \
    do {                                                \
        if (!PyLong_Check(v) || !PyLong_Check(w))       \
            Py_RETURN_NOTIMPLEMENTED;                   \
    } while(0)

/* convert a PyLong of size 1, 0 or -1 to an sdigit */
#define MEDIUM_VALUE(x) (assert(-1 <= Py_SIZE(x) && Py_SIZE(x) <= 1),   \
         Py_SIZE(x) < 0 ? -(sdigit)(x)->ob_digit[0] :   \
             (Py_SIZE(x) == 0 ? (sdigit)0 :                             \
              (sdigit)(x)->ob_digit[0]))

static PyObject *
long_add(PyLongObject *a, PyLongObject *b)
{
    PyLongObject *z;

    CHECK_BINOP(a, b);

    if (Py_ABS(Py_SIZE(a)) <= 1 && Py_ABS(Py_SIZE(b)) <= 1) {
        return PyLong_FromLong(MEDIUM_VALUE(a) + MEDIUM_VALUE(b));
    }
    if (Py_SIZE(a) < 0) {
        if (Py_SIZE(b) < 0) {
            z = x_add(a, b);
            if (z != NULL) {
                /* x_add received at least one multiple-digit int,
                   and thus z must be a multiple-digit int.
                   That also means z is not an element of
                   small_ints, so negating it in-place is safe. */
                assert(Py_REFCNT(z) == 1);
                Py_SET_SIZE(z, -(Py_SIZE(z)));
            }
        }
        else
            z = x_sub(b, a);
    }
    else {
        if (Py_SIZE(b) < 0)
            z = x_sub(a, b);
        else
            z = x_add(a, b);
    }
    return (PyObject *)z;
}
```

可以看到它的实现非常简单，主要分为以下几个步骤：

1. 创建一个用于存储返回值的指针 `PyLongObject *z`；
2. 检查两个参数是否都是 `PyLongObject` 类型的指针 `CHECK_BINOP(a, b)`；
3. 如果两个参数都满足 `ob_size <= 1`（即绝对值均小于 `2^30`），那么先通过 `MEDIUM_VALUE` 获取两个 `ob_digit[0]` 的值，并将两数直接相加（一定不会溢出），再通过 `PyLong_FromLong` 将这个数包装为一个 `PyLongObject` 指针并返回；通常我们进行运算的数都不会太大，因此这里可以利用简化的运算步骤和 CPU 分支预测来提高效率；
4. 判断两者的正负关系，将问题简化为绝对值加减法，利用辅助函数 `x_add` 和 `x_sub` 进行运算并返回结果。

#### 3.2.2 绝对值加法

绝对值加法函数 `x_add` 的定义如下：

```cpp
#if PYLONG_BITS_IN_DIGIT == 30
#define PyLong_SHIFT    30
// ...
#endif
#define PyLong_BASE     ((digit)1 << PyLong_SHIFT)
#define PyLong_MASK     ((digit)(PyLong_BASE - 1))

/* Add the absolute values of two integers. */
static PyLongObject *
x_add(PyLongObject *a, PyLongObject *b)
{
    Py_ssize_t size_a = Py_ABS(Py_SIZE(a)), size_b = Py_ABS(Py_SIZE(b));
    PyLongObject *z;
    Py_ssize_t i;
    digit carry = 0;

    /* Ensure a is the larger of the two: */
    if (size_a < size_b) {
        { PyLongObject *temp = a; a = b; b = temp; }
        { Py_ssize_t size_temp = size_a;
            size_a = size_b;
            size_b = size_temp; }
    }
    z = _PyLong_New(size_a+1);
    if (z == NULL)
        return NULL;
    for (i = 0; i < size_b; ++i) {
        carry += a->ob_digit[i] + b->ob_digit[i];
        z->ob_digit[i] = carry & PyLong_MASK;
        carry >>= PyLong_SHIFT;
    }
    for (; i < size_a; ++i) {
        carry += a->ob_digit[i];
        z->ob_digit[i] = carry & PyLong_MASK;
        carry >>= PyLong_SHIFT;
    }
    z->ob_digit[i] = carry;
    return long_normalize(z);
}
```

其步骤大致分为以下几步：

1. 获取两个参数的 `ob_size` 的绝对值，创建存储返回值的指针 `PyLongObject *z`，
2. 如果 `a->ob_size < b->ob_size` 则交换两者，保证 a 的值较大；
3. 将 z 的 `ob_size` 设置为 `size_a + 1`，保证不会溢出；
4. 以 `i = 0` 为下标，从小到大依次对两数的每一位进行相加操作，将没有超过 `2^30` 的部分存储在 `z->ob_digit[i]` 中，超过的部分进位保存在 `carry` 中；这里充分利用了两个小于 `2^30` 的数相加不会溢出的特性；如果 `size_a > size_b` 还需要将 `a->ob_digit` 多余的部分按照相同的方法置入 `z->ob_digit`里；
5. 得到的结果 `z` 中，其 `ob_digit` 的最后一个元素可能等于 0，因此通过 `long_normalize` 函数将其转换为符合 `PyLongObject` 定义的格式返回。

整个过程可以抽象为类似 `2^30` 进制的加法，和十进制加法的过程几乎完全一样，只不过是把十进制每一位的数字换成了一个最大值为 `2^30` 的数字，每逢 `2^30` 进一位；举个例，假设有 `a->ob_digit = {4, 5, 6}`, `b->ob_digit = {1073741823, 1073741823}`，那么整个加法的步骤为：

1. `v->ob_digit[0] = (4 + 1073741823) % 1073741824 = 3`, `carry = 1`；
2. `v->ob_digit[1] = (5 + 1073741823 + 1) % 1073741824 = 5`, `carry = 1`；
3. `v->ob_digit[2] = (6 + 1) % 1073741824 = 7`, `carry = 0`；
4. `v->ob_digit[3] = 0`；

结果即 `v->ob_digit = {3, 5, 7, 0}`，最后一个元素 0 需要通过 `long_normalize` 函数去掉。

#### 3.2.3 绝对值减法

绝对值减法的实现如下：

```cpp
/* Subtract the absolute values of two integers. */
static PyLongObject *
x_sub(PyLongObject *a, PyLongObject *b)
{
    Py_ssize_t size_a = Py_ABS(Py_SIZE(a)), size_b = Py_ABS(Py_SIZE(b));
    PyLongObject *z;
    Py_ssize_t i;
    int sign = 1;
    digit borrow = 0;

    /* Ensure a is the larger of the two: */
    if (size_a < size_b) {
        sign = -1;
        { PyLongObject *temp = a; a = b; b = temp; }
        { Py_ssize_t size_temp = size_a;
            size_a = size_b;
            size_b = size_temp; }
    }
    else if (size_a == size_b) {
        /* Find highest digit where a and b differ: */
        i = size_a;
        while (--i >= 0 && a->ob_digit[i] == b->ob_digit[i])
            ;
        if (i < 0)
            return (PyLongObject *)PyLong_FromLong(0);
        if (a->ob_digit[i] < b->ob_digit[i]) {
            sign = -1;
            { PyLongObject *temp = a; a = b; b = temp; }
        }
        size_a = size_b = i+1;
    }
    z = _PyLong_New(size_a);
    if (z == NULL)
        return NULL;
    for (i = 0; i < size_b; ++i) {
        /* The following assumes unsigned arithmetic
           works module 2**N for some N>PyLong_SHIFT. */
        borrow = a->ob_digit[i] - b->ob_digit[i] - borrow;
        z->ob_digit[i] = borrow & PyLong_MASK;
        borrow >>= PyLong_SHIFT;
        borrow &= 1; /* Keep only one sign bit */
    }
    for (; i < size_a; ++i) {
        borrow = a->ob_digit[i] - borrow;
        z->ob_digit[i] = borrow & PyLong_MASK;
        borrow >>= PyLong_SHIFT;
        borrow &= 1; /* Keep only one sign bit */
    }
    assert(borrow == 0);
    if (sign < 0) {
        Py_SET_SIZE(z, -Py_SIZE(z));
    }
    return maybe_small_long(long_normalize(z));
}
```

其步骤和绝对值加法类似，大致分为以下几步：

1. 获取两个参数的 `ob_size` 的绝对值，创建存储返回值的指针 `PyLongObject *z`，
2. 如果 `a->ob_size < b->ob_size` 则交换两者，保证 a 的值较大，并用 `sign` 记录运算结果为负；如果 `a->ob_size == b->ob_size` 则从最高位依次往低位找到第一次出现 `a->ob_digit[i] != b->ob_digit[i]` 的位置，比较 `a->ob_digit[i]` 和 `b->ob_digit[i]`，并决定是否交换两者以及 `sign` 的值；
3. 将 z 的 `ob_size` 设置为 `size_a`；
4. 以 `i = 0` 为下标，从小到大依次对两数的每一位进行相减操作，如果被减数 `a->ob_digit[i]`  小于减数 `b->ob_digit[i]` 则要向高位 `a->ob_digit[i + 1]` 借 1；在十进制的减法中向高位借到的数是 10，这里向 `digit` 数组的高位借到的则是 `2^30`；但 `digit` 是通过 `typedef uint32_t digit` 定义出来，通过公式 `borrow = a->ob_digit[i] - b->ob_digit[i] - borrow` 实际上得到的是 `2^32 + a->ob_digit[i] - b->ob_digit[i]`，所以还需要与 `PyLong_MASK` 做 `&` 操作，取后 30 位，便能得到借位相减后的结果，再将结果存储在 `z->ob_digit[i]` 中；借位部分 `borrow` 向右位移 30 位后还剩 2 位，只需要将其与 1 进行 `&` 操作即可知道此次减法运算是否有借位；如果 `size_a > size_b` 还需要将 `a->ob_digit` 多余的部分按照相同的方法置入 `z->ob_digit`里；
5. 得到的结果 `z` 中，其 `ob_digit` 的最后一个元素可能等于 0，因此通过 `long_normalize` 函数将其转换为符合 `PyLongObject` 定义的格式返回。

可以看到这里的步骤也跟十进制减法几乎完全相同。



## 4 list 类型



Python 中的 list 类型在源码中是一个名为 `PyListObject` 的结构体，定义在 `listobject.h` 文件中：

```cpp
// Include/cpython/listobject.h
typedef struct {
    PyObject_VAR_HEAD
    /* Vector of pointers to list elements.  list[0] is ob_item[0], etc. */
    PyObject **ob_item;

    /* ob_item contains space for 'allocated' elements.  The number
     * currently in use is ob_size.
     * Invariants:
     *     0 <= ob_size <= allocated
     *     len(list) == ob_size
     *     ob_item == NULL implies ob_size == allocated == 0
     * list.sort() temporarily sets allocated to -1 to detect mutations.
     *
     * Items must normally not be NULL, except during construction when
     * the list is not yet visible outside the function that builds it.
     */
    Py_ssize_t allocated;
} PyListObject;
```

它的实现和 C++ 中的 `std::vector` 类似，都是通过维护一个动态数组，在增加数据的时候动态扩大数组的容量来实现的；`PyListObject` 结构中包含了一个变长对象头部 `PyObject_VAR_HEAD`，`ob_size` 表示当前动态数组的长度，`**ob_item` 是指向动态数组的指针，`allocated` 是动态数组的容量；我们可以从它的类型指针 `PyTypeObject PyList_Type` 中找到用来操作 list 对象的相关方法：

```cpp
// Objects/listobject.c
PyTypeObject PyList_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "list",
    sizeof(PyListObject),
    list_methods,                               /* tp_methods */
    // ...
};

static PyMethodDef list_methods[] = {
    {"__getitem__", (PyCFunction)list_subscript, METH_O|METH_COEXIST, "x.__getitem__(y) <==> x[y]"},
    LIST___REVERSED___METHODDEF
    LIST___SIZEOF___METHODDEF
    LIST_CLEAR_METHODDEF
    LIST_COPY_METHODDEF
    LIST_APPEND_METHODDEF
    LIST_INSERT_METHODDEF
    LIST_EXTEND_METHODDEF
    LIST_POP_METHODDEF
    LIST_REMOVE_METHODDEF
    LIST_INDEX_METHODDEF
    LIST_COUNT_METHODDEF
    LIST_REVERSE_METHODDEF
    LIST_SORT_METHODDEF
    {"__class_getitem__", (PyCFunction)Py_GenericAlias, METH_O|METH_CLASS, PyDoc_STR("See PEP 585")},
    {NULL,              NULL}           /* sentinel */
};

#define LIST_APPEND_METHODDEF    \
    {"append", (PyCFunction)list_append, METH_O, list_append__doc__},

PyDoc_STRVAR(list_append__doc__,
"append($self, object, /)\n"
"--\n"
"\n"
"Append object to the end of the list.");

#define LIST_COPY_METHODDEF    \
    {"copy", (PyCFunction)list_copy, METH_NOARGS, list_copy__doc__},

PyDoc_STRVAR(list_copy__doc__,
"copy($self, /)\n"
"--\n"
"\n"
"Return a shallow copy of the list.");
```

Python 中也把所谓的函数封装成了一个叫做 `PyMethodDef` 的类型，其中包括了函数的名称 `*ml_name`、对应的 C 函数实现 `ml_meth`、C 函数所需要的标志 `ml_flags`，以及函数说明 `*ml_doc`：

```cpp
// Include/methodobject.h
struct PyMethodDef {
    const char  *ml_name;   /* The name of the built-in function/method */
    PyCFunction ml_meth;    /* The C function that implements it */
    int         ml_flags;   /* Combination of METH_xxx flags, which mostly
                               describe the args expected by the C func */
    const char  *ml_doc;    /* The __doc__ attribute, or NULL */
};
typedef struct PyMethodDef PyMethodDef;
```



### 4.1 append

如 `list_append__doc__` 中所描述的，`list_append` 函数的目的是向 list 的末尾添加新的元素：

```cpp
// Objects/listobject.c
static PyObject *
list_append(PyListObject *self, PyObject *object)
/*[clinic end generated code: output=7c096003a29c0eae input=43a3fe48a7066e91]*/
{
    if (app1(self, object) == 0)
        Py_RETURN_NONE;
    return NULL;
}

static int
app1(PyListObject *self, PyObject *v)
{
    Py_ssize_t n = PyList_GET_SIZE(self);

    assert (v != NULL);
    assert((size_t)n + 1 < PY_SSIZE_T_MAX);
    if (list_resize(self, n+1) < 0)
        return -1;

    Py_INCREF(v);
    PyList_SET_ITEM(self, n, v);
    return 0;
}

static int
list_resize(PyListObject *self, Py_ssize_t newsize)
{
    PyObject **items;
    size_t new_allocated, num_allocated_bytes;
    Py_ssize_t allocated = self->allocated;

    /* Bypass realloc() when a previous overallocation is large enough
       to accommodate the newsize.  If the newsize falls lower than half
       the allocated size, then proceed with the realloc() to shrink the list.
    */
    if (allocated >= newsize && newsize >= (allocated >> 1)) {
        assert(self->ob_item != NULL || newsize == 0);
        Py_SET_SIZE(self, newsize);
        return 0;
    }

    /* This over-allocates proportional to the list size, making room
     * for additional growth.  The over-allocation is mild, but is
     * enough to give linear-time amortized behavior over a long
     * sequence of appends() in the presence of a poorly-performing
     * system realloc().
     * Add padding to make the allocated size multiple of 4.
     * The growth pattern is:  0, 4, 8, 16, 24, 32, 40, 52, 64, 76, ...
     * Note: new_allocated won't overflow because the largest possible value
     *       is PY_SSIZE_T_MAX * (9 / 8) + 6 which always fits in a size_t.
     */
    new_allocated = ((size_t)newsize + (newsize >> 3) + 6) & ~(size_t)3;
    /* Do not overallocate if the new size is closer to overallocated size
     * than to the old size.
     */
    if (newsize - Py_SIZE(self) > (Py_ssize_t)(new_allocated - newsize))
        new_allocated = ((size_t)newsize + 3) & ~(size_t)3;

    if (newsize == 0)
        new_allocated = 0;
    num_allocated_bytes = new_allocated * sizeof(PyObject *);
    items = (PyObject **)PyMem_Realloc(self->ob_item, num_allocated_bytes);
    if (items == NULL) {
        PyErr_NoMemory();
        return -1;
    }
    self->ob_item = items;
    Py_SET_SIZE(self, newsize);
    self->allocated = new_allocated;
    return 0;
}
```

可以看到在 `list_append` 中先调用了 `list_resize`，这个函数可能会进行两个操作：

1. 在添加元素后，如果新的动态数组长度 `newsize` 在区间  `[allocated / 2, allocated]` 内（小于当前容量 `allocated` 且大于等于当前容量的一半 `allocated >> 1`），则将数组容量缩小为 `newsize`；
2. 否则通过公式 `new_allocated = ((size_t)newsize + (newsize >> 3) + 6) & ~(size_t)3` 计算出 `append` 之后动态数组应该被分配的新容量 `new_allocated`，并重新分配内存。

其中第二步的公式不太直观，可以列表观察具体值的变化：



| 动态数组长度 ob_size | 当前容量 allocated | append 后新的长度 newsize | append 后新的容量 new_allocated            |
| -------------------- | ------------------ | ------------------------- | ------------------------------------------ |
| 0                    | 0                  | 1                         | (1 + 0 + 6) & 252 = 111 & 11111100 = 4     |
| 3                    | 4                  | 4                         | 4 ∈ [2, 4]（不变）                         |
| 4                    | 8                  | 5                         | (5 + 0 + 6) & 252 = 1011 & 11111100 = 8    |
| 7                    | 8                  | 8                         | 8 ∈ [4, 8]（不变）                         |
| 8                    | 16                 | 9                         | (9 + 1 + 6) & 252 = 10000 & 11111100 = 16  |
| 15                   | 16                 | 16                        | 16 ∈ [8, 16]（不变）                       |
| 16                   | 16                 | 17                        | (16 + 2 + 6) & 252 = 10011 & 11111100 = 24 |



可以观察到，只有当 append 后新的长度 `newsize` 大于当前容量 `allocated` 时，才会将容量调整为一个更大的值，这个值以 4 的倍数来补足和填充；使用 `python` 测试代码来验证上表的计算结果：



```python
import sys

l = []
s = sys.getsizeof(l)
print((sys.getsizeof(l) - s) // 8)

for _ in range(17):
	l.append(0)
	print("newsize", len(l), "new_allocated", (sys.getsizeof(l) - s) // 8)

```



```shell
$ python3 main.py 
newsize 1 new_allocated 4
newsize 2 new_allocated 4
newsize 3 new_allocated 4
newsize 4 new_allocated 4
newsize 5 new_allocated 8
newsize 6 new_allocated 8
newsize 7 new_allocated 8
newsize 8 new_allocated 8
newsize 9 new_allocated 16
newsize 10 new_allocated 16
newsize 11 new_allocated 16
newsize 12 new_allocated 16
newsize 13 new_allocated 16
newsize 14 new_allocated 16
newsize 15 new_allocated 16
newsize 16 new_allocated 16
newsize 17 new_allocated 24
```



使用 Python3.9 前后的版本测试较大数据时可能会有出入，因为计算 `new_allocated` 的过程进行过修改：



![list_resize](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/python/list_resize.png)



和 `std::vector` 类似，由摊还分析的方法可知 `list_append` 的平均时间复杂度为 *O*(*1*)。



### 4.2 copy

如 `list_copy__doc__` 中所描述的，`list_copy` 函数的目的是返回一个**浅拷贝**（[shallow copy](https://en.wikipedia.org/wiki/Object_copying)）的 list：

```cpp
static PyObject *
list_copy(PyListObject *self, PyObject *Py_UNUSED(ignored))
{
    return list_copy_impl(self);
}

static PyObject *
list_copy_impl(PyListObject *self)
{
    return list_slice(self, 0, Py_SIZE(self));
}

static PyObject *
list_slice(PyListObject *a, Py_ssize_t ilow, Py_ssize_t ihigh)
{
    PyListObject *np;
    PyObject **src, **dest;
    Py_ssize_t i, len;
    len = ihigh - ilow;
    np = (PyListObject *) list_new_prealloc(len);
    if (np == NULL)
        return NULL;

    src = a->ob_item + ilow;
    dest = np->ob_item;
    for (i = 0; i < len; i++) {
        PyObject *v = src[i];
        Py_INCREF(v);
        dest[i] = v;
    }
    Py_SET_SIZE(np, len);
    return (PyObject *)np;
}
```



从 `PyListObject` 的定义中可以得知动态数组指针 `PyObject **ob_item` 所指向的动态数组中存储的是对象的指针，因此在 `list_slice` 函数中，也只是简单地将 `PyListObject *a` 中每一个 `PyObject` 的指针依次赋予 `PyListObject *np`，并将其引用计数加 1；对于被拷贝的 list 中的任意一个值得修改都会反映到拷贝到的 list 上：



```python
a = [1, True, [1, 2]]
b = a
print(a, b)
a[0], a[1], a[2] = 0, False, [3, 4]
print(a, b)
```



这里的 `b = a` 中在 C++ 中一般表示拷贝构造或拷贝赋值操作，但在 Python 中实际上则会调用 `list_copy`：



```shell
$ python3 main.py 
[1, True, [1, 2]] [1, True, [1, 2]]
[0, False, [3, 4]] [0, False, [3, 4]]
```



如果想要对一个 list 进行深拷贝，可以调用 `copy` 模块的 `deepcopy` 函数，这是一个用 Python 实现的模块：



```python
def deepcopy(x, memo=None, _nil=[]):
    """Deep copy operation on arbitrary Python objects.

    See the module's __doc__ string for more info.
    """

    if memo is None:
        memo = {}

    d = id(x)
    y = memo.get(d, _nil)
    if y is not _nil:
        return y

    cls = type(x)

    copier = _deepcopy_dispatch.get(cls)
    if copier is not None:
        y = copier(x, memo)
    else:
        if issubclass(cls, type):
            y = _deepcopy_atomic(x, memo)
        else:
            copier = getattr(x, "__deepcopy__", None)
            if copier is not None:
                y = copier(memo)
            else:
    	# ...

    # If is its own copy, don't memoize.
    if y is not x:
        memo[d] = y
        _keep_alive(x, memo) # Make sure x lives at least as long as d
    return y
```



`deepcopy` 会通过 `_deepcopy_dispatch.get` 来获取内置容器的拷贝器，将内置容器中的数据依次递归地进行拷贝；为了防止某些容器存储的值当中包含指向自己的指针，或是无限重复的数据，函数中会使用一个 dict 变量 `memo` 来记录已经被拷贝过的数据，防止 `deepcopy` 无限地递归下去。



如果容器中存储的是自定义类型的对象，`deepcopy` 会通过 `copier = getattr(x, "__deepcopy__", None)` 获取到这个类型中的函数 `__deepcopy__`，并将其作为一个拷贝器用来生成新的对象，这也就意味着我们需要实现 `__deepcopy__` 函数来保证它可以被正确地深拷贝，以一个自定义的有向图结构为例：



```python
import copy

class DirectedGraphNode:

    def __init__(self, idx, node_list):
        self.idx = idx
        self.node_list = node_list

    def point_to(self, node):
        self.node_list.append(node)

    def __repr__(self):
        return 'id {}, idx {}, node_list {}'.format(id(self), self.idx, [node.idx for node in self.node_list])

    def __deepcopy__(self, memo):
        print(f"DirectedGraphNode: __deepcopy__ from {repr(self)}")
        if self in memo:
            exist_obj = memo.get(self)
            return exist_obj
        cp_obj = DirectedGraphNode(self.idx, [])
        memo[self] = cp_obj
        for node in self.node_list:
            cp_obj.point_to(copy.deepcopy(node, memo))
        print(f"    copy done, self: {repr(self)}")
        return cp_obj

a = DirectedGraphNode(1, [])
b = DirectedGraphNode(2, [])
a.point_to(b)
b.point_to(a)
print(repr(a))
print(repr(b))

c = copy.deepcopy(a)
print(repr(c))
for node in c.node_list:
    print(repr(node))
```



在它的 `__deepcopy__` 函数中，我们以其中一个节点出发，先构造出新的节点对象 `cp_obj`，再将它指向的所有节点以递归的方式依次进行深拷贝，如果在拷贝的过程中发现节点是已经被拷贝过的，则直接返回 `exist_obj`：



```shell
$ python3 main.py 
id 139867268624336, idx 1, node_list [2]
id 139867268624240, idx 2, node_list [1]
DirectedGraphNode: __deepcopy__ from id 139867268624336, idx 1, node_list [2]
DirectedGraphNode: __deepcopy__ from id 139867268624240, idx 2, node_list [1]
DirectedGraphNode: __deepcopy__ from id 139867268624336, idx 1, node_list [2]
    copy done, self: id 139867268624240, idx 2, node_list [1]
    copy done, self: id 139867268624336, idx 1, node_list [2]
id 139867268624048, idx 1, node_list [2]
id 139867268623040, idx 2, node_list [1]
```