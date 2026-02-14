---
title: MPI
date: 2024-03-18 17:19:24
updated: 2024-03-18 17:19:24
tags:
    - 并行
categories:
    - 并行编程
    - MPI
# excerpt: something hhh
copyright: true
# index_img: 文章在首页的封面图, 默认由 default_index_img 配置
# banner_img: img/dahe.jpg # 文章页顶部大图 
# sticky: 100 # 文章在首页排序权重, 越大越靠前
# archive: true # 首页隐藏文章, 归档页显示文章
# hide: true # 不在首页和其他归档分类页里展示, 隐藏后依然可以通过文章链接访问
---

# 编译与运行

## 基本命令

```
mpicc -g -Wall -o <executable> <source file>
```
- `mpicc`: 编译命令, 实际中使用 `gcc` 也可以通过编译
- `-g`: 产生调试信息
- `-Wall`: 打开所有 waring
- `-o`: 输出文件, 后跟输出文件名
- `<executable>`: 输出的可执行程序文件名
- `<source file>`: 源文件名

举例: `mpicc -g -Wall -o mpi_hello mpi_hello.c` 将会编译 mpi_hello.c 生成可执行文件 mpi_hello

```
mpiexec -n <number of processes> <executable>
```
- `mpiexec`: 执行命令
- `-n`: 后跟要运行的进程数
- `<number of processes>`: 运行程序使用的进程数
- `<executable>`: 运行的可执行文件名

举例: `mpiexec -n 3 mpi_hello` 将以三个进程来运行 `mpi_hello`

## 配置方式

在 Windows 下编译与运行, 我的 MPI 下载在 `D:/3rd_lib/MPI`
![1710754413641](/img/MPI/1710754413641.png)

必需的文件夹是 Include(内部是 MPI 的头文件), Bin(内部有 MPI 的执行程序, 也就是 mpiexec), Lib( MPI 的静态库文件)

下面说明两种配置方式, vscode+cmake 或者 devc++

**vscode + cmake**

在 CMakeLists.txt 中添加下面三条命令, 使得 CMake 可以找到对应的头文件和库文件

```
# MPI 路径
set(MPI_DIR "D:/3rd_lib/MPI")

# 指明所有头文件路径
include_directories(${MPI_DIR}/Include)

# 指明所有库文件路径
link_directories(${MPI_DIR}/Lib/x64)
```

生成可执行文件 mpi_hello.exe, 编译器用 gcc 即可.

```
link_libraries(msmpi)

add_executable(mpi_hello mpi_hello.c)
```

运行方式, 在 vscode 中新建一个控制台, 输入如下命令, 以 3 个进程来运行 mpi_hello

```
D:/3rd_lib/MPI/Bin/mpiexec.exe -n 3 mpi_hello
```

## 坑

```
[build] D:/3rd_lib/MPI/Include/mpi.h:245:9: error: unknown type name '__int64'
[build]   245 | typedef __int64 MPI_Aint;
[build]       |         ^~~~~~~
```

找不到 `__int64` 的定义, 在 `#include <mpi.h>` 之前 `#include <stdio.h>`. 只要保证最后引入 `mpi.h` 即可避免类似问题.
```
#include <stdio.h>
#include <mpi.h>
```

# 基本语法

## 框架

```
//... 其他头文件
#include <mpi.h>

int main(void)
{
    int  comm_sz;
    int  my_rank;

    MPI_Init(NULL, NULL);
    MPI_Comm_size(MPI_COMM_WORLD, &comm_sz);
    MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);

    if(my_rank == 0)
    {
        //...
    }
    else
    {
        //...
    }

    MPI_Finalize();
    return 0;
}
```

## 函数

### 初始化 MPI

```c
MPI_Init(
    int* argc,      // 参数的数量, main 函数第一个参数的地址
    char*** argv    // 参数, main 函数的第二个参数
);
```
参数:

- argc: 参数的数量, main 函数第一个参数的地址, 不需要可置 NULL
- argv: 参数, main 函数的第二个参数, 不需要可置 NULL

