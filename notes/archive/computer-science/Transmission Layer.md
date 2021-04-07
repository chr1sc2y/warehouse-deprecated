---
title: "计算机网络 传输层"
date: 2015-07-05T08:57:52+10:00
draft: true
categories: ["Computer Science"]
---

# 计算机网络

## 传输层

### TCP Transmission Control Protocol

1. 特点
    - 面向连接
    - 点对点的全双工通信
    - 利用超时重传的机制来实现可靠传输
    - 面向字节流，将应用层传输下来的报文当作字节流，组织成不同大小的数据块
    - 提供流量控制，拥塞控制等功能

2. 三次握手
    ![三次握手](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/三次握手.png)
    - 流程
        1. server 处于 listen 状态，等待 client 的请求
        2. client 向 server 发送请求连接报文（SYN = 1，seq = x），进入 SYN_SENT 状态
        3. server 收到请求连接报文，向 client 发送确认连接报文（SYN = 1， ACK = 1），进入 SYN_RECV 状态
        4. client 收到确认报文，再次向 server 发送确认报文，进入 ESTABLISHED 状态
        5. server 收到确认报文，进入 ESTABLISHED 状态
    - 目的：防止失效的连接请求到达服务器
    - 客户端发送的连接请求如果在网络中滞留并等待一个超时重传时间之后，就会重新发送连接请求，最终服务器会收到并打开两个连接请求；如果进行了三次握手，客户端会忽略服务器发送的对滞留的连接请求的确认，并不进行第三次握手，也就不会打开重复的连接请求

3. 四次挥手
    ![四次挥手](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/四次挥手.jpg)
    - 流程
        1. 主动关闭方发送释放连接报文（FIN = 1，不再发送数据），进入 FIN_WAIT-1 状态
        2. 被动关闭方收到释放连接报文，进入 CLOSE-WAIT 状态，发送确认报文（ACK = 1），主动关闭方收到确认报文后进入 FIN_WAIT-2 状态
        3. 当不再需要连接时，被动关闭方发送释放连接报文（FIN = 1，不再发送数据），进入 LAST_ACK 状态
        4. 主动关闭方收到释放连接报文，进入 TIME-WAIT 状态，发送确认报文（ACK = 1），等待 2 个最大报文存活时间（MSL）后关闭连接
        5. 被动关闭方收到确认报文，关闭连接
    - CLOSE-WAIT 状态的作用：让服务器端发送还未传输完成的数据
    - TIME-WAIT 状态的作用
        1. 确认最后一个确认报文能够到达被动关闭方，如果被动关闭方没有收到确认报文，则会重新发送释放连接报文，TIME-WAIT 状态能够保证主动关闭方能够处理这种情况
        2. 让本连接持续时间内所产生的所有报文都从网络中消失，使得下一个新的连接建立后不会出现旧的报文

4. 滑动窗口

    - 窗口是缓存的一部分，用来暂时存放字节流；发送方和接收方各有一个窗口，发送方依靠这个值设置自己的窗口大小，接收方通过报文段中的窗口字段来获取发送方窗口的大小

    - 发送窗口内的字节都允许被发送，接收窗口内的字节都允许被接收；如果发送窗口左部的字节已经发送并且收到了确认，那么发送窗口会向右滑动直到左边第一个字节不是已发送的部分；接收窗口左部字节已经发送确认并交付主机，就向右滑动接收窗口

    - 接收窗口只会对窗口内最后一个按序到达的字节进行确认，例如接收窗口已经收到的字节为 {31, 34, 35}，其中 {31} 按序到达，而 {34, 35} 就不是，因此只对字节 31 进行确认。发送方得到一个字节的确认之后，就知道这个字节之前的所有字节都已经被接收

5. 流量控制

    - 通过控制发送方的发送速率，保证接收方来得及接收
    - 接收方发送的确认报文中的窗口字段可以用来控制发送方窗口大小，从而调整发送方的发送速率；若将窗口字段设置为 0，那么发送方将不能发送数据

