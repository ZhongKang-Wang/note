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







# day01-从一个简单的socket开始

----

`socket`是操作系统基于传输层或者网络层，面向应用层提供的`api`，可以认为是一种系统调用。

```c++
#include <sys/socket.h>
int sockfd = socket(AF_INET, SOCK_STREAM, 0);
```

- 第一个参数：`IP`地址类型，`AF_INET`表示使用`IPv4`，如果使用IPv6请使用AF_INET6。
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

服务器通过`socket`建立了一个监听标识符，这个监听标识符占用端口8888，绑定到了本机的所有`IP`上，即表示监听本机所有IP地址；

客户端通过服务器的IP地址和端口号连接到服务器。

如果没有客户端发起连接，服务器会在 `accept()` 函数处一直阻塞（暂停执行），直到有客户端连接到来或者发生错误（如被信号中断）。

**阻塞状态**（暂停运行，不占用 CPU 资源），直到有客户端发起连接请求，操作系统才会唤醒进程继续执行。



# day03-epoll

----



在day01的基础上，我们在day02添加了一些错误处理，并在day03完成了一个简单的echo服务器。但是这个服务器它只能处理一个`client`。

为了向成千上万的客户提供服务，我们需要为服务器添加`IO复用`技术。

在Linux系统中，IO复用使用`select`, `poll`和`epoll`来实现。



接下来我们要学习它们的区别在哪里。

Q1：IO复用能做什么？

> IO复用使得程序能同时监听多个文件描述符。
>
> 换句话说，负责监听的文件描述符经过复用之后还可以监听已经连接的客户有没有说话的需求。

注意：IO复用本身是阻塞的，当多个文件描述符处于就绪态，如果不采取额外措施（并发），便只能一个一个来。



1. `select`

在一段时间内，监听用户感兴趣的文件描述符上的可读、可写和异常等事件。

```c++
#include<sys/select.h>
int select(int nfds, fd_set* readfds, fd_set* writefds, fd_set* exceptfds, struct timeval* timeout);
# 1) nfds 指定被监听的文件描述符的总数 通常被设置为select监听的所有文件描述符的最大值 + 1
# 2) readfds\writefds\exceptfds 可读、可写、异常等事件对应的文件描述符集合
# 3) timeout参数用于设置select函数的超时时间，如果给null，则select一直阻塞，直到某个文件描述符就绪
# 4) select返回就绪（可读、可写、异常）文件描述符的总数
```

2. `poll`

和`select()`类似，指定时间内轮询一定数量的文件描述符。

```c++
#include<poll.h>
int poll(struct pollfd* fds, nfds_t nfds, int timeout);
# 1) fds是一个pollfd类型的数组，指定我们感兴趣的文件描述符上发生的可读、可写和异常等事件。
# 2) nfds参数指定被监听事件集合fds的大小
# 3) timeout为-1时，poll阻塞，等待某个事件发生；timeout为0时，poll调用立即返回

struct pollfd {
    int fd; // 文件描述符
    short events; // 注册的事件
    short revents; // 实际发生的事件
}
```

3. `epoll`

`epoll`使用一组函数完成任务



- **select**：
  靠 **“用户态把事件集合拷贝到内核态，内核轮询检查事件”** 。

  每次调用要把读、写、异常的文件描述符集合（fd_set）从用户态传到内核态，内核逐个遍历判断是否就绪，效率随 fd 数量上升而下降（O (n) 遍历）。而且最大支持 fd 数有限（受 `FD_SETSIZE` 限制，一般是 1024 或 2048 ）。

- **poll**：
  机制和 select 类似，**核心还是内核轮询检查** 。但用 `pollfd` 结构体数组替代 fd_set，解决了 “每次调用要重置事件集合” 的问题（select 每次触发后，未就绪的 fd 会被内核清空，用户态得重新填；poll 的 `pollfd` 里有 `revents` 单独存内核返回的事件，`events` 保留用户关心的事件，不用每次重置 ）。不过本质还是内核遍历所有 fd 找就绪的，依旧是 O (n) ，且大并发下效率问题没解决。

