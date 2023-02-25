# Clang CLI

## 1. 打印系统头文件路径

什么是系统头文件？具体的含义在不同的操作系统上有区别，在Linux上主要是以下的几种：

1. Standard library, e.g: <iostream>
2. Third party libraries, e.g: boost
3. Posix, e.g: <pthread.h>
4. Compiler’s built-in headers, e.g: <stddef.h>

在Linux和macOS上按照如下的命令可以打印出clang的系统搜索路径

```shell
clang -v -c -xc /dev/null   # C
clang -v -c -xc++ /dev/null # C++
```

## 2. 打印系统库路径

```shell
clang --print-search-dirs
```
