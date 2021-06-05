





这是因为 string 本身是一个对 char* 的一个封装，在 string 头文件的源码中我们可以看到如下代码：

```c++
using string = basic_string<char>;
using u16string = basic_string<char16_t>;
using u32string = basic_string<char32_t>;
using wstring = basic_string<wchar_t>;
```

string, u16string, u32string, wstring 这四种类型其实都是由模板类 basic_string 所衍生出来的



### 1.2 编译

对于 C/C++ 程序，在使用 gcc/clang 编译的时候需要加上参数 -g，才能生成完整的调试信息并在 GDB 中调试：

```cpp
$ clang++ -g -std=c++11 -m64 -o test test.cpp
```

其中参数 -std=c++11 代表使用 C++11 标准进行编译，-m64 代表生成 64 位的二进制程序（Linux 下 ELF 格式的二进制可执行程序的第 5 个字节代表了这个程序的位宽，01 代表 32位，02 代表 64 位）：

```
$ od -t x1 -t c test | head -n 2

0000000  7f  45  4c  46  02  01  01  00  00  00  00  00  00  00  00  00
        177   E   L   F 002 001 001  \0  \0  \0  \0  \0  \0  \0  \0  \0
```

## 