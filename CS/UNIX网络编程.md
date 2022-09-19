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

## 应用层

### DNS

域名系统（Domain Name System）主要用于主机名字和IP地址之间的映射。

DNS中的条目称为资源记录（resource record，RR）。

- A A记录把一个主机名映射成一个32位的IPv4地址
- AAAA 称为"4A"（quad A）记录的AAAA记录把一个主机名映射成一个128位的IPv6地址
- PTR 称为"指针记录"（pointer record）的PTR记录把IP地址映射成主机名
- MX MX记录把一个主机指定作为给定主机的"邮件交换器"（mail exchanger）
- CNAME CNAME代表"canonical name"（规范名字），它的常见用法是为常用的服务（例如ftp和www）指派CNAME记录

#### 相关函数

##### gethostbyname函数

gethostbyname函数试图由一个主机名找到相应的二进制的IP地址。只返回IPv4地址。

```
#include <netdb.h>

// @return 若成功则为非空指针，若出错则为NULL且设置h_errno
struct hostent *gethostbyname(const char *hostname);

struct hostent {
   char  *h_name;       /* official (canonical) name of host */
   char **h_aliases;    /* pointer to array of pointers to alias names */
   int    h_addrtype;   /* host address type: AF_INET */
   int    h_length;     /* length of address: 4 */
   char **h_addr_list;  /* ptr to array of ptrs with IPv4 addrs */
}
```

##### gethostbyaddr函数

gethostbyaddr函数试图由一个二进制的IP地址找到相应的主机名。

```
#include <netdb.h>

// @return 若成功则为非空指针，若出错则为NULL且设置h_errno
struct hostent *gethostbyaddr(const char *addr, socklen_t len, int family);
```

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

```
#include <unistd.h>

// @return 在子进程中为0，在父进程中为子进程ID，若出错则为-1
pid_t fork(void);
```

存放在硬盘上的可执行程序文件能够被Unix执行的唯一方法是：由一个现有进程调用六个exec函数中的某一个。

```
#include <unistd.h>

// @return 若成功则不返回，若出错则为-1

int execl(const char *pathname, const char *arg0, .../* (char *) 0 */);

int execv(const char *pathname, char *const *argv[]);

int execle(const char *pathname, const char *arg0, .../* (char *) 0, char *const envp[] */);

int execve(const char *pathname, char *const argv[], char *const envp[]);

int execlp(const char *filename, const char *arg0, .../* (char *) 0 */);

int execvp(const char *filename, char *const argv[]);
```

##### close函数

通常的Unix close函数也用来关闭套接字，并终止TCP连接。

```
#include <unistd.h>

// @sockfd 套接字描述符
// @return 若成功则为0，若出错则为-1
int close(int sockfd);
```

##### shutdown函数

终止网络连接的通常方法是调用close函数。不过close有两个限制，可以使用shutdown函数来避免。

1. close把描述符的引用计数减1，仅在该计数变为0时才关闭套接字
2. close终止读和写两个方向的数据传送

```
#include <sys/socket.h>

// @sockfd 套接字描述符
// @howto SHUT_RD，关闭连接的读这一半；SHUT_WR，关闭连接的写这一半；SHUT_RDWR，连接的读和写都关闭
// @return 若成功则为0，若出错则为-1
int shutdown(int sockfd, int howto);
```

##### getsockname和getpeername函数

getsockname函数返回与某个套接字关联的本地协议地址，getpeername函数返回与某个套接字关联的外地协议地址。

```
#include <sys/socket.h>

// @return 若成功则为0，若出错则为-1

int getsockname(int sockfd, struct sockaddr *localaddr, socklen_t *addrlen);

int getpeername(int sockfd, struct sockaddr *peeraddr, socklen_t *addrlen);
```

##### 信号相关

信号（signal）就是告知某个进程发生了某个时间的通知，有时也称为软件中断（software interrupt）。
信号通常是异步发生的，也就是说进程预先不知道信号的准确发生时刻。

信号可以：

1. 由一个进程发给另一个进程（或自身）
2. 由内核发给进程

**signal函数**

