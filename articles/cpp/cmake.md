# CMake 入门

## 0. 序

CMake 是一个跨平台的开源构建工具，使用 CMake 能够方便地管理依赖多个库的目录层次结构并生成 makefile 和使用 GNU make 来编译和连接程序。

## 1. 构建单个文件

### 1.1 使用 GCC 编译

假设现在我们希望编写一个函数来实现安全的 int 类型加法防止数据溢出，这个源文件没有任何依赖的源码或静态库：

```c++
// safe_add.cpp
#include <iostream>
#include <memory>
#define INT_MAX 2147483647
#define ERROR_DATA_OVERFLOW 2

int SafeIntAdd(std::unique_ptr<int> &sum, int a, int b)
{
    if (a > INT_MAX - b)
    {
        *sum = INT_MAX;
        return ERROR_DATA_OVERFLOW;
    }
    *sum = a + b;
    return EXIT_SUCCESS;
}

int main()
{
    int a, b;
    std::cin >> a >> b;
    std::unique_ptr<int> sum(new int(1));
    int res = SafeIntAdd(sum, a, b);
    std::cout << *sum << std::endl;
    return res;
}

```

我们可以直接使用一句简单的 gcc 命令来编译这个文件并执行：

```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ g++ main.cc -g -Wall -std=c++11 -o SafeIntAdd
[joelzychen@DevCloud ~/cmake-tutorial]$ ./SafeIntAdd 
2100000000 2100000000
2147483647
```

### 1.2 使用 cmake 构建

如果要使用 cmake 来生成 makefile 的话我们需要首先新建一个 CMakeLists.txt 文件，cmake 的所有配置都在这个文件中完成，CMakeLists.txt 中的内容大致如下：

```cmake
cmake_minimum_required(VERSION 3.10)

project(SafeIntAdd)

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
message(STATUS "CMAKE_CXX_FLAGS: " "${CMAKE_CXX_FLAGS}")
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
message(STATUS "CMAKE_CXX_FLAGS: " "${CMAKE_CXX_FLAGS}")

add_executable(SafeIntAdd main.cc)
```

其中有一些基础的 cmake 指令，它们的含义如下：

1. cmake_minimum_required：cmake 的最低版本要求
2. project：指定项目的名称
3. set：设置普通变量，缓存变量或环境变量，上面例子中的
4. add_executable：使用列出的源文件构建可执行文件

有几个需要注意的点：

1. cmake 的指令是不区分大小写的，写作 CMAKE_MINIMUM_REQUIRED 或 cmake_minimum_required，甚至是 cmAkE_mInImUm_rEquIrEd（不建议）都是可以的

2. 在使用 set 指令指定 CMAKE_CXX_FLAGS 的时候通过空格来分隔多个编译选项，生成的 CMAKE_CXX_FLAGS 字符串是 "-g;-Wall"，需要用字符串替换将分号替换为空格

3. message 可以在构建的过程中向 stdout 输出一些信息，上面例子中的输出信息为：

   ```bash
   -- CMAKE_CXX_FLAGS: -g;-Wall
   -- CMAKE_CXX_FLAGS: -g -Wall
   ```

4. 类似于 bash 脚本，在 CMakeLists.txt 中输出变量时要使用 "${CMAKE_CXX_FLAGS}" 的形式，而不能直接使用 CMAKE_CXX_FLAGS

编辑好 CMakeLists.txt 之后，我们可以新建一个 build 目录，并在 build 目录下使用 cmake 来进行构建，构建成功的话再使用 make 来进行编译和链接，最终得到 SafeAdd 这个可执行文件：

```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ mkdir build/
[joelzychen@DevCloud ~/cmake-tutorial]$ cd build/
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- CMAKE_CXX_FLAGS: -g;-Wall
-- CMAKE_CXX_FLAGS: -g -Wall
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target SafeIntAdd
[ 50%] Building CXX object CMakeFiles/SafeIntAdd.dir/main.cc.o
[100%] Linking CXX executable SafeIntAdd
[100%] Built target SafeIntAdd
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./SafeIntAdd 
2100000000 2100000000
2147483647
```

## 2. 构建多个文件

### 2.1 使用 GCC 编译

假设现在我们希望将加法函数放到单独的文件中去，并在 main 函数所在的源文件中包含这个文件：

```c++
// main.cc
#include "math.h"
#include "error_code.h"
#include <iostream>

int main()
{
    int a{ 0 }, b{ 0 }, c{ 0 };
    std::cin >> a >> b >> c;
    int sum{ 0 };
    int ret_val = SafeAdd(sum, a, b, c);
    std::cout << sum << std::endl;
    return ret_val;
}

```