6. 拥塞控制

    - 控制发送方的发送速率，降低网络的拥塞程度
    - 如果网络出现拥塞，数据将会丢失，此时发送方如果继续重传，会导致网络拥塞程度更高，因此当出现拥塞时，应该控制发送方的速率
    - 主要通过四个方法来进行拥塞控制：慢开始、拥塞避免、快重传、快恢复
    - 发送方需要维护一个叫做拥塞窗口（cwnd）的状态变量；拥塞窗口是一个状态变量，实际决定发送方能发送多少数据的仍然是发送方窗口
    - 假设：接收方有足够大的接收缓存，因此不会发生流量控制；虽然 TCP 的窗口基于字节，但是这里设窗口的大小单位为报文段

    ![cwnd](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/cwnd.png)

    1. 慢开始与拥塞避免
        - 慢开始：发送的起始阶段，发送方只发送 1 个报文段，cwnd = 1；收到确认后每次都将 cwnd 加倍，使发送方能够发送的报文段数量为2，4，8...
        - 发送方发送速度增长很快，造成网络拥塞的可能性也就更高。设置一个慢开始门限 ssthresh 进行拥塞避免，当 cwnd >= ssthresh 时，每次只将 cwnd 加 1
        - 如果出现超时，那么令 ssthresh = cwnd / 2，并重新执行慢开始

    2. 快重传与快恢复
        - 快重传：接收方每次收到报文段后都应该对最后一个已收到的有序报文段进行确认，例如已经接收到 1，2，4，5，那么此时应当发送对 2 的确认；发送方如果收到三个重复确认就会知道丢失的报文段，执行快重传，立即重传丢失的报文段，例如收到三个 2，则 3 丢失，立即重传 3
        - 快恢复：这种情况下，只是丢失了个别报文段，而没有发生网络拥塞，因此执行快恢复，令 ssthresh = cwnd / 2 ，cwnd = ssthresh，此时直接进入拥塞避免
        - 慢开始和快恢复的快慢指的是 cwnd 的设定值，而不是 cwnd 的增长速率；慢开始 cwnd 设定为 1，而快恢复 cwnd 设定为 ssthresh

7. 黏包

- 原因：TCP 基于字节流，其传输的数据没有边界，所以可能会出现两个不同的数据包黏在一起的情况
- 解决
    1. 发送定长数据包；如果每个消息的大小都是一样的，那么在接收对等方只要累计接收数据，直到数据等于一个定长的数值就将它作为一个消息
    2. 包头加上包体长度；包头是定长的 4 个字节，说明了包体的长度。接收对等方先接收包头长度，依据包头长度来接收包体
    3. 在数据包之间设置边界，如添加特殊符号 \r\n 标记；FTP 协议正是这么做的；但问题在于如果数据正文中也含有 \r\n，则会误判为消息的边界
    4. 使用更加复杂的应用层协议

### UDP User Datagram Protocol

1. 特点
    - 无连接的
    - 支持一对一，一对多，多对一，多对多的交互
    - 面向报文，仅在应用层传输下来的报文添加 UDP 首部
    - 不保证交付

### Linux 的 5 种 I/O 模型 (Socket)

- 在 Linux下，网络 I/O 要经过两个阶段：
    1. 等待数据报到达
    2. 将数据从内核缓冲区拷贝到用户进程
- 根据这两个阶段的不同形式，分为了不同的 I/O 模型

#### 1. 阻塞 I/O 模型

![Blocking I/O](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/Blocking%20IO.png)

- 通常 IO 操作都是阻塞 I/O 的，也就是当调用 recvfrom 时，如果没有数据收到，那么线程或进程就会被挂起，直到收到数据
- 当连接非常多时，每个线程的时间非常短，而且线程切换非常频繁，线程的存放是有内存开销的，上下文切换是有 CPU 开销的
- 用户进程调用 recvfrom，被阻塞；内核态等待数据，然后拷贝数据到内核缓冲区，返回结果；用户进程解除阻塞状态
- 阻塞并不意味着整个操作系统都被阻塞，阻塞的过程中，其它用户进程仍然可以执行
- 其他用户进程仍然可以执行，所以不消耗 CPU 时间，这种模型的 CPU 利用率较高
- 特点：IO 执行的两个阶段都会被 block

#### 2. 非阻塞 I/O 模型

![Nonblocking I/O](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/Nonblocking%20IO.png)