- **epoll**：
  靠 **“内核主动通知 + 红黑树管理事件”** 。

  创建 epoll 时，内核会建红黑树存用户关心的 fd 和事件，还有就绪链表存已就绪事件。

  注册事件（`epoll_ctl`）时，把 fd 放到红黑树；事件就绪时，内核主动把 fd 复制到就绪链表。调用 `epoll_wait` 时，直接从就绪链表拿数据，**不用遍历所有 fd** （O (1) ，只处理就绪的）。而且支持海量 fd（理论上只受系统文件描述符限制 ）。







- **select/poll**：都是 **“水平触发（LT）”** 。

比如读事件，只要 fd 有数据没读完，每次调用 `select/poll` 都会触发；写事件同理，只要缓冲区有空间，就会持续触发。好处是简单，坏处是高并发下可能重复触发、冗余处理。

- **epoll**：默认 **“水平触发（LT）”** ，但支持 **“边缘触发（ET）”** 。
  - LT 模式和 select/poll 类似，数据没处理完会持续触发；
  - **ET 模式只在事件 “从无到有” 变化时触发一次** （比如 socket 从 “无数据” 变 “有数据” 触发读事件，读完数据前不会再触发，必须程序主动把数据读完 ）。ET 模式更高效，但要求程序 “一次处理彻底”，否则可能丢事件，所以一般配合 **非阻塞 fd** 使用。





- **select**： 老旧系统、简单小并发场景（比如嵌入式设备，代码改动小），但受 fd 数量限制大，现在用得少。
- **poll**： 比 select 灵活点（不用重置事件集合），但本质还是轮询，大并发一样拉胯，也逐渐被替代。
- **epoll**： 高并发、高性能网络编程（比如 Nginx、Redis 等）的首选，尤其是 ET 模式 + 非阻塞 fd 配合，能极大减少无效遍历，提升效率。







# 20250828

----

Q1：阻塞模式和非阻塞模式的区别？

### 1. 阻塞模式（Blocking）

- **核心特点**：当程序执行 I/O 操作时，会暂停当前线程的执行，直到 I/O 操作完成（成功或失败）后才继续往下执行。
- 行为表现：
  - 例如调用`read()`读取文件时，如果数据未准备好，线程会进入休眠状态，不占用 CPU 资源。
  - 直到数据准备完毕，操作系统会将数据从内核缓冲区复制到用户缓冲区，完成读取操作。
  - 最后，线程被唤醒，`read()` 函数才会返回**实际读取的字节数**（或 -1 表示出错），程序继续执行后续逻辑。
- 劣势：
  - 在高并发场景下效率低下，一个线程一次只能处理一个 I/O 操作。
  - 可能导致资源浪费（如线程阻塞时仍占用内存等资源）。

### 2. 非阻塞模式（Non-blocking）

- **核心特点**：当程序执行 I/O 操作时，无论操作是否完成，都会立即返回结果。如果操作未完成，会返回一个特定的标识（如`EAGAIN`或`EWOULDBLOCK`）。
- 行为表现：
  - 例如调用非阻塞的`read()`时，如果数据未准备好，会立即返回 "操作暂时无法完成" 的信息，线程可以继续执行其他任务。
  - 程序需要通过轮询（Polling）或事件通知机制来判断操作是否完成。
- 优势：
  - 单线程可以处理多个 I/O 操作，提高了系统的并发处理能力。
  - 避免了线程阻塞带来的资源浪费。
- 劣势：
  - 代码逻辑复杂，需要处理 "操作未完成" 的各种情况。
  - 盲目的轮询可能导致 CPU 资源浪费。





Q4：为什么使用ET(Edge Trigger，边沿触发模式)的文件描述符一定要是非阻塞的？



对于采用ET工作模式的文件描述符，当epoll_wait检测到其上有事件发生并将此事通知应用程序后，应用程序必须立即处理该事件，因为epoll_wait调用后续不再向应用程序通知这一事件。

使用阻塞IO会造成两个问题：

1. 一次数据没读完，后续永远读不到

举个例子。

客户端发数据后，Socket从“无数据”变为“有数据”，`epoll_wait`检测到这个“边沿变化”，通知应用程序“可以读了”。应用程序调用`read(fd, buf, 50)`（想读50字节），此时有100字节数据，`read()`能顺利读完50字节并返回。但Socket上还剩50字节数据没读——**ET模式只在“无数据→有数据”时通知一次，现在数据没读完，状态没有“新的边沿变化”，`epoll_wait`不会再通知**

