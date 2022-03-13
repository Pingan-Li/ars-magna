# Socket 简介
## 概述
Socket（套接字）是网络编程的一个抽象模型，服务器和用户端之间的网络通信通过Socket模型进行。不过，这里要区分socket和socket discriptor（套接字描述符，简写为sockfd）。所谓的socket是操作系统内核在管理，而sockfd是操作系统对外发布的一个资源句柄( resource handler)。在使用系统调用时，我们操作的实际上是sockfd和socket address。为了更好地理解socket，可以将文件作为类比：

| Socket            | File            | 说明                                       |
| ----------------- | --------------- | ------------------------------------------ |
| socket            | file            | 由内核管理的一种资源，也是一种IO的抽象模型 |
| socket address    | filepath        | 调用者向内核表明IO操作的对象               |
| socket discriptor | file discriptor | 内核向调用者提供的IO资源句柄               |

以上三个概念非常容易混淆，socket != socket_address != sockfd。三者之间共同写协作才能完成网络IO。

## API
### 1. socket
通过调用socket来创建socket。
```C++
int socket(int domain, int type, int protocol);
```
返回值：
（1）-1， 表明错误产生。
（2）其他，返回sockfd。

返回值的情况是比较清晰的，但是socket函数的参数却复杂的多，虽然三个参数的值类型都是int，但是其含义却有所不同。
| domain   | 说明       |
| -------- | ---------- |
| AF_INET  | IPv4 协议  |
| AF_INET6 | IPv6 协议  |
| AF_LOCAL | Unix域协议 |
| AF_ROUTE | 路由套接字 |
| AF_KEY   | 密钥套接字 |

---

| type           | 说明           |
| -------------- | -------------- |
| SOCK_STREAM    | 字节流套接字   |
| SOCK_DGRAM     | 数据包套接字   |
| SOCK_SEQPACKET | 有序分组套接字 |
| SOCK_RAW       | 原始套接字     |

---
| protocol     | 说明     |
| ------------ | -------- |
| IPPROTO_TCP  | TCP协议  |
| IPPROTO_UDP  | UDP协议  |
| IPPROTO_SCTP | SCTP协议 |


AF_前缀表示协议族，PF_前缀表示地址族。建议推荐使用AF_开头的值。

如果socket创建成功，那么就会返回一个int类型，它被称作是socket discriptor(套接字描述符)，简写为sockfd。在这里，我们可以发现socket和file的相似性。

### 2. connect
TCP的客户端调用connect函数来发起与服务器的连接
```C++
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr * servaddr, socklen_t addrlen);
```
返回值:（1）0，表示成功。(2) -1 表示失败，connect的返回值单纯表示结果成功与否，并没有其他的含义。

从connect函数的参数来看，第一个参数就是sockfd，这不难理解，sockfd是内核向外提供的资源句柄。
第二个参数是传入一个sockaddr的地址，servaddr。用const修饰意味着这个参数是输入参数，这个参数的作用是告诉服务器的socket address。这里需要注意的是sockaddr是一个通用的类型，因此在传入时要强制转型。第三个参数就是具体socket类型的长度。

客户端在调用connect函数前不一定需要调用bind函数，因为客户端知道自己的ip地址，而且可以随机选择一个端口。

调用connect可以触发TCP的三路握手，而且仅仅在连接建立成功或者失败后才会返回，出错可能是一下的情况：

（1）如果TCP客户端没有接受到SYN分节的响应，则返回ETIMEDOUT错误。

（2）若对客户的SYN的响应是RST(表示复位)，则表明该服务器主机在我们指定的端口上没有京城在等待与之连接，这是一种硬错误，客户一接到RST就马上返回ECONNREFUSED错误。

（3）若客户端发出的SYN在中间的某个路由器上引发了一个“destination unreachable”ICMP错误，则认为是一种软错误(soft error)，会在一定的时间后(BSD 75s)才返回。
### 3. bind
```C++
int bind(int sockfd, struct sockaddr* myaddr, socklen_t addrlen);
```
返回值:若成功，返回0，失败，返回-1。

bind会将地址(address)和端口(port)绑定到一个套接字(socket)上，不过地址和端口可以交由内核指定，于是有以下四种情况：
| Address  | Port | 效果                       |
| -------- | ---- | -------------------------- |
| 通配地址 | 0    | 内核选择地址和端口         |
| 通配地址 | 非0  | 内核选择地址，进程决定端口 |
| 本地地址 | 0    | 进程决定地址，内核选择端口 |
| 本地地址 | 非0  | 进程决定地址和端口         |
表中通配地址是INADDR_ANY，通常用0代表。

### 4. listen
```C++
#include <sys/socket.h>
int listen(int sockfd, int backlog);
```
返回值：0，表示成功；-1，表示失败。

listen 仅仅由服务器调用，它做两件事：

(1)将一个未连接的套接字转换成一个被动套接字(passive socket)。

    socket函数调用时，返回的sockfd所关联的socket是未连接主动套接字(unconnected positive socekt)。而listen导致套接字从CLOSED状态转换到LISTEN状态。

(2)设置套接字排队的最大的连接数。
    这个其实与内核有关，内核每一个监听套接字维护了两个队列：
    1. 未完成连接队列(incomplete connection queue)
        连接正在完成TCP三路握手，这些套接字处于SYN_RCVD队列。
    2. 已完成连接队列(completed coneection queue)
        已经完成TCP三路握手，这些套接字处于ESTABLISHED状态。

以上两个队列的长度不会超过backlog，但是这个并不是标准规定的。相同的backlog在不同的操作系统中，队列的长度并不相同。

backlog值不应该设置为0。

### 5. accept
```C++
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr * cliaddr, socklen_t *addrlen);
```
返回值：-1，表示出错；非负值，表示sockfd，这一点需要注意。

accpet也仅能由服务器调用，它主要做了以下的事情：

从已完成连接队列中返回下一个已经完成的连接，如果已经完成的连接时空的，那么accpet一般会阻塞。

我们看accpet的参数，它要求传入一个sockfd，这里就有问题了，因为accpet自身会返回一个sockfd。这两个sockfd之间到底有什么区别呢？实际上，我们传递给accpet的sockfd被称为监听套接字(listening socket)描述符。监听套接字是通过socket->bind->listen这套流程产生的。

而accpet返回的那个sockfd，被称为是已连接套接字(connected socket)描述符。区分这两个套接字描述符非常重要，为了方便对比，可以参考下面的表格：

| 项目     | 监听套接字描述符 | 已连接套接字描述符 |
| -------- | ---------------- | ------------------ |
| 产生     | socket()         | accept             |
| 生命周期 | 整个进程         | 调用close关闭      |
| 数量     | 一般只有1个      | 多个               |

剩下的两个参数：cliaddr和addrlen都是传入的非const指针，如果两个指针都是有效的，那么accpet会将客户端的sockaddr写入这个结构体中，如果服务器对客户端的sockaddr毫无兴趣，那么可以直接设置为NULL。
