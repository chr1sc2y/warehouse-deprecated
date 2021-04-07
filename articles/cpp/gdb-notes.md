# GDB 笔记

## 1 准备

### 1.1 生成调试信息

为了使得程序能够被调试，需要在编译的时候加上两个关键选项：

- `-g` 生成调试信息
- `-O0` 关闭优化

调试完成之后，可以使用 `strip` 命令移除程序中的调试信息。

### 1.2 启动 GDB

GDB 的启动方式有三种：

1. `gdb binary` 直接启动程序
2. `gdb -p pid` 将某个进程暂停并进行调试，也可以先直接使用 `gdb` 启动，再使用 `attach pid` 命令将进程依附；调试完成后可以使用 `detach` 命令将进程脱离
3. `gdb binary corefile` 调试程序发生 core dump 时生成的 core file，为了生成这个文件需要进行两项设置：
   - 在 `/etc/profile` 文件中添加一行 `ulimit -c unlimited`
   - 在 `/proc/sys/kernel/core_patter` 文件中设置 core file 的存储目录和文件名

## 2 调试命令



