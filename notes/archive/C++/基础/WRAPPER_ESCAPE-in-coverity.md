---
title: "coverity 的 WRAPPER_ESCAPE 告警"
date: 2020-03-15T17:24:27+08:00
draft: false
categories: ["C++"]
---

# coverity 的 WRAPPER_ESCAPE 告警

```c++
const char* Foo()
{
    std::string str_msg("test");
    return str_msg.c_str();
}

int main() {
    const char *p_msg = Foo();
    printf("%s\n", p_msg);
    return 0;
}

// output:（为空，或乱码）
D?
```

上面代码中的 Foo 函数会被 coverity 报告 WRAPPER_ESCAPE，详细说明是：

```
Wrapper object use after free (WRAPPER_ESCAPE)
1. escape: The internal representation of local strMsg escapes, but is destroyed when it exits scope
```

大意是局部变量 str_msg 在离开函数 Foo 的时候会被释放（因为 str_msg 是分配在栈上的变量），而通过函数 std::string::c_str() 获取的指向 str_msg 头部的指针会因此变为一个悬空指针，将这个悬空指针返回给函数调用者使用将会发生不可预知的行为。

而 c_str() 本身返回的是一个 const char *p，虽然我们无法直接修改指针 p 所指向的数据，但我们可以通过修改 str_msg 来达到修改 p 所指向内存的效果，例如如下的代码：

```c++
int main() {
    std::string str_msg("test");
    const char *p_msg = str_msg.c_str();
    printf("%s\n", p_msg);
    str_msg[2] = 'x';
    printf("%s\n", p_msg);
    return 0;
}

// output:
test
text
```

想要正确地使用返回的 const char*，我们可以在堆上分配一块内从，将要使用的字符串拷贝到其中并返回：

```c++
const char* Foo()
{
    std::string str_msg("test");
    uint32_t u32_msg_size = str_msg.size() + 1;
    char *p_return = new char[u32_msg_size];
    strcpy_s(p_return, u32_msg_size, str_msg.c_str());
    return p_return;
}

int main() {
    const char *p_msg = Foo();
    printf("%s\n", p_msg);
    return 0;
}

// output:
test
```

当然，调用者也应该则适当的时机对 p_msg 进行 delete 操作，否则将会造成内存泄漏。

应该记住的是，除非你需要立即以 const char* 的方式使用字符串，否则应该尽量避免使用 c_str()，尤其是在发生函数调用和返回时。