```
/* include signal */
#include	"unp.h"

Sigfunc *
signal(int signo, Sigfunc *func)
{
	struct sigaction	act, oact;

	act.sa_handler = func;
	sigemptyset(&act.sa_mask);
	act.sa_flags = 0;
	if (signo == SIGALRM) {
#ifdef	SA_INTERRUPT
		act.sa_flags |= SA_INTERRUPT;	/* SunOS 4.x */
#endif
	} else {
#ifdef	SA_RESTART
		act.sa_flags |= SA_RESTART;		/* SVR4, 44BSD */
#endif
	}
	if (sigaction(signo, &act, &oact) < 0)
		return(SIG_ERR);
	return(oact.sa_handler);
}
/* end signal */

Sigfunc *
Signal(int signo, Sigfunc *func)	/* for our signal() function */
{
	Sigfunc	*sigfunc;

	if ( (sigfunc = signal(signo, func)) == SIG_ERR)
		err_sys("signal error");
	return(sigfunc);
}
```

**wait和waitpid函数**

```
#include <sys/wait.h>

// @return 若成功则为进程ID，若出错则为0或-1

pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
```

##### I/O复用：select和poll函数

有些进程需要一种预先告知内核的能力，使得内核一旦发现进程指定的一个或多个I/O条件就绪，它就通知进程。
这种能力称为I/O复用，一般应用于下列场合：

- 当客户处理多个描述符（通常是交互式输入和网络套接字）是，必须使用I/O复用
- 客户同时处理多个套接字
- 一个TCP服务器既要处理监听套接字，又要处理已连接套接字
- 一个服务器既要处理TCP，又要处理UDP
- 一个服务器要处理多个服务或者多个协议

**I/O模型**

- 阻塞式I/O
- 非阻塞式I/O
- I/O复用（select和poll）
- 信号驱动式I/O（SIGIO）
- 异步I/O（POSIX的aio_系列函数）

**select函数**

select函数允许进程指示内核等待多个事件中的任何一个发生，并只在有一个或多个事件发生或经历一段指定的时间后才唤醒它。

```
#include <sys/select.h>
#include <sys/time.h>

// @maxfdp1 指定待测试的描述符的个数
// @*readset @*writeset @exceptset 要内核测试读、写和异常条件的描述符
// @*timeout 告诉内核等待的时间
// @return 若有就绪描述符则为其数目，若超时则为0，若出错则为-1
int select(int maxfdp1, fd_set *readset, fd_set *writeset, fd_set *exceptset, const struct timeval *timeout);
```

**poll函数**

poll函数提供的功能与select类似，不过在处理流设备时，它能够提供额外的信息。

```
#include <poll.h>

// @nfds 结构数组中元素的个数
// @timeout 指定poll函数返回前等待多长时间
// @return 若有就绪描述符则为其数目，若超时则为0，若出错则为-1
int poll(struct pollfd *fdarray, unsigned long nfds, int timeout);

struct pollfd {
   int   fd;         /* descriptor to check */
   short events;     /* events of interest on fd */
   short revents;    /* events that occurred on fd */
}
```

#### TCP回射程序

##### 服务器

```
#include	"unp.h"

int
main(int argc, char **argv)
{
	int					i, maxi, maxfd, listenfd, connfd, sockfd;
	int					nready, client[FD_SETSIZE];
	ssize_t				n;
	fd_set				rset, allset;
	char				buf[MAXLINE];
	socklen_t			clilen;
	struct sockaddr_in	cliaddr, servaddr;

	listenfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family      = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port        = htons(SERV_PORT);

	Bind(listenfd, (SA *) &servaddr, sizeof(servaddr));

	Listen(listenfd, LISTENQ);

	maxfd = listenfd;			/* initialize */
	maxi = -1;					/* index into client[] array */
	for (i = 0; i < FD_SETSIZE; i++)
		client[i] = -1;			/* -1 indicates available entry */
	FD_ZERO(&allset);
	FD_SET(listenfd, &allset);

	for ( ; ; ) {
		rset = allset;		/* structure assignment */
		nready = Select(maxfd+1, &rset, NULL, NULL, NULL);

		if (FD_ISSET(listenfd, &rset)) {	/* new client connection */
			clilen = sizeof(cliaddr);
			connfd = Accept(listenfd, (SA *) &cliaddr, &clilen);
#ifdef	NOTDEF
			printf("new client: %s, port %d\n",
					Inet_ntop(AF_INET, &cliaddr.sin_addr, 4, NULL),
					ntohs(cliaddr.sin_port));
#endif

			for (i = 0; i < FD_SETSIZE; i++)
				if (client[i] < 0) {
					client[i] = connfd;	/* save descriptor */
					break;
				}
			if (i == FD_SETSIZE)
				err_quit("too many clients");

			FD_SET(connfd, &allset);	/* add new descriptor to set */
			if (connfd > maxfd)
				maxfd = connfd;			/* for select */
			if (i > maxi)
				maxi = i;				/* max index in client[] array */

			if (--nready <= 0)
				continue;				/* no more readable descriptors */
		}

		for (i = 0; i <= maxi; i++) {	/* check all clients for data */
			if ( (sockfd = client[i]) < 0)
				continue;
			if (FD_ISSET(sockfd, &rset)) {
				if ( (n = Read(sockfd, buf, MAXLINE)) == 0) {
						/*4connection closed by client */
					Close(sockfd);
					FD_CLR(sockfd, &allset);
					client[i] = -1;
				} else
					Writen(sockfd, buf, n);

				if (--nready <= 0)
					break;				/* no more readable descriptors */
			}
		}
	}
}
```