- 用户态不断执行系统调用来轮询（polling） I/O 是否完成，如果没有则返回 EWOULDBLOCK，不会阻塞线程
- 如果内核态中的数据还没有准备好，那么立刻返回 error，且不会阻塞用户态；否则阻塞用户态，拷贝数据，并返回结果
- 需要处理更多的系统调用，因此 CPU 利用率比较低

#### 3. I/O 多路复用

![I/O Multiplexing](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/IO%20Multiplexing.png)

- 使用一个线程来检查多个文件描述符（Socket）的状态，比如 select/poll/epoll，直到多个 Socket 中的任意一个变为可读时返回 readable，再调用 recvfrom 把数据从内核复制到进程中
- 让单个线程具有处理多个 I/O 事件的能力，又称为事件驱动（Event Driven I/O）
- 如果一个 Web 服务器没有使用 I/O 多路复用模型，那么每一个 Socket 连接都需要创建一个线程来处理，如果同时有几万个连接，那么就需要创建相同数量的线程
- 相较于多进程和多线程，I/O 多路复用不需要进行进程和线程的创建和切换，系统开销更小

#### 4. 信号驱动 I/O

![Signal-Driven IO.png](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/Signal-Driven%20IO.png)

- 用户进程执行系统调用 sigaction 并立即返回，等待数据阶段是非阻塞的
- 内核态在数据到达后向用户进程发送 SIGIO 信号，用户进程收到之后执行系统调用 recvfrom 并拷贝数据
- 相比于非阻塞 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高

#### 5. 异步 I/O

![Asynchronous IO](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/Asynchronous%20IO.png)

- 用户进程执行系统调用 aio_read 后立即返回，不会被阻塞
- 内核态在数据拷贝完成后向用户进程发送信号
- 异步 I/O 的信号是通知用户进程 I/O 已经完成；而信号驱动 I/O 的信号是通知用户进程可以开始 I/O

#### 对比

- 同步 I/O：将数据从内核缓冲区拷贝到用户进程的（第二阶段）时，用户进程会阻塞；包括阻塞 I/O 模型，非阻塞 I/O 模型，I/O 复用和信号驱动 I/O。
- 异步 I/O：将数据从内核缓冲区拷贝到用户进程的（第二阶段）时，用户进程不会阻塞。

![Asynchronous IO](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/IO%20Compare.png)

### I/O 多路复用的机制

#### select

```c++
int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

- select 允许进程监视一组文件描述符，等待一个或者多个描述符成为就绪状态，完成 I/O 操作
- fd_set 使用数组实现，数组大小使用 FD_SETSIZE 定义，所以只能监听少于 FD_SETSIZE 数量的描述符。有三种类型的描述符类型：readset、writeset、exceptset，分别对应读、写、异常条件的描述符集合
- timeout 为超时参数，调用 select 会一直阻塞直到有描述符的事件到达或者等待的时间超过 timeout
- 成功调用返回结果大于 0，出错返回结果为 -1，超时返回结果为 0

#### poll

```c++
int poll(struct pollfd *fds, unsigned int nfds, int timeout);

struct pollfd {
               int   fd;         /* file descriptor */
               short events;     /* requested events */
               short revents;    /* returned events */
           };
```

- poll 的功能与 select 类似
- poll 中的描述符是 pollfd 类型的数组

#### select 和 poll 对比

1. 可移植性

    - 几乎所有的系统都支持 select，只有比较新的系统支持 poll

2. 实现

    - select 会修改文件描述符，poll 不会
    - select 的文件描述符类型使用数组实现，默认只能监听 1024 个描述符（FD_SETSIZE = 1024）；poll 没有文件描述符数量的限制
    - poll 提供了更多的事件类型，并且对描述符的重复利用上比 select 高
    - 如果一个线程对某个文件描述符调用了 select 或者 poll，另一个线程关闭了该描述符，会导致调用结果不确定

3. 速度

    - select 和 poll 速度都比较慢，每次调用都需要将全部文件描述符从用户进程拷贝到内核缓冲区

#### epoll

```c++
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

