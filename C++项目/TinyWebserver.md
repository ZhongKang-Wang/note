# 20250812

Web服务器端通过`socket`监听来自用户的请求。

```c++
#include <sys/socket.h> // 套接字头文件，是运输层、网络层向上暴露出的一些接口
#include <netinet/in.h> // network internet 定义了 TCP/IP 协议中 IPv4/IPv6 地址结构及相关常量，供网络编程描述通信地址和协议类型
#include <arpa/inet.h>  // 提供 IP 地址与字符串之间的转换函数
#include <string.h>
/* 创建监听socket文件描述符 */
int listenfd = socket(PF_INET, SOCK_STREAM, 0); 
/* 监听socket的TCP/IP的IPV4 socket地址 */
struct sockaddr_in address;
// bzero(&address, sizeof(address));
memset(&address, 0, sizeof(address));
address.sin_family = AF_INET;
address.sin_addr.s_addr = 
htonl(INADDR_ANY);  /* INADDR_ANY：将套接字绑定到所有可用的接口 */
// sin_addr是一个包含unit32_t的结构体
/* htonl将主机字节序 转换成符合网络标准的网络字节序 */
address.sin_port = htons(port);
// sin_port是uint16_t

// 端口是传输层的概念，IP地址+端口被记作套接字地址，标识了一个特定的进程

int flag = 1;
/* SO_REUSEADDR 允许端口被重复使用 */
setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &flag, sizeof(flag));
/* 绑定socket和它的地址 */
ret = bind(listenfd, (struct sockaddr*)&address, sizeof(address));  
/* 创建监听队列以存放待处理的客户连接，在这些客户连接被accept()之前 */
ret = listen(listenfd, 5);
```



服务器通过`epoll`这种I/O复用技术（还有`select`和`poll`）来实现对监听socket（`listenfd`）和连接socket（客户请求）的同时监听

```c++
#include <sys/epoll.h> // 使用I/O复用技术

/* 将fd上的EPOLLIN和EPOLLET事件注册到epollfd指示的epoll内核事件中 */
void addfd(int epollfd, int fd, bool one_shot) {
    epoll_event event; // 定义一个epoll_event结构体
    event.data.fd = fd;
    event.events = EPOLLIN | EPOLLET | EPOLLRDHUP;
    // EPOLLIN 读事件 ex:客户端发来数据
    // EPOLLET 边缘触发事件，只在状态从无到有的状态变化瞬间触发一次 ex:有数据时EPOLLIN只通知一次，程序需要一次性把数据读走
    // EPOLLRDHUP romete half close，客户端主动关闭连接
    /* 针对connfd，开启EPOLLONESHOT，因为我们希望每个socket在任意时刻都只被一个线程处理 */
    if(one_shot)
        event.events |= EPOLLONESHOT;
    epoll_ctl(epollfd, EPOLL_CTL_ADD, fd, &event); // 修改epoll事件
    setnonblocking(fd); // 设置为非阻塞模式
}
/* 创建一个额外的文件描述符来唯一标识内核中的epoll事件表 */
int epollfd = epoll_create(5);  
/* 用于存储epoll事件表中就绪事件的event数组 */
epoll_event events[MAX_EVENT_NUMBER];  
/* 主线程往epoll内核事件表中注册监听socket事件，当listen到新的客户连接时，listenfd变为就绪事件 */
addfd(epollfd, listenfd, false);  
/* 主线程调用epoll_wait等待一组文件描述符上的事件，并将当前所有就绪的epoll_event复制到events数组中 */
int number = epoll_wait(epollfd, events, MAX_EVENT_NUMBER, -1); // 表示等待事情发生
/* 然后我们遍历这一数组以处理这些已经就绪的事件 */
for(int i = 0; i < number; ++i) {
    int sockfd = events[i].data.fd;  // 事件表中就绪的socket文件描述符
    if(sockfd == listenfd) {  // 当listen到新的用户连接，listenfd上则产生就绪事件
        struct sockaddr_in client_address;
        socklen_t client_addrlength = sizeof(client_address);
        /* ET模式 */
        while(1) {
            /* accept()返回一个新的socket文件描述符用于send()和recv() */
            int connfd = accept(listenfd, (struct sockaddr *) &client_address, &client_addrlength);
            /* 并将connfd注册到内核事件表中 */
            users[connfd].init(connfd, client_address);
            /* ... */
        }
    }
    else if(events[i].events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR)) {
        // 如有异常，则直接关闭客户连接，并删除该用户的timer
        /* ... */
    }
    else if(events[i].events & EPOLLIN) {
        /* 当这一sockfd上有可读事件时，epoll_wait通知主线程。*/
        if(users[sockfd].read()) { /* 主线程从这一sockfd循环读取数据, 直到没有更多数据可读 */
            pool->append(users + sockfd);  /* 然后将读取到的数据封装成一个请求对象并插入请求队列 */
            /* ... */
        }
        else
            /* ... */
    }
    else if(events[i].events & EPOLLOUT) {
        /* 当这一sockfd上有可写事件时，epoll_wait通知主线程。主线程往socket上写入服务器处理客户请求的结果 */
        if(users[sockfd].write()) {
            /* ... */
        }
        else
            /* ... */
    }
}
```

# 20250821

我们先来熟悉一些网络编程常用的函数





# 20250825

## day01-从一个简单的socket开始

```c++
#include <sys/socket.h>
int sockfd = socket(AF_INET, SOCK_STREAM, 0);
```

