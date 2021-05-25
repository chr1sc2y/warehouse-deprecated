# Python 源码学习（3）：list 类型

[TOC]

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

## 1 append

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

## 2 copy

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
