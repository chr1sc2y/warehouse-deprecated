# Python 源码学习（1）：类型和对象

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

本系列主要以阅读和分析 CPython 源码的方式学习 Python。

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

## 附录

### 1 PyType_Type 构造

```cpp
struct Bar;

struct Foo
{
	Bar* p_b;
	int ref_count;
};

struct Bar
{
	Foo f;
	const char* name;
};

Bar b
{
	Foo{ &b, 1 },
	"ClassBar"
};

int main()
{
	printf("%s\n", b.f.p_b->name);
	return 0;
}
```

```shell
$ ./test
ClassBar
```



