##  移动语义

### 值类别

C++ 中的值类别（[Value Categories](https://en.cppreference.com/w/cpp/language/value_category)）根据是否具名和是否可被移动可以被分为 5 种，简单地来说我们平时常用的只有左值 lvalue 和右值 rvalue，其中左值拥有标识符且可以被取地址并访问，而右值是不具名的临时变量，不能被取地址，但能够被移动。

![glvalue&rvalue(0)](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/cpp/glvalue&rvalue(0).png)

![glvalue&rvalue(1)](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/cpp/glvalue&rvalue(1).png)

C++ 11 中引入了右值引用（[Rvalue Reference](http://thbecker.net/articles/rvalue_references/section_01.html)），移动语义（Move Semantics）和完美转发（Perfect Forwarding），它实现了移动语义 move semantics 和完美转发 perfect forwarding。

和通用引用（[Universal Reference](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)）

移动语义指资源所有权的转让，即在进行数据转移（例如函数返回）时，将内存空间中所储存值的所有权直接转移，而无需使用额外的内存空间进行数据的拷贝；进行移动操作的函数是 `std::move`，它会将传入的万能引用使用 `std::remove_reference<_Tp>` 移除引用，转化为右值引用：

```cpp
  /**
   *  @brief  Convert a value to an rvalue.
   *  @param  __t  A thing of arbitrary type.
   *  @return The parameter cast to an rvalue-reference to allow moving it.
  */
  template<typename _Tp>
    constexpr typename std::remove_reference<_Tp>::type&&
    move(_Tp&& __t) noexcept
    { return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); }
```