```
#include	"unp.h"

void
str_echo(int sockfd)
{
	ssize_t		n;
	char		buf[MAXLINE];

again:
	while ( (n = read(sockfd, buf, MAXLINE)) > 0)
		Writen(sockfd, buf, n);

	if (n < 0 && errno == EINTR)
		goto again;
	else if (n < 0)
		err_sys("str_echo: read error");
}
```

##### 客户端

```
#include	"unp.h"

int
main(int argc, char **argv)
{
	int					sockfd;
	struct sockaddr_in	servaddr;

	if (argc != 2)
		err_quit("usage: tcpcli <IPaddress>");

	sockfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	Inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

	Connect(sockfd, (SA *) &servaddr, sizeof(servaddr));

	str_cli(stdin, sockfd);		/* do it all */

	exit(0);
}
```

```
#include	"unp.h"

void
str_cli(FILE *fp, int sockfd)
{
	int			maxfdp1, stdineof;
	fd_set		rset;
	char		buf[MAXLINE];
	int		n;

	stdineof = 0;
	FD_ZERO(&rset);
	for ( ; ; ) {
		if (stdineof == 0)
			FD_SET(fileno(fp), &rset);
		FD_SET(sockfd, &rset);
		maxfdp1 = max(fileno(fp), sockfd) + 1;
		Select(maxfdp1, &rset, NULL, NULL, NULL);

		if (FD_ISSET(sockfd, &rset)) {	/* socket is readable */
			if ( (n = Read(sockfd, buf, MAXLINE)) == 0) {
				if (stdineof == 1)
					return;		/* normal termination */
				else
					err_quit("str_cli: server terminated prematurely");
			}

			Write(fileno(stdout), buf, n);
		}

		if (FD_ISSET(fileno(fp), &rset)) {  /* input is readable */
			if ( (n = Read(fileno(fp), buf, MAXLINE)) == 0) {
				stdineof = 1;
				Shutdown(sockfd, SHUT_WR);	/* send FIN */
				FD_CLR(fileno(fp), &rset);
				continue;
			}

			Writen(sockfd, buf, n);
		}
	}
}
```

### 基本UDP套接字

#### 相关函数

##### recvfrom和sendto函数

```
#include <sys/socket.h>

// @sockfd 描述符
// @*buff 指向读入或写出缓冲区的指针
// @nbytes 读写字节数
// @flags 
// @return 若成功则为读或写的字节数，若出错则为-1
 
ssize_t recvfrom(int sockfd, void *buff, size_t nbytes, int flags, struct sockaddr *from, socklen_t *addrlen);

ssize_t sendto(int sockfd, const void *buff, size_t nbytes, int flags, const struct sockaddr *to, socklen_t addrlen);
```

##### UDP的connect函数

给UDP套接字调用connect，与TCP不同的是没有三路握手过程。内核只是检查是否存在立即可知的错误，记录对端的IP地址和端口号，然后立即返回给调用进程。

#### UDP回射程序

##### 服务器

```
#include	"unp.h"

int
main(int argc, char **argv)
{
	int					sockfd;
	struct sockaddr_in	servaddr, cliaddr;

	sockfd = Socket(AF_INET, SOCK_DGRAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family      = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port        = htons(SERV_PORT);

	Bind(sockfd, (SA *) &servaddr, sizeof(servaddr));

	dg_echo(sockfd, (SA *) &cliaddr, sizeof(cliaddr));
}
```