应用程序误以为“数据已经读完了”，不会再主动调用`read()`，这剩下的50字节数据就被永久“遗忘”，客户端后续再发数据也可能被阻塞（因为服务器没处理完之前的数据）

2. `read()`直接卡死线程，程序彻底无响应

客户端只发了 **30字节数据**，应用程序调用`read(fd, buf, 100)`（想读100字节）。因为Socket是阻塞的，`read()`发现“只有30字节，不够100字节”，就会让当前线程 **休眠等待**（直到有更多数据过来）；   

但ET模式已经通知过一次“可读事件”，后续即使客户端再发70字节（凑够100字节），`epoll_wait`也不会再通知——因为“有数据→更多数据”不算“新的边沿变化”；   -

结果就是：线程永远卡在`read()`里休眠，既处理不了这个Socket的后续数据，也处理不了其他FD的事件（比如新客户端连接），程序直接“假死”。 





如果把Socket设为 **非阻塞**，上述两个问题会被完美规避，核心是“**非阻塞IO不会让线程卡住，能强制把所有数据读完”**

还是刚才的场景（客户端发100字节，应用程序读100字节）： 

`epoll_wait`通知“可读事件”后，应用程序调用`read(fd, buf, 100)`；   

第一次`read()`读了100字节（如果数据够），直接返回，处理完成；   

如果数据不够（比如只发了30字节），非阻塞的`read()`会立刻返回“-1”，并设置`errno=EAGAIN`（“暂时没数据了”）；   

应用程序看到`EAGAIN`就知道“不是真的出错，而是当前数据读完了”，于是停止`read()`；   

后续如果客户端再发数据（比如70字节），Socket状态从“无新数据”变为“有新数据”，触发新的“边沿变化”，`epoll_wait`会再次通知，应用程序再调用`read()`即可。 





非阻塞IO让应用程序能通过“循环读+判断`EAGAIN`”的方式，**把当前FD上所有能读的数据一次性读完**，不会遗漏；同时也不会让线程卡住，保证程序能继续处理其他事件。  



总结：ET模式的核心是“**只通知一次状态变化**”，要求应用程序必须“一次性处理完当前FD上的所有事件”；而阻塞IO会导致“要么数据没读完就停手，要么卡住线程”，无法满足这个要求；只有非阻塞IO能强制“读完所有数据就停，没数据就立刻返回”，完美匹配ET的触发规则。 因此，**ET模式的文件描述符必须设为非阻塞**，这是由两者的工作机制共同决定的，缺一不可。



# 20250902

Q1: 为什么监听socket使用epoll也要设置成非阻塞的呢？



监听 socket 的唯一作用是 “接收新的连接请求”，对应的核心系统调用是`accept()`。监听 socket（`listen_fd`）维护着一个**半连接队列**（SYN_RECV 状态）和**全连接队列**（ESTABLISHED 状态）。`accept()`的作用是从**全连接队列**中取出一个已完成三次握手的连接。



epoll 通知 “`listen_fd`可读”，本质是 “全连接队列非空”（有已完成握手的连接可处理）。



边缘触发（ET）模式下，通知具有 “一次性”

- ET 模式下，epoll 仅在 “全连接队列从空变为非空” 时通知一次。

- 假设：线程 A 收到通知后调用`accept()`，但此时全连接队列中的连接已被线程 B 抢先取走（队列变空）。

- 后果：阻塞模式的`accept()`会一直阻塞，没有机会调用`epoll_wait`，因此无法处理 epoll 的任何新通知。线程 A 永久卡死。

  

水平触发（LT）模式下

- 只要全连接队列非空，epoll 会反复通知。但如果队列已空（连接被其他线程取走），阻塞`accept()`仍会卡住线程。

- 此时，即使后续有新连接进入队列（LT模式的反复通知特性，epoll 再次通知），但线程 A 仍卡在之前的`accept()`调用中，**没有机会再次调用`epoll_wait`，无法处理新通知 —— 相当于 “旧的阻塞调用占用了线程，新的事件无法被处理”。

  

