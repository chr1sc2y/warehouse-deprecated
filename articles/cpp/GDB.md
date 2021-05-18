# GDB 调试入门

## 0. 序

调试程序是开发过程中必不可少的一环，在 Windows 或 MacOS 上开发时，可以使用 VS 和 CLion 等 IDE 上自带的调试功能来打断点或查看变量和堆栈，但 Linux 并没有图形化的操作界面，而如果只通过打 log 的方式来查找问题的话效率将会非常低下，此时我们可以利用 GDB 来提升我们的开发效率。

[GDB](https://en.wikipedia.org/wiki/GNU_Debugger) 是 GNU Debugger 的简写，是 [GNU](https://zh.wikipedia.org/wiki/GNU) 软件系统中的标准[调试器](https://en.wikipedia.org/wiki/Debugger)。GDB 具备各种调试功能，包括但不限于打断点、单步执行、打印变量、查看寄存器、查看函数调用堆栈等，能够有效地针对函数的运行进行追踪和警告；使用 GDB 调试时，可以监督和修改程序的变量，并且这些修改是独立于主程序之外的。GDB 主要用于调试编译型语言，对 C，C++，Go，Fortran 等语言有内置的支持，但它不支持解释型语言。

## 1. 环境搭建

### 1.1 编写程序

为了进行调试，我们需要准备一个简单的 C++ 程序：

```cpp
$ cat test.cpp
#include <iostream>

void Func(const char *s) {
    int *p = nullptr;
    int &r = static_cast<int&>(*p);

    int num = std::atoi(s);
    r = num;
    printf("%d\n", r);
}

int main (int argc, char *argv[]) {
    if (argc != 2) {
        printf("test [int]\n");
        return -1;
    }
    Func(argv[1]);
    return 0;
}

```

### 1.2 编译

对于 C/C++ 程序，在使用 gcc/clang 编译的时候需要加上参数 -g，才能生成完整的调试信息并在 GDB 中调试：

```cpp
$ clang++ -g -std=c++11 -m64 -o test test.cpp
```

## 2. 调试示例

GDB 有非常多的功能，当我们忘记如何使用这些功能时，可以在 GDB 交互界面里输入 ```help``` 或 ```help all``` 来查看指令：

```
(gdb) help
List of classes of commands:
...
```

### 2.1 启动

我们可以使用 ```gdb [executable file]``` 来启动调试：

```
$ gdb test.cpp
...
Reading symbols from test...done.
(gdb)
```

也可以直接使用 ```gdb``` 来进入交互界面，再使用 ```file``` 来指定要调试的程序：

```
$ gdb
...
(gdb) file test
Reading symbols from test...done.
```

### 2.2 运行

对于带参数的程序，我们可以使用 ```set args [arg] ...``` 设置参数，再使用 ```run``` 来运行：

```
(gdb) set args 1
(gdb) run
Starting program: test 1
...
```

当然也可以直接在 ```run``` 后加上参数来运行：

```
(gdb) run 1
Starting program: test 1

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400749 in Func (s=0x7fffffffe167 "/data/home/joelzychen/test/test") at test.cpp:8
8           r = num;
```

我们发现程序出现了 Segmentation fault，此时可以通过打断点来调试程序。

### 2.3 断点

通过错误信息我们可以看到是函数 Func 中，test.cpp 的第 8 行出现了错误，于是我们可以打上两个断点，一个在进入 Func 函数时，一个在 test.cpp 文件的第 8 行，并使用 ```info breakpoints``` 或 ```info b``` 来查看我们设置了哪些断点：

```
(gdb) b test.cpp:8
Breakpoint 1 at 0x400742: file test.cpp, line 8.
(gdb) b Func
Breakpoint 2 at 0x40071c: file test.cpp, line 4.
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400742 in Func(char const*) at test.cpp:8
2       breakpoint     keep y   0x000000000040071c in Func(char const*) at test.cpp:4
```

### 2.4 调试

设置好断点之后就可以使用 ```run``` 开始运行了：

```
(gdb) run
Starting program: test 1

Breakpoint 2, Func (s=0x7fffffffe167 "/data/home/joelzychen/test/test") at test.cpp:4
4           int *p = nullptr;
```

因为我们在 Func 函数的开头设置了一个断点，所以程序在进入 Func 函数时暂停了；我们可以使用 ```list``` 或 ```l [function name]``` 来查看函数的上下文：

```
(gdb) l Func
1       #include <iostream>
2
3       void Func(const char *s) {
4           int *p = nullptr;
5           int &r = static_cast<int&>(*p);
6
7           int num = std::atoi(s);
8           r = num;
9           printf("%d\n", r);
10      }
```

此时我们可以使用 ```print``` 或 ```p``` 来查看变量：

```
(gdb) p s
$1 = 0x7fffffffe187 "1"
(gdb) p *s
$2 = 49 '1'
```

到目前为止还没有出现问题，我们可以使用 ```continue``` 或 ```c``` 走到下一个断点处：

```
(gdb) c
Continuing.

Breakpoint 2, Func (s=0x7fffffffe187 "1") at test.cpp:8
8           r = num;
```

查看一下当前作用域的一些变量：

```
(gdb) p s
$8 = 0x7fffffffe187 "1"
(gdb) p p
$9 = (int *) 0x0
(gdb) p *p
Cannot access memory at address 0x0
(gdb) p r
$10 = (int &) @0x0: <error reading variable>
(gdb) p &r
$11 = (int *) 0x0
(gdb) p num
$12 = 1
```

会发现 r 这个 int& 绑定到了一个被解引用的空指针上，所以 &r == nullptr，因此不能读取 r 的值，我们可以用 ```next``` 或 ```n``` 来进行单步执行，看看接下来会发生什么：

```
(gdb) n

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400749 in Func (s=0x7fffffffe187 "1") at test.cpp:8
8           r = num;
```

果然发生了 Segmentation fault。

## 3. GDB 命令

根据刚才的示例程序，可以整理出一些常用的 GDB 命令。

### 3.1 启动

1. 将 GDB 链接到一个可执行文件并启动

   ```
   gdb [executable file]
   
   gdb
   (gdb) file [executable file]
   ```
   
2. 将 GDB 链接到一个正在运行的进程并启动

   ```
   gdb
   (gdb) attach [PID]
   ```

### 3.2 断点

1. 增加断点

   - ```b [function name]```
   - ```b [file:line]```

2. 查看断点

   - ```info b```

3. 删除断点

   ```
   delete [breakpoint number]
   d [breakpoint number]
   clear # 删除当前行的断点
   clear [function name] # 删除某个函数处的断点
   clear [file:line] # 删除某一行的断点
   ```

4. 禁用和启用断点

   ```
   disable # 禁用所有断点
   disable [breakpoint number] # 禁用某个断点
   enable # 启用所有断点
   enable [breakpoint number] # 启用某个断点
   ```

5. 运行至某一行
   - `until [line]`

### 3.3 运行

1. 从头开始运行
   - ```run```
2. 单步执行
   - ```next``` 或 ```n```
3. 进入函数内部
   - ```step``` 或 ```s```
   - ```stepi``` 执行一条机器指令

### 3.4 查看

1. 查看代码

   ```
   list # 从头开始依次打印，默认每次打印 10 行
   l # 同上
   l [function name] # 从函数定义开始打印
   l [file:line] # 从某一行开始打印
   set listsize 20 # 修改每次打印的行数
   ```

2. 查看变量

   ```
   print [expression] # 打印表达式
   p [expression] # 同上
   ptype [expression] # 打印表达式的类型
   info args # 打印函数参数
   info locals # 打印局部变量
   info registers # 打印寄存器信息
   ```

### 3.5 修改

1. 修改变量

   ```
   set variable i = 10 # 将变量 i 设置为 10
   set var i = 10 # 同上
   p i = 10 # 将变量 i 设置为 10，并打印
   ```

### 3.6 界面

```
layout src: 仅源码
layout asm: 仅汇编代码
layout regs: 显示汇编代码和寄存器信息
layout split: 显示源码和汇编代码
```

### 3.7 调用信息

1. 查看函数调用栈信息
   - ```backtrace``` 或 ```bt```
   - ```where```
2. 查看当前栈帧
   - ```frame``` 或 ```f```

## 4. corefile

[core dump](https://en.wikipedia.org/wiki/Core_dump) / crash dump / memory dump / system dump 都是指一个程序在特定时间崩溃（crash）时的内存记录，它包含了很多关键信息，比如寄存器（包括程序计数器和堆栈指针），内存管理信息，操作系统标志信息等。corefile 就是转储（dump）时的快照，corefile可以被重新执行用以调试错误信息。

### 4.1 生成

为了让系统能够生成 corefile，需要先检查配置：

```
$ ulimit -c
unlimited
```

如果结果是 0 则说明系统禁止了 corefile 的生成，需要执行 ```ulimit -c unlimited``` 来让 corefile 能够正常生成。以刚才的示例程序为例，先执行 test 文件，生成一个 corefile：

```
$ ./test 1
段错误 (core dumped)
$ ll /data/corefile
-rw------- 1 joelzychen dev      450560 4月  22 17:07 core_test_1587546447.28284
```

能够看到系统在特定的目录下（可以修改）生成了一个 corefile 叫做 core_test_1587546447.28284，我们使用 gdb 来对这个 corefile 进行调试。

### 4.2 调试

要执行 corefile 还需要准备对应的可执行文件，运行 ```gdb [executable file] [corefile]``` 就能开始调试了：

```
$ gdb test /data/corefile/core_test_1587546447.28284
...
Core was generated by `./test 1'.
Program terminated with signal 11, Segmentation fault.
#0  0x0000000000400749 in Func (s=0x7ffed098c19e "1") at test.cpp:8
8           r = num;
```

因为示例程序比较简单，因此函数调用栈也比较少，我们可以先使用 ```bt``` 打印函数调用栈信息：

```
(gdb) bt
#0  0x0000000000400749 in Func (s=0x7ffed098c19e "1") at test.cpp:8
#1  0x00000000004007c0 in main (argc=2, argv=0x7ffed098afb8) at test.cpp:17
```

我们通过 ```frame 0``` 或 ```f 0``` 进入程序崩溃的栈帧来查看相关信息：

```
(gdb) f 0
#0  0x0000000000400749 in Func (s=0x7ffed098c19e "1") at test.cpp:8
8           r = num;
(gdb) info args
s = 0x7ffed098c19e "1"
(gdb) info locals
p = 0x0
r = @0x0: <error reading variable>
num = 1
(gdb) ptype r
type = int &
(gdb) ptype p
type = int *
```

查看一下当前栈帧的汇编代码：

```
(gdb) disas
Dump of assembler code for function Func(char const*):
   0x0000000000400710 <+0>:     push   %rbp
   0x0000000000400711 <+1>:     mov    %rsp,%rbp
   0x0000000000400714 <+4>:     sub    $0x20,%rsp
   0x0000000000400718 <+8>:     mov    %rdi,-0x8(%rbp)
   0x000000000040071c <+12>:    movq   $0x0,-0x10(%rbp)
   0x0000000000400724 <+20>:    mov    -0x10(%rbp),%rdi
   0x0000000000400728 <+24>:    mov    %rdi,-0x18(%rbp)
   0x000000000040072c <+28>:    mov    -0x8(%rbp),%rdi
   0x0000000000400730 <+32>:    callq  0x4005a0 <atoi@plt>
   0x0000000000400735 <+37>:    movabs $0x400860,%rdi
   0x000000000040073f <+47>:    mov    %eax,-0x1c(%rbp)
   0x0000000000400742 <+50>:    mov    -0x1c(%rbp),%eax
   0x0000000000400745 <+53>:    mov    -0x18(%rbp),%rcx
=> 0x0000000000400749 <+57>:    mov    %eax,(%rcx)
   0x000000000040074b <+59>:    mov    -0x18(%rbp),%rcx
   0x000000000040074f <+63>:    mov    (%rcx),%esi
   0x0000000000400751 <+65>:    mov    $0x0,%al
   0x0000000000400753 <+67>:    callq  0x400550 <printf@plt>
   0x0000000000400758 <+72>:    mov    %eax,-0x20(%rbp)
   0x000000000040075b <+75>:    add    $0x20,%rsp
   0x000000000040075f <+79>:    pop    %rbp
   0x0000000000400760 <+80>:    retq
End of assembler dump.
```

查看寄存器状态：

```
(gdb) i r
rax            0x1      1
rbx            0x0      0
rcx            0x0      0
rdx            0xa      10
rsi            0x0      0
rdi            0x400860 4196448
rbp            0x7ffed098aea0   0x7ffed098aea0
rsp            0x7ffed098ae80   0x7ffed098ae80
r8             0x7f9c88bbf060   140310285643872
r9             0x7ffed098c19f   140732398092703
r10            0x1      1
r11            0x0      0
r12            0x40061c 4195868
r13            0x7ffed098afb0   140732398088112
r14            0x0      0
r15            0x0      0
rip            0x400749 0x400749 <Func(char const*)+57>
eflags         0x10206  [ PF IF RF ]
cs             0x33     51
ss             0x2b     43
ds             0x0      0
es             0x0      0
fs             0x0      0
gs             0x0      0
```

## 5. 总结

本文主要通过一个示例程序演示了在 Linux 环境下 GDB 的基本使用方法，整理了 GDB 的常用指令，以及调试 C/C++ 程序和 corefile 的步骤。在实际应用中 GDB 能个极大程度地提高开发和调试效率，更多的使用技巧还需要结合实践来练习。