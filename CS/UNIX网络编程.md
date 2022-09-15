# UNIX网络编程

## 传输层

### TCP

传输控制协议（Transmission Control Protocol）。TCP是一个面向连接的协议，为用户进程提供可靠的全双工字节流。
TCP套接字是一种流套接字（stream socket）。TCP关心确认、超时和重传之类的细节。大多数因特网应用程序使用TCP。
注意，TCP既可以使用IPv4，也可以使用IPv6。

#### 三路握手

1. 服务器通过调用socket、bind和listen来被动打开（passive open）
2. 客户端通过调用connect发起主动打开（active open），发送SYN（同步）分节
3. 服务器必须确认（ACK）客户端的SYN，同时发送自己的SYN分节。服务器在**单个分节**中发送SYN和对客户端SYN的确认（ACK）
4. 客户端必须确认（ACK）服务器的SYN

#### 四路挥手

1. 某个应用进程调用close，称该端主动关闭(active close)。该端的TCP发送一个FIN分节，表示数据发送完毕
2. 接收到这个FIN的对端执行被动关闭（passive close）。这个FIN由TCP确认（ACK）。它的接收也作为一个文件结束符（end-of-file）
   传递给接收端应用进程（放在已排队等候该应用进程接收的任何其他数据之后），因为FIN的接收意味着接收端应用进程在相应连接上再无额外数据可接收
3. 一段时间后，接收到这个文件结束符的应用进程将调用close关闭它的套接字，这导致它的TCP也发送一个FIN
4. 接收这个最终FIN的原发送端TCP确认（ACK）这个FIN

#### TIME_WAIT状态

- 可靠的实现TCP全双工连接的终止
- 允许老的重复分节在网络中消逝

### UDP

用户数据报协议（User Datagram Protocol）。UDP是一个无连接协议。UDP套接字是一种数据报套接字（datagram socket）。
UDP数据报不能保证最终到达他们的目的地。与TCP一样，UDP既可以使用IPv4，也可以使用IPv6。

### SCTP

流控制传输协议（Stream Control Transmission Protocol）。SCTP是一个提供可靠全双工关联的面向连接的协议，我们使用"关联"
一词来指称SCTP中的连接，因为SCTP是多宿的，从而每个关联的两端均涉及一组IP地址和一个端口号。SCTP提供消息服务，也就是维护来自应用层的记录边界。
与TCP和UDP一样，SCTP既可以使用IPv4，也可以使用IPv6，而且能够在同一个关联中同时使用它们。

## 基本套接字编程

### 基本TCP套接字编程

#### 相关函数

##### socket函数

为了执行网络I/O，一个进程必须要做的第一件事就是调用socket函数，指定期望的通信协议类型。

```
#include <sys/socket.h>

// @family 协议族
// @type 套接字类型
// @protocol 协议类型常值，设为0则取默认值
// @return 若成功则为非负描述符，若出错则为-1
int socket(int family, int type, int protocol);
```   

##### connect函数

TCP客户使用connect来建立与TCP服务器的连接。

```
#include <sys/socket.h>

// @sockfd 套接字描述符
// @*servaddr 指向套接字地址结构的指针
// @addrlen 该结构的大小
// @return 若成功则为0，若出错则为-1
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);
```

##### bind函数

bind函数把一个本地协议地址赋予一个套接字。

```
#include <sys/socket.h>

// @sockfd 套接字描述符
// @*myaddr 指向特定于协议的地址结构的指针 
// @addrlen 该结构的大小
// @return 若成功则为0，若出错则为-1
int bind(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen);
```

##### listen函数

listen函数仅由TCP服务器调用，将socket创建的主动套接字转化成被动套接字，即套接字从CLOSED状态转换为LISTEN状态。

```
#include <sys/socket.h>

// @sockfd 套接字描述符
// @backlog 内核应该为相应套接字排队的最大连接个数
// @return 若成功则为0，若出错则为-1
int listen(int sockfd, int backlog);
```

##### accept函数

accept函数仅由TCP服务器调用，用于从已完成连接队列队头返回下一个已完成连接。
如果已完成连接队列为空，那么进程被投入睡眠（假定套接字为默认的阻塞方式）。

```
#include <sys/socket.h>

// @sockfd 套接字描述符
// @*cliaddr 用来返回已连接的对端进程（客户）的协议地址
// @*addrlen 该地址结构的大小，值-结果参数
// @return 若成功则为非负描述符，若出错则为-1
int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen)
```

##### fork和exec函数

fork是Unix中派生新进程的唯一方法。

#### TCP回射程序

```
include 
```

### 基本UDP套接字