```c++
// util/math.h
#ifndef UTIL_MATH_H
#define UTIL_MATH_H

#include "error_code.h"
#include <limits>

template<typename ValueType>
ValueType ValueTypeMax(ValueType)
{
    return std::numeric_limits<ValueType>::max();
}

template<typename ValueType>
int SafeAdd(ValueType &sum)
{
    return exit_success;
}

template<typename ValueType, typename ...ValueTypes>
int SafeAdd(ValueType &sum, const ValueType &value, const ValueTypes &...other_values)
{
    int ret_val = SafeAdd<ValueType>(sum, other_values...);
    if (ret_val != exit_success)
    {
        return ret_val;
    }
    if (sum > ValueTypeMax(value) - value)
    {
        sum = ValueTypeMax(value);
        return error_data_overflow;
    }
    sum += value;
    return exit_success;
}

#endif
```

```c++
// definition/error_code.h
#ifndef DEFINITION_ERROR_CODE_H
#define DEFINITION_ERROR_CODE_H

constexpr int exit_success = 0;
constexpr int exit_failure = 1;
constexpr int error_data_overflow = 2;

#endif
```

我们可以在使用 GCC 编译的时候使用 -I 参数指定头文件所在的目录：

```bash
[joelzychen@DevCloud ~/safe_add]$ g++ -g -Wall -std=c++11 -Ilib -Idefinition -o SafeAdd main.cc
[joelzychen@DevCloud ~/safe_add]$ ./SafeAdd
20000 50000 80000
150000
```

### 2.2 使用 cmake 构建

```cmake
cmake_minimum_required(VERSION 3.10)

project(SafeIntAdd)

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")

include_directories(lib/ definition/)

aux_source_directory(./ SOURCE_DIR)

add_executable(SafeIntAdd ${SOURCE_DIR})
```

相比于构建单个文件，我们额外使用了两个指令：

1. include_directories：添加多个头文件搜索路径，路径之间用空格分隔；如果将 lib 和 definition 目录都添加到到搜索路径的话，在 include 的时候就不需要使用相对路径了
2. aux_source_directory：在目录中查找所有源文件，并将这些源文件存储在变量 SOURCE_DIR 中；需要注意这个指令不会递归包含子目录

接下来进入 build 目录进行构建：

```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target SafeIntAdd
[ 50%] Building CXX object CMakeFiles/SafeIntAdd.dir/main.cc.o
[100%] Linking CXX executable SafeIntAdd
[100%] Built target SafeIntAdd
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./SafeIntAdd 
2000000000 1900000000
2147483647
```

## 3. 构建依赖于静态库的项目

