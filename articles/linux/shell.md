# shell notes

`#!` 使用指定文件执行

`#!/bin/sh `无论使用的是什么shell，都使用 /bin/sh 执行

`#` 注释 comment

`*` 通配符 wildcards

`\` (backslash) 转义符 escape character

## variable

### echo

`ehco hello        world` echo 接受两个参数，中间用于分隔的空格被当作一个空格对待

`echo "wsaerg  				09jw4e"` 接受一个字符串作为参数

`echo "Hello   "World""` 只接受了一个参数

`msg="hello world"` 变量赋值的等号前后不能有空格

`echo $msg` 使用 $ 来指明变量

`read TEST` 读取 TEST 变量

`${TEST}_p` 拼接变量

### parameter

`$#` 参数个数，shell 脚本本身不占参数

`$@` 参数列表；`$*` 字符串类型的参数列表

```shell
#!/bin/sh
echo $#
echo $@
for para in $@
do
   echo $para
done
```

```shell
$ sh var.sh 1 2 3
3
1 2 3
1
2
3
```

`$?` 上一个命令的返回码

`$$`：当前 shell 的 PID；`$!` 上一个后台进程的 PID

`IFS` Internal Field Seperator 是 shell 内部预留的特殊环境变量

### export & source

```bash
$ vim ./test.sh
#!/bin/sh
echo $TTT
TTT="sss"
```

`export` 使用 shell A 运行一个 .sh 脚本时，会启动一个新的 shell B 来运行这个脚本，A 和 B 不共享变量；可以使用 `export` 来将变量设置为可**派生**的，这个变量在由当前 shell 派生出子 shell 时会被拷贝过去：

```bash
$ TTT="ttt"
$ echo $TTT
ttt
$ ./test.sh 

$ export TTT
$ ./test.sh 
ttt
```

`source` 执行脚本并**获取环境修改**，可以用 `.` 执行

```bash
$ TTT="ttt"
$ ./test.sh 
ttt
$ echo $TTT
ttt
$ source ./test.sh 
ttt
$ echo $TTT
sss
```

## 参数

`$#` 获取参数个数

`$@` 获取所有参数，保存在一个字符串列表里

`$*` 获取所有参数，保存在一个字符串里

`$0` 第 0 个参数

`$1` 第 1 个参数

## 循环

1. for

```sh
#!/bin/sh
for i in 1 2 hello world *
do
    echo $i
done
```

通配符会列出所有文件：

```shell
$ sh for.sh
sh for.sh
1
2
hello
world
for.sh
test.sh
while.sh
```

`{0,1,2}` 等效于 `for i in 0 1 2`

```bash
$ touch f{0,1,2}
```

2. while

```shell
#!/bin/sh
STR="input \"exit\" to exit"
while [ "$STR" != "exit" ]
do
    echo $STR
    read STR
done
```

## command

`type` 查看命令的类型

```shell
$ type ll
ll is aliased to `ls -lrtsh --color=auto'
$ type for
for is a shell keyword
$ type [
[ is a shell builtin
$ type date
date is /usr/bin/date
```

`which` 查看文件路径

```shell
$ which [
/usr/bin/[
```

## test

`[` 是一个文件，执行时必须用空格与其他命令分隔开，等同于 `test`：

```shell
read foo

if test $foo -eq 1; then
    echo 1
fi

if [ $foo = 2 ]
then
    echo 2
fi
```

`if` 和 `elif` 后需接 `then` ，且需要写在不同的行，或者用 `;` 分隔开；而 `else` 后不用：

```shell
read foo

if [ $foo = 1 ]
then
        echo 1
elif test $foo -eq 2; then
        echo 2
else
        echo 3
fi
```

