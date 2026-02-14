---
title: 网络编程
copyright: true
date: 2023-09-18 22:56:09
updated: 2023-09-18 22:56:09
tags:
    - 计算机网络
    - 课程笔记
categories:
    - 计算机网络
    - 网络编程
#excerpt:
#index_img:
#banner_img:
---
# 网络通信

## 框架

![1695052575889](/img/internet-programming/1695052575889.png)

![1695052787608](/img/internet-programming/1695052787608.png)

## 套接字

![](/img/internet-programming/1695049787130.png)

## 函数说明

### WSAStartup & WSACleanup

在程序创建第一个套接字之前使用 `WSAStartup()`初始化 Windows 套接字的动态连接库

程序结束后执行 `WSACleanup`清除

```c
WINSOCK_API_LINKAGE int WSAAPI WSAStartup(WORD wVersionRequested,LPWSADATA lpWSAData);
WINSOCK_API_LINKAGE int WSAAPI WSACleanup(void);
```

其中 `WSAStartup()`第一个参数传入 `MAKEWORD(2, 0)`, 这是一个宏定义. 第二个参数传入一个 `WSADATA`类型数据的地址, 用于存储 `WSAStartup()`返回的一些信息

### socket

创建套接字

1. 协议族, socket中只能是 `AF_INET`(这是说明IP地址为IPv4类型)
2. 协议类型, `SOCK_STREAM`代表使用TCP, `SOCK_DGRAM`代表UDP
3. 具体协议, 0会自动选择协议

### connect

客户端向服务器请求连接, 返回0代表连接成功

1. socket函数返回的值
2. 指明目标服务器的协议族, IP, 端口
3. 第二个参数的大小, 使用 `sizeof`即可

### send

用于向TCP的另一端发送数据, 返回发送的字节数

1. 发送端的套接字描述符, 由 `socket`函数创建
2. 发送数据的内存地址(可以是基本数据类型, 数组, 结构体, 字符串)
3. 指明要发送的字节数
4. 设置为0

### recv

用于接收另一端socket发送过来的数据, 如果没有收到会一直等待, 收到则返回收到的字符数. 返回-1代表出错, 0代表socket关闭.

1. 接收端的套接字描述符, 由 `socket`函数创建返回
2. 接受数据的内存地址(可以是基本数据类型, 数组, 结构体, 字符串)
3. 需要接受的字节数(不能超过第二个参数指定的缓冲区大小)
4. 设置为0

### close

关闭套接字, 并终止TCP连接, 成功返回0, 失败返回-1.

### bind

服务端把用于通信的地址和端口绑定到socket上，当bind函数返回0时，为正确绑定，返回-1，则为绑定失败.

1. 需要绑定的socket, 由 `socket`函数创建
2. 服务端的通信地址(IP)与端口
3. 第二个参数的大小(使用 `sizeof`即可)

### listen

当有较多的client发起connect时，server端不能及时的处理已经建立的连接，这时就会将connect连接放在等待队列中缓存起来. 这个等待队列的长度有listen中的backlog参数来设定. 当listen运行成功时，返回0；运行失败时，返回值为-1.

`listen`函数使进程从指定的套接字开始接收连接消息, 类似一个开关

1. `socket`函数创建的套接字, 从这个套接字接收连接消息
2. server可以缓存的连接最大个数, 即等待队列的长度

### accept

accept函数等待客户端的连接，如果没有客户端连上来，它就一直等待. accept等待到客户端的连接后，创建一个新的socket，函数返回值就是这个新的socket，服务端用这个新的socket和客户端进行报文的收发.

1. 被 `listen`过的socket
2. 存放客户端的地址信息(协议族, IP, 端口)
3. 存放第二个参数的大小

### recvfrom

UDP中用于接收数据的方法

`int recvfrom(int s, void *buf, int len, unsigned int flags, struct sockaddr *from,int *fromlen);`

1. 接收数据的套接字
2. 接收数据的缓冲地址
3. 缓冲区大小(使用sizeof)
4. 设置为0
5. 用于存储发送数据的网络地址
6. 第五个参数的大小(使用sizeof)

### sendto

UDP用于发送数据的方法

`int sendto(int s, const void * msg, int len, unsigned int flags, const struct sockaddr * to, int tolen);`

1. 发送数据的套接字
2. 发送的数据地址
3. 数据的长度
4. 设置为0
5. 指定目的网络的地址
6. 第5个参数的大小(使用sizeof)

## TCP

### 基本流程