- epoll_ctl() 用于向内核注册新的文件描述符或者是改变某个文件文件描述符的状态；已注册的文件描述符在内核中会被维护在一棵红黑树上，通过回调函数内核会将 I/O 准备好的文件描述符加入到一个链表中进行管理
- 进程调用 epoll_wait() 便可以得到事件完成的文件描述符
- epoll 只需要将文件描述符从用户进程向内核缓冲区拷贝一次，并且进程不需要通过轮询来获得事件完成的描述符
- epoll 仅适用于 Linux
- epoll 比 select 和 poll 更加灵活而且没有文件描述符数量限制
- 当一个线程调用了 epoll_wait() 而另一个线程关闭了同一个文件描述符时，不会产生类似于 select 和 poll 的不确定情况
- epoll 的文件描述符事件有两种触发模式：水平触发（LT，level trigger）和 边缘触发（ET，edge trigger）
    - 水平触发
        - 只要读缓冲区不为空，或写缓冲区不为满，epoll_wait就会一直触发事件
        - 当 epoll_wait() 检测到文件描述符事件到达时，会通知进程，但进程可以不立即处理该事件，下次调用 epoll_wait() 时会再次通知进程
        - LT模式是默认的模式，并且同时支持 blocking 和 non-blocking
    - 边缘触发
        - 当读缓冲区从空变为不空，或写缓冲区从满变为不满时，事件触发一次；只有缓冲区状态变化那一刻会有事件
        - 当 epoll_wait() 检测到文件描述符事件到达时，会通知进程，进程必须立即处理事件，下次再调用 epoll_wait() 时不会再得到事件到达的通知
        - 很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 non-Blocking，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死

#### 应用场景

1. select

    - select 的 timeout 参数精度为 1ns，而 poll 和 epoll 为 1ms，所以 select 更适用于对实时性要求比较高的场景，比如核反应堆的控制
    - select 可移植性更好，几乎被所有主流平台所支持

2. poll

    - poll 没有最大文件描述符数量的限制

3. epoll

    - 当只需要运行在 Linux 平台上，且有大量的文件描述符需要同时轮询（这些连接最好都是长连接）时，可以使用 epoll；如果同时监控小于 1000 个文件描述符，就没有必要使用 epoll
    - 因为 epoll 中的所有文件描述符都存储在内核中，每次修改文件描述符的状态都需要执行系统调用 epoll_ctl()，频繁进行系统调用会降低效率，且内核中的文件描述符不容易调试；因此需要监控的文件描述符状态变化多且短暂时，不应该使用 epoll

### Socket 编程

#### Socket 类型

1. SOCK_STREAM 流式套接字
    - TCP
2. SOCK_DGRAM 数据报套接字
3. SOCK_RAW 原始套接字

#### Socket 通信过程

- 服务器：socket() -> bind() -> listen() -> accept() -> read() 阻塞 -> read()/write() -> read() -> close()
- 客户端：socket() -> connect() -> read()/write() -> close()

1. int socket(int domain, int type, int protocol);
    - domain：通信协议域，有本地，ipv4 或 ipv6
    - type：Socket 类型，有 SOCK_STREAM，SOCK_DGRAM，SOCK_RAW
    - protocol：传输层协议

2. int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    - 设置 Socket 通信的地址，如果是 INADDR_ANY 则表示监听本机所有的 interface，如果是 127.0.0.1 则表示监听本地的 process 通信
    - sockfd：之前socket()获得的file handle
   addr：绑定地址，有 IP 地址或本地文件路径
   addrlen：地址长度

3. int listen(int sockfd, int backlog);
    - 使服务器端开始监听端口
    - 只用于 server，设置接收 queue 的长度；如果 queue 满，可以丢弃新到的connection或者回复客户端ECONNREFUSED
    - sockfd：之前socket()获得的file handle
    - backlog：设置server可以同时接收的最大链接数，server端会有个处理connection的queue，listen设置这个queue的长度。

4. int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    - 从 queue 中获取第一个 pending 的 connection，新建一个 socket 并返回
    - addr：对端地址
    - addrlen：地址长度

5. int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    - 建立双方连接；对于 TCP，connect 发起三次握手，连接在成功返回后建立；对于 UDP，connect 用来指定要通信的对端地址，也可以在 sendto 中指定对端地址
    - sockfd：socket的标示filehandle
    - addr：server端地址
    - addrlen：地址长度