说明:

在调用 MPI 函数之前需要调用此函数, 所有 MPI 的函数都要在此之后调用

### 终止 MPI

```
MPI_Finalize();
```
说明:

该函数之后不可再调用 MPI 函数, 所有 MPI 的函数都要在此之前调用

### 通信子

```c
MPI_Comm_size(MPI_Comm comm, int* size);
```
参数:

- comm: 通信子, 一个通信子包括一组进程, 填入 `MPI_COMM_WORLD` 即可
- size: 输出值, 通信子中进程的数量, 进程编号从 0 开始

说明:

获取通信子中进程数量

```c
MPI_Comm_rank(MPI_Comm comm, int* rank);
```
参数:

- comm: 通信子, 一个通信子包括一组进程, 填入 `MPI_COMM_WORLD` 即可
- rank: 输出值, 通信子中进程的编号, 进程编号从 0 开始

说明:

获取本进程在通信子中的编号

# 点对点通信

## Send 函数

```c
MPI_Send(
    const void* buf,
    int count,
    MPI_Datatype datatype,
    int dest,
    int tag,
    MPI_Comm comm
);
```

| 类型          | 参数名    | 描述 |
|---------------|-----------|-----|
| const void*   | buf       | 待发送的数据缓冲区指针, 需要是连续的内存
| int           | count     | 发送的数据个数(不是字节数)
| MPI_Datatype  | datatype  | 发送的数据类型
| int           | dest      | 目标进程的编号
| int           | tag       | 消息的标签, 用于区分发往同一个进程的数据, 因为 MPI 不保证数据到达接收端的先后与调用 send 函数的顺序一致
| MPI_Comm      | comm      | 进程所处的通信子

注意:
该函数必需和 recv 函数成对出现, 否则会导致内存溢出等问题

示例:
向 1 号进程发送两个 3*3 大小的 double 矩阵, 为了区分两个矩阵, tag 字段分别设置为了 0 和 1
```c
MPI_Send(matB, 3*3, MPI_DOUBLE, 1, 0, MPI_COMM_WORLD);
MPI_Send(matC, 3*3, MPI_DOUBLE, 1, 1, MPI_COMM_WORLD);
```

## Recv 函数

```c
MPI_Recv(
    void* buf,
    int count,
    MPI_Datatype datatype,
    int source,
    int tag,
    MPI_Comm comm,
    MPI_Status* status
);
```

| 类型          | 参数名    | 描述 |
|---------------|-----------|-----|
| void*         | buf       | 接收数据的缓冲区指针, 需要是连续的内存
| int           | count     | 接收数据个数(不是字节数)
| MPI_Datatype  | datatype  | 接收数据类型
| int           | source    | 接收数据的发送进程编号
| int           | tag       | 消息的标签, 用于区分来自同一进程的数据
| MPI_Comm      | comm      | 进程所在的通信子
| MPI_Status*   | status    | 存储返回状态信息, status.MPI_SOURCE, status.MPI_TAG 和 status.MPI_ERROR 分别存储所收到数据发送源的进程号, 该消息的 tag 值和接收操作的错误代码. 如果不用该状态, 一般可以填入 MPI_STATUS_IGNORE

注意:
该函数是阻塞调用的, 它会一直等待期望的数据知道收到. 当两个进程互相收发消息时, 需要严格确定收发的顺序, 避免出现死锁现象(e.g. 两个进程都在等对方发送完才发送自己的数据).

示例:
从 0 号矩阵接收两个 3*3 的 double 矩阵, 将 tag 对应设置, 避免接收到错误顺序的数据(错误顺序指matC的数据先到, 所以存入matB中, matB的数据后到, 所以存到matC 中), 同时最后填入 MPI_STATUS_IGNORE 忽略错误信息.
```c
MPI_Recv(matB, 3*3, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
MPI_Recv(matC, 3*3, MPI_DOUBLE, 0, 1, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
```