关于静态库和动态库相关内容可以参考[浅谈静态库和动态库](https://zhuanlan.zhihu.com/p/71372182)。

### 3.1 使用 GCC 编译静态库文件

假设现在我们希望将 exp2 函数封装到一个计算器单例类中，并计算 2 的 n 次方，并为此编写了以下这些文件：

```c++
// main.cc
#include "util/calculator.h"
#include "definition/error_code.h"
#include <iostream>

int main()
{
    double a{ 0 };
    std::cin >> a;
    double exp2{ 0 };
    int ret_val = Calculator::GetInstance().Exp2(exp2, a);
    if (ret_val != exit_success)
    {
        return ret_val;
    }
    std::cout << exp2 << std::endl;
    return exit_success;
}

```

```c++
// util/singleton.h
#ifndef UTIL_SINGLETON_H
#define UTIL_SINGLETON_H

template <typename T>
class Singleton
{
public:
    static T& GetInstance()
    {
        static T instance;
        return instance;
    }
    Singleton(Singleton const &) = delete;
    Singleton& operator=(Singleton const &) = delete;

protected:
    Singleton() = default;
    ~Singleton() = default;

};

#endif
```

```cpp
// definition/error_code.h
#ifndef DEFINITION_ERROR_CODE_H
#define DEFINITION_ERROR_CODE_H

constexpr int exit_success = 0;
constexpr int exit_failure = 1;
constexpr int error_data_overflow = 2;

#endif
```

```c++
// util/calculator.h
#ifndef UTIL_CALCULATOR_H
#define UTIL_CALCULATOR_H

#include "singleton.h"
#include "../definition/error_code.h"
#include <limits>
#include <cmath>

class Calculator : public Singleton<Calculator>
{
public:
    template<typename ValueType>
    ValueType ValueTypeMax(ValueType)
    {
        return std::numeric_limits<ValueType>::max();
    }

    int Exp2(double &exp2, const double &val);
};

#endif
```

为了方便生成静态库文件，我们先将 calculator.cc 这个文件放到 archive 目录下：

```c++
// archive/calculator.cc
#include "../util/calculator.h"

int Calculator::Exp2(double &exp2, const double &val)
{
    if (std::sqrt(ValueTypeMax(val)) < val)
    {
        exp2 = ValueTypeMax(val);
        return error_data_overflow;
    }
    exp2 = std::exp2(val);
    return exit_success;
}

```

对于 calculator.cc 这个源文件，我们可以使用 -c 参数将其编译为 obj 文件，再使用 ar 归档就能将其编译为一个 .a 库文件了：：

```bash
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ g++ calculator.cc -g -Wall -std=c++11 -c
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ ar -crv libcalculator.a calculator.o
a - calculator.o
```

### 3.2 使用 GCC 编译项目和链接静态库

现在的目录结构如下：

```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ tree
.
|-- archive
|   |-- calculator.cc
|   |-- calculator.o
|   `-- libcalculator.a
|-- CMakeLists.txt
|-- definition
|   `-- error_code.h
|-- main.cc
`-- util
    |-- calculator.h
    `-- singleton.h

3 directories, 8 files
```

使用 GCC 编译和链接到对应的静态库文件即可得到可执行文件：

```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ g++ main.cc -g -Wall -std=c++11 -o Exp2 -Larchive -lcalculator
[joelzychen@DevCloud ~/cmake-tutorial]$ ll Exp2 
32K -rwxrwxr-x 1 joelzychen joelzychen 31K Jun 21 14:38 Exp2
[joelzychen@DevCloud ~/cmake-tutorial]$ ./Exp2
8
256
```

需要注意的点有：

1. -Llib 用于指定库文件目录
2. -lsingleton 用于指定库文件
3. -L 和 -l 参数一定要在 -o 参数之后

### 3.3 使用 cmake 构建项目和链接静态库

我们刚才已经用 GCC 编译好了静态库文件，所以可以直接在 CMakeLists.txt 里添加链接静态库的指令：

```cmake
cmake_minimum_required(VERSION 3.10)

project(Exp2)

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)

message(STATUS "source dir: " ${PROJECT_SOURCE_DIR})
message(STATUS "binary dir: " ${PROJECT_BINARY_DIR})
message(STATUS "output dir: " ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})

include_directories(./)
aux_source_directory(./ SOURCE_DIR)

link_directories(archive/)
add_executable(Exp2 ${SOURCE_DIR})
target_link_libraries(Exp2 libcalculator.a)
# target_link_libraries(Exp2 calculator)
```

和之前相比，额外使用的指令有：

1. link_directories：指定静态库或动态库的搜索路径
2. target_link_libraries：将指定的静态库连接到可执行文件上，singleton 和 libsingleton.a 两种形式等价

然后进入 build 目录进行构建：

```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ../
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- source dir: /home/joelzychen/cmake-tutorial
-- binary dir: /home/joelzychen/cmake-tutorial/build
-- output dir: /home/joelzychen/cmake-tutorial/build/bin/
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target Exp2
[ 50%] Building CXX object CMakeFiles/Exp2.dir/main.cc.o
[100%] Linking CXX executable bin/Exp2
[100%] Built target Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll bin/Exp2 
32K -rwxrwxr-x 1 joelzychen joelzychen 31K Jun 21 14:58 bin/Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./bin/Exp2 
6
64
```

可以看到通过 cmake 构建得到二进制文件大小和直接通过 GCC 编译和链接得到的二进制文件大小是相同的。这次用 `set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)` 命令将生成的二进制文件放到了 bin 目录下，注意这里的 bin 目录是使用 cmake 进行构建的目录（PROJECT_BINARY_DIR），不是 CMakeLists.txt 所在的目录（PROJECT_SOURCE_DIR）。

### 3.4 使用 cmake 构建静态库文件和项目

除了直接引用外部的静态库，cmake 还可以先将源文件编译成静态库之后在进行构建：

```cmake
cmake_minimum_required(VERSION 3.10)

set(project_name Exp2)
project(${project_name})

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)

message(STATUS "source dir: " ${PROJECT_SOURCE_DIR})
message(STATUS "binary dir: " ${PROJECT_BINARY_DIR})
message(STATUS "output dir: " "${PROJECT_BINARY_DIR}/${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

include_directories(./)
aux_source_directory(./ SOURCE_DIR)

set(static_lib_source_file archive/calculator.cc)
add_library(calculator_static STATIC ${static_lib_source_file})
add_executable(${project_name} ${SOURCE_DIR})
target_link_libraries(${project_name} calculator_static)
```

这里用到了一个新的指令 [add_library](https://cmake.org/cmake/help/latest/command/add_library.html?highlight=add_library) 来使用指定的源文件生成库文件，再使用 target_link_libraries 将生成的库文件添加到项目中；接下来进行构建：

```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ../
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- source dir: /home/joelzychen/cmake-tutorial
-- binary dir: /home/joelzychen/cmake-tutorial/build
-- output dir: /home/joelzychen/cmake-tutorial/build/bin/
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target calculator_static
[ 25%] Building CXX object CMakeFiles/calculator_static.dir/archive/calculator.cc.o
[ 50%] Linking CXX static library libcalculator_static.a
[ 50%] Built target calculator_static
Scanning dependencies of target Exp2
[ 75%] Building CXX object CMakeFiles/Exp2.dir/main.cc.o
[100%] Linking CXX executable bin/Exp2
[100%] Built target Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll | grep calculator
 12K -rw-rw-r-- 1 joelzychen joelzychen  11K Jun 21 15:30 libcalculator_static.a
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./bin/Exp2 
5
32
```

可以看到在 build 目录下生成了一个名为 libcalculator_static.a 的静态库文件，这个名字是由使用 add_library 指令时的第一个参数指定的。

## 4. 构建依赖于动态库的项目

### 4.1 使用 GCC 编译动态库文件

仍然使用之前已有的文件，在 archive 目录下生成动态库文件：

```bash
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ rm calculator.o libcalculator.a
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ g++ calculator.cc -g -Wall -std=c++11 -c -fPIC
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ g++ calculator.o -g -Wall -std=c++11 -shared -o libcalculator.so
```

分为两个步骤：

1. 使用 -c 和 -fPIC 参数生成位置无关的（position independent code）机器码 .o 文件
2. 使用 -shared 参数生成 .so 动态库文件

也可以合并为一个步骤：

```bash
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ g++ calculator.cc -g -Wall -std=c++11 -shared -fPIC -o libcalculator.so
```

这样可以直接生成动态库文件，省去生成机器码文件的中间步骤。

### 4.2 使用 GCC 编译项目和链接动态库

和链接到静态库类似，在编译后链接动态库即可生成二进制文件：

```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ g++ main.cc -g -Wall -std=c++11 -o Exp2 -Larchive -lcalculator
[joelzychen@DevCloud ~/cmake-tutorial]$ ./Exp2 
./Exp2: error while loading shared libraries: libcalculator.so: cannot open shared object file: No such file or directory
```

我们发现在运行二级制文件时会出现找不到动态库的报错，这是因为动态链接库环境变量的目录下没有找到编译时所用到的动态库，我们可以将对应的目录添加到环境变量下，或是将动态库拷贝到环境变量的目录下：

```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ echo $LD_LIBRARY_PATH

[joelzychen@DevCloud ~/cmake-tutorial]$ export LD_LIBRARY_PATH="/usr/lib/"
[joelzychen@DevCloud ~/cmake-tutorial]$ sudo cp archive/libcalculator.so /usr/lib/
```

接下来就可以运行可执行文件了，可以看到使用链接动态库方式生成的可执行文件的大小要小于使用链接静态库方式生成的可执行文件，使用 ldd 命令也能看到可执行文件是正确地调用了对应的动态库文件：

```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ ./Exp2 
9
512
[joelzychen@DevCloud ~/cmake-tutorial]$ ll Exp2 
28K -rwxrwxr-x 1 joelzychen joelzychen 28K Jun 21 14:25 Exp2
[joelzychen@DevCloud ~/cmake-tutorial]$ ldd Exp2 | grep calculator
        libcalculator.so => /usr/lib/libcalculator.so (0x00007fd2391f0000)
```

### 4.3 使用 cmake 构建项目和链接动态库

和构建静态库类似，只需要将 CMakeLists.txt 中链接静态库的指令修改为链接动态库即可进行构建：

```cmake
cmake_minimum_required(VERSION 3.10)

project(Exp2)

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)

message(STATUS "source dir: " ${PROJECT_SOURCE_DIR})
message(STATUS "binary dir: " ${PROJECT_BINARY_DIR})
message(STATUS "output dir: " "${PROJECT_BINARY_DIR}/${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

include_directories(./)
aux_source_directory(./ SOURCE_DIR)

link_directories(archive/)
add_executable(Exp2 ${SOURCE_DIR})
target_link_libraries(Exp2 libcalculator.so)
```

```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ../
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- source dir: /home/joelzychen/cmake-tutorial
-- binary dir: /home/joelzychen/cmake-tutorial/build
-- output dir: /home/joelzychen/cmake-tutorial/build/bin/
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target Exp2
[ 50%] Building CXX object CMakeFiles/Exp2.dir/main.cc.o
[100%] Linking CXX executable bin/Exp2
[100%] Built target Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll bin/Exp2 
28K -rwxrwxr-x 1 joelzychen joelzychen 28K Jun 21 15:04 bin/Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ldd bin/Exp2 | grep calculator
        libcalculator.so => /home/joelzychen/cmake-tutorial/archive/libcalculator.so (0x00007f4c87e35000)
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./bin/Exp2 
6
64
```

可以看到通过 cmake 构建得到二进制文件大小和直接通过 GCC 编译和链接动态库得到的二进制文件大小也是相同的。

### 4.4 使用 cmake 构建动态库文件和项目

使用 cmake 构建动态库的步骤和构建静态库的步骤几乎一模一样，只需要将 add_library 的 STATIC 参数改为 SHARED 即可：

```cmake
cmake_minimum_required(VERSION 3.10)

set(project_name Exp2)
project(${project_name})

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)

message(STATUS "source dir: " ${PROJECT_SOURCE_DIR})
message(STATUS "binary dir: " ${PROJECT_BINARY_DIR})
message(STATUS "output dir: " "${PROJECT_BINARY_DIR}/${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

include_directories(./)
aux_source_directory(./ SOURCE_DIR)

set(shared_lib_source_file archive/calculator.cc)
add_library(calculator_shared SHARED ${shared_lib_source_file})
add_executable(${project_name} ${SOURCE_DIR})
target_link_libraries(${project_name} calculator_shared)
```

```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- source dir: /home/joelzychen/cmake-tutorial
-- binary dir: /home/joelzychen/cmake-tutorial/build
-- output dir: /home/joelzychen/cmake-tutorial/build/bin/
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target calculator_shared
[ 25%] Building CXX object CMakeFiles/calculator_shared.dir/archive/calculator.cc.o
[ 50%] Linking CXX shared library libcalculator_shared.so
[ 50%] Built target calculator_shared
Scanning dependencies of target Exp2
[ 75%] Building CXX object CMakeFiles/Exp2.dir/main.cc.o
[100%] Linking CXX executable bin/Exp2
[100%] Built target Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll | grep calculator
 16K -rwxrwxr-x 1 joelzychen joelzychen  13K Jun 21 15:34 libcalculator_shared.so
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll bin/Exp2 
28K -rwxrwxr-x 1 joelzychen joelzychen 28K Jun 21 15:34 bin/Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ldd bin/Exp2 | grep calculator
        libcalculator_shared.so => /home/joelzychen/cmake-tutorial/build/libcalculator_shared.so (0x00007f1fa5aa5000)
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./bin/Exp2 
7
128
```

可以看到得到的二进制文件的相关信息和使用前几种方法得到的都是相同的。

## 5. 常用命令

### 5.1 项目相关

- cmake_minimum_required(VERSION 3.10)：指定 cmake 的最低版本要求
- project(project_name)：指定项目的名称
- set(CMAKE_CXX_FLAGS -g -Wall -pthread)：设置普通变量，缓存变量或环境变量

### 5.2 编译相关

- include_directories：指定头文件的目录，类似于 GCC 编译时的 `-I` 参数；等价于将头文件目录添加到环境变量 CPLUS_INCLUDE_PATH 中
- aux_source_directory：在目录中（不含子目录）查找所有源文件，并将这些源文件存储在变量 SOURCE_DIR 中
- add_executable：使用列出的源文件构建可执行文件
- add_definitions：宏定义

### 5.3 链接相关

- link_directories：指定库文件的目录，类似于 GCC 链接时的 `-L` 参数；等价于将库文件目录添加到环境变量 LD_LIBRARY_PATH 中
- target_link_libraries：将指定的静态库连接到可执行文件上，singleton 和 libsingleton.a 两种形式等价

## 6. 总结

本文通过几个示例程序对比了在 Linux 下使用 GCC 和 cmake 来编译和构建程序的基本步骤，了解了一些 cmake 的基础指令。实际开发中的项目往往会非常大，以致于不能够直接使用 GCC 来编译整个项目，在这种场景下使用 cmake 进行构建往往能够节省时间和提高效率，让我们能够专注在项目的开发上。在实际应用中编写 CMakeLists.txt 的时候还会遇到非常多的问题、不熟悉的指令和其他的使用技巧，这些都需要结合[教程](https://cmake.org/cmake/help/latest/index.html)和实践来进一步学习。