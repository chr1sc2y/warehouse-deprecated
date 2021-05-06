# 通过剥离符号表提升 C++ 程序的编译速度

[TOC]

## 基础

### C++ 程序的构建过程

构建 C++ 程序并不是一个一蹴而就的过程，无论是 GCC, Clang 还是 MSVC 都只不过是把 [C++ 程序的构建过程](https://people.cs.pitt.edu/~xianeizhang/notes/Linking.html)打包自动化了而已；C++ 程序的构建分为四个步骤：

1. [预处理器](https://en.wikipedia.org/wiki/Preprocessor)将被包含的头文件内容拷贝到源文件中（[任何时候都不应该 include 源文件](https://stackoverflow.com/questions/1686204/why-should-i-not-include-cpp-files-and-instead-use-a-header)），并将由 `#define` 定义的符号常量（symbolic constants）替换掉；

2. [编译器](https://en.wikipedia.org/wiki/Compiler)通过[词法分析（lexical analysis）](https://en.wikipedia.org/wiki/Lexical_analysis)、[语法分析（syntactic analysis）](https://en.wikipedia.org/wiki/Parsing)、[语义分析（semantic analysis）](https://en.wikipedia.org/wiki/Semantic_analysis_(compilers))、中间代码生成和优化、目标代码生成和优化等步骤，将预处理后的源文件（expanded source code file）翻译为对应平台的汇编代码（assembler code）；其中，在**词法分析**阶段，编译器会将所有的**标识符/符号 （identifier / symbol）**添加到**符号表**中；在接下来的**语法分析**和**语义分析**阶段，编译器会更新符号表，将符号的**类型**和**作用域**与其关联；在**生成中间代码**时，编译器会通过符号的类型来确定使用哪些指令来进行寄存器分配；最后在**代码生成**阶段，将符号的内存地址添加到符号表中；即使代码中出现了只有声明而没有定义的符号，编译器也是可以正常生成汇编文件的，这些未定义的符号会在链接的时候被替换为对应的地址；

3. [汇编器](https://en.wikipedia.org/wiki/GNU_Assembler)将汇编代码逐行转换为[机器码/字节码（bytecode）](https://en.wikipedia.org/wiki/Bytecode)，并生成 **[目标文件（object file）](https://refspecs.linuxbase.org/elf/gabi4+/ch4.intro.html)**<!-- ，也叫做 elf (Executable and Linking Format) 文件 -->；目标文件有 **可重定位目标文件（relocatable object file）**，**可执行目标文件（executable object file）** 和 **共享目标文件（shared object files）**三种，由汇编器生成的包含机器码的文件是可重定位目标文件，或共享目标文件；汇编器会列出所有未被解析符号的列表；通常**可重定位目标文件**包含了程序指令、数据、它们对应的绝对地址、以及未被解析的符号的符号表等；在这个阶段中生成的文件可以作为静态链接库文件被保存下来；

4. [链接器](https://en.wikipedia.org/wiki/Linker_(computing))通过[符号解析（symbol resolution）](https://docs.oracle.com/cd/E19120-01/open.solaris/819-0690/chapter2-93321/index.html)和[重定位（relocation）](https://docs.oracle.com/cd/E19683-01/817-3677/chapter3-2/index.html)，将**可重定位目标文件**中未被解析的符号替换为其对应的地址，生成**可执行目标文件**或是**动态链接库**。

以一个简单的 helloworld 程序为例，利用 GCC 的编译选项可以获得上述任意一个步骤生成的文件：

```cpp
#include <iostream>

using namespace std;

int main()
{
    cout << "hello world!" << endl;
    return 0;
}
```

在构建时加上 `-E` 选项可以在预处理阶段后中断；预处理后生成的文件一般会非常大，因为它会直接将包含的头文件展开：

```shell
$ gcc -E helloworld.cpp > helloworld.i
$ ll
total 416K
4.0K -rw-rw-r-- 1 joelzychen joelzychen  107 Apr 22 20:28 helloworld.cpp
412K -rw-rw-r-- 1 joelzychen joelzychen 411K Apr 22 20:30 helloworld.i
```

在构建时加上 `-S` 选项可以在编译阶段后中断，生成的汇编文件会以 `.s` 作为扩展名：

```shell
$ gcc -S helloworld.cpp
$ head helloworld.s
        .file   "helloworld.cpp"
        .local  _ZStL8__ioinit
        .comm   _ZStL8__ioinit,1,1
        .section        .rodata
.LC0:
        .string "hello world!"
        .text
        .globl  main
        .type   main, @function
main:
```

在构建时加上 `-c` 选项可以在汇编阶段后中断，生成的目标文件会以 `.o` 作为扩展名：

```shell
$ gcc -c helloworld.cpp 
$ ll
total 424K
412K -rw-rw-r-- 1 joelzychen joelzychen 411K Apr 22 20:31 helloworld.temp
4.0K -rw-rw-r-- 1 joelzychen joelzychen  107 Apr 22 20:31 helloworld.cpp
4.0K -rw-rw-r-- 1 joelzychen joelzychen 1.9K Apr 22 20:33 helloworld.s
4.0K -rw-rw-r-- 1 joelzychen joelzychen 2.7K Apr 22 20:37 helloworld.o
```

### ELF

`readelf` 命令可以用来展示 `elf` 文件


### 符号表

在编译和汇编过程结束之后可以得到 relocatable object file，



会生成一个[符号表](https://en.wikipedia.org/wiki/Symbol_table)，它会将函数名或变量名等标识符/符号 (identifier / symbol) 与其相关信息，包括地址、类型、作用域、占用空间等关联在一起；链接器在链接时会根据符号表来对标识符进行寻址。























