- 非阻塞模式下，`accept()`能在 “无可用连接” 时立即返回，确保线程始终能回到`epoll_wait`等待新事件，而不是卡在accept()`调用中。





Q2：为什么监听socket最好设置成水平触发而非边缘触发？区别是什么？



在多线程 / 多进程模型中，多个线程可能同时处理监听 socket 的 “可读” 事件，此时：

- **水平触发（LT）**：即使一个线程抢先 `accept()` 取走了连接，只要队列还有剩余连接，其他线程调用 `epoll_wait` 仍会收到通知，确保所有连接都能被处理。
  即使队列被取空，后续新连接到来时，LT 仍会再次通知，无需担心 “错过事件”。

- **边缘触发（ET）**：若一个线程收到通知后未处理完所有连接（例如被其他线程抢占 CPU），后续线程不会再收到通知，剩余连接会一直留在队列中（直到新连接到来触发 “空→非空” 状态变化）。
  这会导致连接处理延迟，甚至在高并发场景下积累大量未处理连接。





# 20250907

服务器做的事情就是监听epoll上的事件，然后对不同事件类型进行不同的处理。

这种以事件为核心的模式又叫事件驱动。



`Reactor`模式和`Proactor`模式

同步IO常用于实现Reactor模式，异步IO则用于实现Praactor模式。可以用同步IO模拟Proactor模式。



Proactor 核心流程

1. **应用程序（主线程）发起异步 I/O 请求**：明确告诉内核 “要做什么 I/O 操作”（比如读哪个文件描述符、数据存到用户空间的哪个缓冲区、读多少字节等），然后主线程不需要阻塞等待，直接去处理其他任务。
2. **内核独立完成 I/O 操作**：内核接手后，会自行完成整个 I/O 过程（包括等待数据就绪、将数据从内核空间复制到用户空间，或反之），期间完全不需要应用程序干预。
3. **内核通知操作完成**：当内核彻底完成 I/O 操作后，会主动通知应用程序（比如通过回调函数、事件队列等方式）。
4. **应用程序处理结果**：应用程序收到通知后，直接处理已经准备好的 I/O 结果（比如读取缓冲区中的数据），无需再与内核交互进行 I/O 操作本身。

Proactor 模式中**内核会完成整个 I/O 操作的细节**，应用程序只需要 “发起请求” 和 “处理结果”。





Reactor 模式

- **内核态**负责：执行 epoll_wait 系统调用、检测 I/O 就绪、将就绪事件复制到用户态缓冲区、执行 read/write 等实际 I/O 操作（数据在内核态与用户态之间的复制）。
- **用户态**负责：发起 epoll_wait 调用、主线程分发事件、工作线程处理业务逻辑（以及决定何时发起 read/write 等系统调用）。





同步 I/O 与异步 I/O 的核心区别在于：

- **同步 I/O**：应用程序发起 I/O 请求后，必须等待 I/O 操作（如数据从内核缓冲区复制到用户缓冲区，或反之）完成后才能继续执行后续逻辑，哪怕这个等待过程可以通过非阻塞方式 “轮询” 而非 “阻塞睡眠”。
- **异步 I/O**：应用程序发起 I/O 请求后，无需等待，内核会自行完成整个 I/O 操作（包括数据复制），完成后再通知应用程序处理结果，应用程序全程不参与 I/O 操作的执行




## Reactor 与 Proactor 模式：区别、工作流程及同步 / 异步本质

### 概述

Reactor 和 Proactor 是两种经典的**事件驱动 I/O 模型**，广泛应用于高性能网络编程（如服务器开发）。两者的核心目标是高效处理多并发 I/O 请求，但实现方式和底层逻辑存在本质区别，这也导致了它们分别被归类为 “同步” 和 “异步” 模式。

#### 一、Reactor 模式：同步 I/O 的事件驱动优化

##### 1.1 核心定义

Reactor 模式是**同步 I/O 多路复用**的典型实现，其核心思想是：**由一个 “反应器”（Reactor）统一监听 I/O 事件，当事件就绪后通知应用程序处理**。应用程序需要主动参与 I/O 操作的最终完成。

##### 1.2 工作流程

Reactor 模式的工作流程可分为 4 个步骤：

1. **注册事件**：应用程序向 Reactor 注册感兴趣的 I/O 事件（如 “可读”“可写”），并关联对应的处理逻辑（回调函数）。
2. **等待事件就绪**：Reactor 通过**多路复用器**（如 Linux 的 `epoll`）阻塞，等待事件就绪。这一步是**内核态操作**（系统调用），内核负责检测 I/O 设备状态。
3. **分发事件**：当内核检测到 I/O 事件就绪（如数据到达内核缓冲区），多路复用器会将事件从内核态复制到用户态，Reactor 唤醒并根据事件类型调用对应的处理逻辑。
4. **处理 I/O 操作**：应用程序的工作线程（用户态）主动发起 I/O 系统调用（如 `read`/`write`），完成数据从内核缓冲区到用户缓冲区的复制（或反之）。**这一步是同步阻塞的**—— 线程必须等待复制完成才能继续执行。





#### 二、Proactor 模式：真正的异步 I/O 模型

##### 2.1 核心定义

Proactor 模式是**异步 I/O**的典型实现，其核心思想是：**应用程序发起 I/O 请求后完全脱手，由内核完成整个 I/O 操作（包括数据复制），完成后再通知应用程序处理结果**

##### 2.2 工作流程

Proactor 模式的工作流程可分为 4 个步骤：

1. **发起异步 I/O 请求**：应用程序向内核发起异步 I/O 请求（如 `aio_read`），明确操作类型（读 / 写）、目标缓冲区、数据长度等，并注册回调函数。发起后，应用程序无需等待，可立即处理其他任务。

2. 内核执行完整 I/O 操作

   ：内核接手后，自行完成整个 I/O 过程：

   - 等待设备就绪（如数据到达）；
   - 将数据从设备复制到内核缓冲区；
   - 将数据从内核缓冲区复制到用户指定的缓冲区。

3. **通知操作完成**：内核完成所有 I/O 操作后，通过事件队列或信号通知 Proactor。

4. **处理结果**：Proactor 调用应用程序注册的回调函数，应用程序直接处理用户缓冲区中已准备好的数据，无需再与内核交互。



#### 三、核心区别对比

| 维度                    | Reactor 模式                           | Proactor 模式                            |
| ----------------------- | -------------------------------------- | ---------------------------------------- |
| **I/O 操作主体**        | 应用程序主动执行 I/O 操作（如 `read`） | 内核全程执行 I/O 操作                    |
| **事件通知内容**        | 通知 “事件就绪”（如 “可读取”）         | 通知 “操作完成”（如 “数据已读入缓冲区”） |
| **用户态 / 内核态切换** | 频繁（事件就绪后需再次系统调用）       | 少（仅发起请求和接收结果时切换）         |
| **复杂度**              | 应用程序需处理 I/O 细节，实现较简单    | 内核实现复杂，应用程序逻辑更简洁         |

### 四、为何 Reactor 是同步，Proactor 是异步？

同步与异步的核心区别在于 **“应用程序是否需要等待 I/O 操作完成”**：

- **Reactor 是同步的**：
  **实际的 I/O 操作（数据复制）需应用程序主动发起并等待完成**。例如，当 “可读事件” 发生后，应用程序必须调用 `read` 系统调用，且线程会阻塞到数据从内核缓冲区复制到用户缓冲区为止。因此，应用程序仍需参与并等待 I/O 操作的最终完成，符合**同步 I/O**的定义。
- **Proactor 是异步的**：
  应用程序发起异步请求后，**无需等待任何 I/O 操作**，内核会自行完成从 “等待设备就绪” 到 “数据复制” 的全过程。只有当所有 I/O 操作彻底完成后，应用程序才会被通知处理结果。整个过程中，应用程序不参与 I/O 执行，也不等待其完成，符合**异步 I/O**的定义。

### 五、适用场景

- **Reactor 模式**：
  适用于 I/O 操作耗时短、并发量中等的场景（如 Web 服务器、聊天服务器）。实现简单，对内核依赖低（多数操作系统支持 `epoll`/`select`）。
- **Proactor 模式**：
  适用于 I/O 操作耗时长、需要极致性能的场景（如大文件传输、数据库读写）。但依赖内核对异步 I/O 的支持（如 Linux 的 `libaio`、Windows 的 `IOCP`），实现复杂度较高。

### 六、总结

- Reactor 是 “同步 I/O + 事件驱动”，核心是 “等待事件就绪后由应用程序处理 I/O”；
- Proactor 是 “异步 I/O + 事件驱动”，核心是 “内核处理完整 I/O 后通知应用程序”；
- 同步与异步的分界点在于 **“I/O 操作的执行和等待是否由应用程序承担”**。