```
#include	"unp.h"

void
dg_echo(int sockfd, SA *pcliaddr, socklen_t clilen)
{
	int			n;
	socklen_t	len;
	char		mesg[MAXLINE];

	for ( ; ; ) {
		len = clilen;
		n = Recvfrom(sockfd, mesg, MAXLINE, 0, pcliaddr, &len);

		Sendto(sockfd, mesg, n, 0, pcliaddr, len);
	}
}
```

##### 客户端

```
#include	"unp.h"

int
main(int argc, char **argv)
{
	int					sockfd;
	struct sockaddr_in	servaddr;

	if (argc != 2)
		err_quit("usage: udpcli <IPaddress>");

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	Inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

	sockfd = Socket(AF_INET, SOCK_DGRAM, 0);

	dg_cli(stdin, sockfd, (SA *) &servaddr, sizeof(servaddr));

	exit(0);
}
```

```
#include	"unp.h"

void
dg_cli(FILE *fp, int sockfd, const SA *pservaddr, socklen_t servlen)
{
	int				n;
	char			sendline[MAXLINE], recvline[MAXLINE + 1];
	socklen_t		len;
	struct sockaddr	*preply_addr;

	preply_addr = Malloc(servlen);

	while (Fgets(sendline, MAXLINE, fp) != NULL) {

		Sendto(sockfd, sendline, strlen(sendline), 0, pservaddr, servlen);

		len = servlen;
		n = Recvfrom(sockfd, recvline, MAXLINE, 0, preply_addr, &len);
		if (len != servlen || memcmp(pservaddr, preply_addr, len) != 0) {
			printf("reply from %s (ignored)\n",
					Sock_ntop(preply_addr, len));
			continue;
		}

		recvline[n] = 0;	/* null terminate */
		Fputs(recvline, stdout);
	}
}
```

#### 使用select处理TCP和UDP的回射服务器

```
#include	"unp.h"

int
main(int argc, char **argv)
{
	int					listenfd, connfd, udpfd, nready, maxfdp1;
	char				mesg[MAXLINE];
	pid_t				childpid;
	fd_set				rset;
	ssize_t				n;
	socklen_t			len;
	const int			on = 1;
	struct sockaddr_in	cliaddr, servaddr;
	void				sig_chld(int);

		/* 4create listening TCP socket */
	listenfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family      = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port        = htons(SERV_PORT);

	Setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on));
	Bind(listenfd, (SA *) &servaddr, sizeof(servaddr));

	Listen(listenfd, LISTENQ);

		/* 4create UDP socket */
	udpfd = Socket(AF_INET, SOCK_DGRAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family      = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port        = htons(SERV_PORT);

	Bind(udpfd, (SA *) &servaddr, sizeof(servaddr));

	Signal(SIGCHLD, sig_chld);	/* must call waitpid() */

	FD_ZERO(&rset);
	maxfdp1 = max(listenfd, udpfd) + 1;
	for ( ; ; ) {
		FD_SET(listenfd, &rset);
		FD_SET(udpfd, &rset);
		if ( (nready = select(maxfdp1, &rset, NULL, NULL, NULL)) < 0) {
			if (errno == EINTR)
				continue;		/* back to for() */
			else
				err_sys("select error");
		}

		if (FD_ISSET(listenfd, &rset)) {
			len = sizeof(cliaddr);
			connfd = Accept(listenfd, (SA *) &cliaddr, &len);
	
			if ( (childpid = Fork()) == 0) {	/* child process */
				Close(listenfd);	/* close listening socket */
				str_echo(connfd);	/* process the request */
				exit(0);
			}
			Close(connfd);			/* parent closes connected socket */
		}

		if (FD_ISSET(udpfd, &rset)) {
			len = sizeof(cliaddr);
			n = Recvfrom(udpfd, mesg, MAXLINE, 0, (SA *) &cliaddr, &len);

			Sendto(udpfd, mesg, n, 0, (SA *) &cliaddr, len);
		}
	}
}
```

### 基本SCTP套接字编程

#### 相关函数

##### sctp_bindx函数

##### sctp_connectx函数

##### sctp_getpaddrs函数

##### sctp_freepaddrs函数

##### sctp_getladdrs函数

##### sctp_freeladdrs函数

##### sctp_sendmsg函数

##### sctp_recvmsg函数

##### sctp_opt_info函数

##### sctp_peeloff函数

##### shutdown函数

#### SCTP回射程序