- 第一个参数：IP地址类型，AF_INET表示使用IPv4，如果使用IPv6请使用AF_INET6。
- 第二个参数：数据传输方式，SOCK_STREAM表示流格式、面向连接，多用于TCP。SOCK_DGRAM表示数据报(datagram)格式、无连接，多用于UDP。
- 第三个参数：协议，0表示根据前面的两个参数自动推导协议类型。设置为IPPROTO_TCP和IPPTOTO_UDP，分别表示TCP和UDP。(**Internet Protocol Protocol**,基于IP层之上的协议)



`Server.cpp`

```c++
/* 
    day01 socket网络编程
*/
#include<sys/socket.h>
#include <arpa/inet.h> //这个头文件包含了<netinet/in.h>，同时也提供了IP地址和字符串之间的转换函数
#include <cstring>
#include <iostream>
#include <unistd.h> // 包含close()
using namespace std;
int main() {
    // 1. 创建套接字
    int sockfd = socket(AF_INET, SOCK_STREAM, 0); // ipv4 tcp
    if (sockfd == -1) {
        perror("socket creation failed");
        return -1;
    }

    // 2. 配置服务器端口号和地址
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr)); // 定义在c头文件string.h中

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY; // 让服务器监听自己本机的所有IP
    serv_addr.sin_port = htons(8888); // 将主机字节序转换成符合网络标准的字节序
    
    // 3. 绑定地址和端口号
    if (bind(sockfd, (sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
        perror("bind failed");
        close(sockfd);
        return -1;
    }

    // 4. 开始监听
    if (listen(sockfd, SOMAXCONN) == -1) {
        perror("listen failed");
        close(sockfd);
        return -1;
    }
    cout << "Server listening on port 8888..." << endl;

    // 5. 接受客户端连接
    struct sockaddr_in clnt_addr;
    socklen_t clnt_addr_len = sizeof(clnt_addr);
    bzero(&clnt_addr, sizeof(clnt_addr));

    int clnt_sockfd = accept(sockfd, (sockaddr*)&clnt_addr, &clnt_addr_len);
    if (clnt_sockfd == -1) {
        perror("accept failed");
        close(sockfd);
        return -1;
    }

    // 打印客户端信息
    printf("New client connected! fd: %d, IP: %s, Port: %d\n",
           clnt_sockfd,
           inet_ntoa(clnt_addr.sin_addr),
           ntohs(clnt_addr.sin_port));

    // 6. 关闭连接（实际中可能会先收发数据）
    close(clnt_sockfd);
    close(sockfd);
    return 0;
}
```



`Client.cpp`

```c++
#include<iostream>
#include<sys/socket.h>
#include <arpa/inet.h> //这个头文件包含了<netinet/in.h>，同时也提供了IP地址和字符串之间的转换函数
#include <cstring>
#include <unistd.h>
using namespace std;
int main() {
    // 1. 创建套接字
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd == -1) {
        perror("socket creation failed");
        return -1;
    }

    // 2. 配置服务器地址（要连接的目标）
    struct sockaddr_in serv_addr;
    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1"); // 连接本机服务器
    serv_addr.sin_port = htons(8888); // 服务器端口

    // 3. 发起连接
    int res = connect(sockfd, (sockaddr*)&serv_addr, sizeof(serv_addr));
    if (res == 0) {
        printf("connect success!\n");
    } else {
        perror("connect failure");
        close(sockfd);
        return -1;
    }

    // 4. 关闭套接字（实际中可能会先收发数据）
    close(sockfd);
    return 0;
}
```

服务器通过`socket`建立了一个监听标识符，这个监听标识符占用端口8888，绑定到了本机的所有IP上，即表示监听本机所有IP地址；客户端通过服务器的IP地址和端口号连接到服务器。

如果没有客户端发起连接，服务器会在 `accept()` 函数处一直阻塞（暂停执行），直到有客户端连接到来或者发生错误（如被信号中断）。

**阻塞状态**（暂停运行，不占用 CPU 资源），直到有客户端发起连接请求，操作系统才会唤醒进程继续执行。



# 20250827

在day01的基础上，我们在day02添加了一些错误处理，并在day03完成了一个简单的echo服务器。但是这个服务器它只能处理一个client。为了向成千上万的客户提供服务，我们需要为服务器添加`IO复用`这项技术。

在Linux系统中，IO复用使用`select`, `poll`和`epoll`来实现。

接下来我们要学习它们的区别在哪里。

Q1：IO复用能做什么？

IO复用使得程序能同时监听多个文件描述符。换句话说，负责监听的文件描述符经过复用之后还可以监听已经连接的客户有没有说话的需求。

注意：IO复用本身是阻塞的，当多个文件描述符处于就绪态，如果不采取额外措施（并发），便只能一个一个来。



```c++
#include<sys/select.h>
int select(int nfds, fd_set* readfds, fd_set* writefds, fd_set* exceptfds, struct timeval* timeout);
# 1) nfds 指定被监听的文件描述符的总数 通常被设置为select监听的所有文件描述符的最大值 + 1
# 2) readfds\writefds\exceptfds 可读、可写、异常等事件对应的文件描述符集合
# 3) timeout参数用于设置select函数的超时时间，如果给null，则select一直阻塞，知道某个文件描述符就绪
# 4) select返回就绪（可读、可写、异常）文件描述符的总数

#include<poll.h>
int poll(struct pollfd* fds, nfds_t nfds, int timeout);

#
```

