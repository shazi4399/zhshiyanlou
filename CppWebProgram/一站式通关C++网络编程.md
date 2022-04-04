https://www.lanqiao.cn/courses/3496

#  一 网络编程概述

## 1 实验介绍

通过本小节的内容讲解及部分示例程序演示，对 Linux 下网络编程有一定的基础了解和知识掌握。本小节从理解网络编程和 socket 套接字到一系列对网络地址分析、转换、域名系统等等网络名称的解释，让大家在课程的最开始对网络编程内容的涉及有一定地理解。

#### 知识点

- 理解网络编程和 socket 套接字
- 基于 Linux 的文件操作
- Socket 套接字协议及传输特性
- IP 地址和端口号
- 网络字节序和地址变换
- 网络地址的初始化
- 域名系统、IP 地址和域名之间的转换

## 2 理解网络编程和 socket 套接字

简单理解，网络编程其实就是基于编写能够使得两台连接网络的计算机之间进行数据交互的程序。通过以太网络的物理连接能够使得两台计算机之间实现物理连接，但是如何能正常实现两台计算机上的软件交互呢。下面通过一个图示简单介绍一下著名的网络编程 OSI 七层模型，从中能够直观理解我们在网络编程时所处的交互层次，从而对具体应该做的数据处理有一定理解。

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/2c051c065b8e6e32777e5e614715ec49-0)

从上图中的模型能够看到，传输层、会话层、表示层、应用层这几部分都应该算是网络编程的主要部分，而其中最基础的也就是处于传输层的 TCP、UDP 协议实现。不论是 TCP 还是 UDP 协议在操作系统中实现都需要借助 socket 套接字作为软件接口来进行操作实现。

在 Linux 操作系统下，socket 套接字有一系列相关的系统 API 函数，这里分别简单介绍一下，在后面的网络编程应用中，这些都是最基本的操作接口，借助这些操作接口我们能够在操作系统上实现网络程序的数据传输和应用。

```c++
#include <sys/socket.h>
//成功返回文件描述符，失败返回-1
int socket(int domain, int type, int protocol);
```

`socket()` 函数的调用会生成套接字，并返回套接字的文件描述符，后面经过一系列操作最终实现 socket 套接字的成功连接后，就可以像实现本地文件 I/O 操作一样进行网络数据的写入和读取了。

```c++
#include <sys/socket.h>
//成功返回0，失败返回-1
int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
```

调用 `bind()` 函数会为创建好的 socket 套接字分配网络 IP 地址和本地接听端口（当本地计算机存在多张网卡时尤其重要，因为本机会有多个 IP 地址），初始化好该 socket 套接字所属的 IP 地址和端口号后，这个 socket 套接字就相当于与这个固定的网络通信位置相绑定了。

```c++
#include <sys/socket.h>
//成功返回0，失败返回-1
int listen(int sockfd, int backlog);
```

前面完成对 socket 套接字的网络 IP 地址和端口绑定后，通过调用 `listen()` 函数就使得这个 socket 套接字进入等待接听状态，第二个参数设置同一时间运行等待接听的对方机器数量（在未成功建立连接前，对方处于这个 `listen` 等待队列中并不会回复失败，等待快速的建立连接操作）。

```c++
#include <sys/socket.h>
//成功返回文件描述符，失败返回-1
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

在进入等待接听后，调用 `accept()` 函数，正常情况下程序会进入阻塞状态，等待有连接请求到达这个 socket 绑定的网络地址端口上，一旦接收到对方请求会就会返回一个建立正常数据通信的文件描述符供双方使用，并且会将对方的 IP 地址及端口信息使用指针传参的方式传递回来。

简单概括以上的网络套接字创建过程如下（ TCP 方式）：

1. 调用 `socket()` 函数创建 socket 套接字。
2. 调用 `bind()` 函数初始化套接字的 IP 地址和端口号。
3. 调用 `listen()` 函数将 socket 套接字转为可接收请求状态。
4. 调用 `accept()` 函数接收连接请求，并返回能够正常进行数据通信的文件描述符。

下面通过一个简单示例来演示一个最简单的 HELLO TCP 服务器，该服务器接收到连接请求后会主动向请求客户端返回 `hello world` 的消息进行答复。

具体操作如下：

1. 首先在 VS Code 平台中新建文件 `hello_server.cpp` 。点击 VS Code 左侧的文件控件，然后在左侧 `PROJECT` 窗口中点击 `新建文件` 按钮，输入新建文件名称 `hello_server.cpp` 具体如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/55dfa255b7dfe552ac359d84ac4f701d-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/7415ef082eb16fb27a2893a9e86337cd-0)

在完成新建文件操作后，就可以在 VS Code 窗口右侧中添加代码，具体添加如下代码：

```C++
#include <iostream>
#include <string>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/socket.h>

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    int serv_sock;
    int client_sock;

    struct sockaddr_in serv_addr;
    struct sockaddr_in client_addr;

    socklen_t client_addr_size;
    char message[] = "hello world";

    //判断参数数量，Usage: <port>， 需要在命令行输入服务器接收消息的端口号
    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << " <prot>" << std::endl;
        exit(1);
    }

    //创建socket 套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //初始化套接字结构体
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);//选择当前任意网卡
    serv_addr.sin_port = htons(atoi(argv[1]));//设置接收消息的端口号

    //绑定端口
    if(bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0)
    {
        error_handle("bind", "bind() error.");
    }

    //监听端口，设置等待队列数量为5
    if(listen(serv_sock, 5) < 0)
    {
        error_handle("listen", "listen() error.");
    }

    //打印输出等待连接
    std::cout << "Waiting Client...." << std::endl;

    client_addr_size = sizeof(client_addr);
    //等待接收客户端建立连接
    client_sock = accept(serv_sock, (struct sockaddr*)&client_sock, &client_addr_size);
    if(client_sock < 0)
    {
        error_handle("accept", "accept() error.");
    }
    //accept() 成功建立连接后，服务器就会得到客户端的 IP 地址和端口号。
    //打印客户端 IP 和端口号
    std::cout << "Client IP : " << inet_ntoa(client_addr.sin_addr) << " , port : " << ntohs(client_addr.sin_port) << std::endl;

    //向客户端发送 "hello world" 消息，使用write 标准IO接口就可以
    write(client_sock, message, sizeof(message));

    //关闭TCP连接
    close(client_sock);
    //关闭socket套接字
    close(serv_sock);

    return 0;
}
```

1. 在 VS Code 的终端窗口中执行终端命令进行代码的编译。

通常在打开的 VS Code 环境下就会直接有打开好的终端窗口，如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/10f717c4c772f600e39256b10f06c452-0)

如果不小心关闭了打开的终端窗口，只需要通过 VS Code 左侧的菜单中找到终端窗口再次打开即可，如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/e8879a62ab3d2bb952983297174edab6-0)

在终端窗口中输入下列命令完成服务器代码的编译。

```bash
g++ hello_server.cpp -o hello_server
```

完成如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/a7a2718dc71afcec4820ae33ec3ecab8-0)

在完成服务器端的程序构建后，我们需要也创建一个与之对应的客户端来进行 socket 连接请求并能够读取服务器返回的消息数据，这里概括网络套接字请求连接的过程如下（ TCP 方式）：

1. 调用 `socket()` 函数创建 socket 套接字。
2. 作为请求连接的客户端，这里不需要进行 `bind()` 和 `listen()` 操作，只需要设置服务器对方的 IP 地址和端口号。
3. 调用 `connect()` 函数发送 socket 连接请求，并返回能够正常进行数据通信的文件描述符。

参照这样的过程，编写客户端的代码并构建客户端程序，操作如下：

1. 在 VS Code 平台中新建文件 `hello_client.cpp` ，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/4dd7ea1908b632c7dc5ea09b62836432-0)

在 `hello_client.cpp` 源文件中添加如下代码：

```C++
#include <iostream>
#include <string>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/socket.h>


//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    int sock;//socket套接字
    struct sockaddr_in serv_addr;//服务器套接字结构体
    char message[64];//用于接收服务器消息的缓冲区
    int str_len;

    //判断当前参数数量，需要命令行参数 <IP> <port>
    if(argc < 3)
    {
        std::cout << "Usage : " << argv[0] << " <IP> <port>" << std::endl;
        exit(1);
    }

    //创建socket 套接字
    sock = socket(PF_INET, SOCK_STREAM, 0);
    if(sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //初始化服务器套接字结构体参数，设置对方的IP地址和端口号
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_addr.sin_port = htons(atoi(argv[2]));

    //与服务器建立连接
    if(connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0)
    {
        error_handle("connect", "connect() error.");
    }

    //读取服务器发送来的消息
    str_len = read(sock, message, sizeof(message) -1);
    if(str_len < 0)
    {
        //read() 读取数据失败
        error_handle("read", "read() error.");
    }

    //将读取到的输出打印出来
    std::cout << "Recv Message : " << message << std::endl;

    //关闭socket 套接字
    close(sock);
    return 0;

}
```

1. 在 VS Code 的终端窗口中执行如下命令完成客户端代码的编译。

```bash
g++ hello_client.cpp -o hello_client
```

具体如下图所示；

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/6c58be954d49c828200984dc0f946e1e-0)

1. 演示客户端与服务端进行网络通信，通过 VS Code 的终端窗口运行程序进行测试：

- 启动服务器，指定接收消息端口号为 18888。在终端窗口中输入如下命令：

```bash
./hello_server 18888
```

如图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/756068eda418f360c07d61abe86b65b3-0)

- 使用另一个终端窗口启动客户端，指定服务器 IP 地址和端口号（`127.0.0.1` 表示本地环回地址，`18888` 指前面服务器启动指定端口号）

```bash
./hello_client 127.0.0.1 18888
```

新建终端窗口，只需要点击原来终端窗口上的 `+` 号键，然后中间就会显示 `2:zsh` 表示第二个终端：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/b7abf170f69b19b73e403edeaa731928-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/166509081b8719d6c3e7a60fa1b7ddee-0)

## 3 Linux 下的文件 I/O 操作

要充分理解 Linux 平台下的网络编程相关知识，还需要对 Linux 平台下的文件操作有一定掌握。Linux 系统的“一切皆文件”理念要有一定体会。

接下来，就简单了解和掌握一下 Linux 下的文件基本操作（这里仅讲述 Linux 系统提供的文件操作函数）。虽然对于 C++ 来说有 `stream` 这个 I/O 类来处理相关问题，但是这里要讲的文件操作是基于 Linux 系统的，也就是像 C/C++ 或者其它一些高级语言中对于文件的操作其实都是基于下面的函数接口做出的封装实现，所以大家一定要对此有一定的理解。

### 文件描述符

文件描述符其实就是 Linux 系统分配给文件或者套接字的整数。其实对于标准输入输出以及标准错误输出，在 Linux 下也都有分配，具体如下：

| 文件描述符 | 对象                      |
| ---------- | ------------------------- |
| 0          | 标准输入：standard input  |
| 1          | 标准输出：standard output |
| 2          | 标准错误：standard Error  |

在 Linux 系统中，网络套接字操作其实也是一种文件操作，所以这里讲到的关于文件操作的相关概念，对网络套接字来说也是一样的。

文件描述符只有在文件或套接字经过创建过程后才会被分配。前面说的标准输入输出和标准错误输出文件描述符都是在进程启动后系统自动分配的，不需要自己创建就能够使用的。

### 文件操作相关 API

接下来分别介绍对于文件操作的相关 API：

1. 打开文件

```C++
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
//成功返回文件描述符，失败返回-1
int open(const char *path, int flag);
```

在 `open()` 函数中，第一个参数为打开文件的路径。第二个参数为文件打开模式，如果同时需要多个模式则使用按位或允许符组合即可，具体如下：

| 打开模式 | 含义                                 |
| -------- | ------------------------------------ |
| O_CREAT  | 必要时创建文件                       |
| O_TRUNC  | 删除全部现有数据                     |
| O_APPEND | 维持文件现有数据，保存到当前数据最后 |
| O_RDONLY | 只读方式打开                         |
| O_WRONLY | 只写方式打开                         |
| O_RDWR   | 读写方式打开                         |

1. 关闭文件

```C++
#include <unistd.h>
//成功返回0， 失败返回-1
int close(int fd);
```

调用 `close()` 函数将关闭（终止）对应的文件操作，该函数在网络套接字操作中是相同函数，由此证明套接字和文件是一样的。

1. 向文件写入数据

```C++
#include <unistd.h>
//成功返回写入的字节数，失败返回-1
ssize_t write(int fd, const void *buf, size_t nbytes);
```

在文件或者套接字操作中，通过 `write()` 函数能够向文件（套接字）中写入数据。（前面的网络编程示例也用到了这一函数）

1. 读取文件数据

```C++
#include <unistd.h>
//成功返回接收的字节数（遇到文件结尾返回0），失败返回-1
ssize_t read(int fd, void *buf, size_t nbytes);
```

同 `write()` 函数类似，`read()` 函数调用能够从文件（套接字）中读取数据，前面的例程中也用到了，相信大家也有所体会。

### Linux 文件操作示例

接下来完成一个示例，使得对 Linux 下文件操作更加清楚：

创建一个文件，创建模式、读写模式、清空数据模式的方式打开，向其中输入 `hello` 字符串后，然后从标准输入获取当前用户的名字后，将新获得的用户名字和 `welcome` 语句写入文件中，然后关闭文件。再次以只读方式打开文件，读取文件中全部数据打印输出到标准输出中，最后关闭文件结束操作。

跟前面的方式相同，在 VS Code 平台中新建示例文件 `filedemo.cpp`，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/b849a5426b7da483f7d3a9dad4412a2a-0)

在新建的 `filedemo.cpp` 文件中添加如下代码：

```C++
#include <iostream>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    char Hbuf[] = "Hello,";
    char Wbuf[] = ", Welcome to learn network programming!!!";
    char message[128] = {0};
    char recv_buf[64] = {0};
    int fd;
    ssize_t write_size, read_size;

    if(argc < 2 )
    {
        std::cout << "Usage : " << argv[0] << " [filename] " << std::endl;
        exit(1);
    }

    fd = open(argv[1], O_CREAT | O_TRUNC | O_RDWR);
    if(fd < 0)
    {
        error_handle("open", "open() error.");
    }

    //写入 hello
    write_size = write(fd, Hbuf, strlen(Hbuf));
    if(write_size < 0)
    {
        error_handle("write", "write() Hbuf error.");
    }

    //读取用户输入信息
    std::cout << " 请输入用户名："  << std::endl;
    std::cin >> recv_buf;

    //写入用户名和 welcome 语句
    write_size = write(fd, recv_buf, strlen(recv_buf));
    if(write_size < 0)
    {
        error_handle("write", "write() recv_buf error.");
    }
    write_size = write(fd, Wbuf, strlen(Wbuf));
    if(write_size < 0)
    {
        error_handle("write", "write() Wbuf error.");
    }

    //关闭文件
    close(fd);

    //再次打开文件，只读模式打开
    fd = open(argv[1], O_RDONLY);
    if(fd < 0)
    {
        error_handle("open", "open() error.");
    }

    //读取文件数据，文件内容已知不足 128字节，所以使用 sizeof(message) 为长度读取能够全部将数据读取出来
    read_size = read(fd, message, sizeof(message));
    if(read_size < 0)
    {
        error_handle("read", "read() message error.");
    }

    //将读取数据打印输出
    std::cout << " 从文件中读取到 " << read_size << " 个字节的数据." << std::endl;
    std::cout << message << std::endl;

    //最后关闭文件
    close(fd);

    return 0;
}
```

添加完成代码后，在 VS Code 平台中的终端窗口输入如下操作命令完成代码编译：

```bash
g++ filedemo.cpp -o filedemo
```

最终执行效果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/b6b63f80e187698587308604913d440d-0)

## 4 实验总结

通过本节实验的学习，能够对 Linux 下网络编程有一定概念，并且通过完成几个简单示例程序，能对相关的 API 函数操作有一定掌握，相信经过本小节的学习后，能够对后面的知识的掌握提供一个很好地基础。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code1.zip
```

# 二 基于 UDP 的服务器和客户端实现

## 1 实验介绍

本章实验内容主要介绍讲解关于 UDP 网络传输协议。通过前一小节的实验内容已经基本对网络编程的主要功能和作用有了一定的理解，本章将先从最简单的 UDP 网络传输方式开始，通过简单的代码实现基本的服务器和客户端例程使得大家能够从代码的具体使用上理解和掌握 Linux 网络编程中的 UDP 通信协议部分。

#### 知识点

- UDP 网络通信协议
- UDP 协议的数据传输特性和 connect 函数解析
- 实现基于 UDP 的服务器实例程序
- 实现基于 UDP 的客户端实例程序

## 2 UDP网络通信协议

前一节讲到了 OSI 七层网络模型，其中提到了传输层，而传输层中的主要协议就分为 TCP 协议和 UDP 协议两种，这两种协议方式都是用来做数据交换的，在操作系统上分别是通过 TCP 套接字完成的 TCP 方式和通过 UDP 套接字完成的 UDP 方式。

前面已经基本讲过关于网络地址和端口的相关概念了，UDP 传输方式其实就是指定了目标 IP 地址和端口后，将数据直接发送给对方而不需要通过调用 `accept()` 和 `connect()` 进行双方建立连接的操作，简单来说就是数据点对点传输，发送方只管发送，接收方只管接受数据，双方除了对同样的数据有接触以外并没有发生太多的关系。有下图作为参考。后面会有详细讲到 TCP 通信作为对比能够更好理解。

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/d0eae74fe4f3b2b106d2ab24ff7e7452-0)

从图中的示例可以理解到，UDP 传输方式其实和现实生活中的信件邮寄方式很像。发送数据方将数据标好接收者的 IP 地址和端口号后就将信件（数据）塞到邮箱里（网络中）了，而接收信件（接收方）需要守在固定的邮箱中等待邮递员（网络）将数据送过来，一旦发现邮箱中（UDP 套接字）中有信件（数据）到达，那么就会取出信件（数据）进行读取。同时，在信件（数据）上也会存在发送者的相关信息（ IP 地址和端口号）以方便接收者在收到数据后能够再次进行数据地回复（利用 UDP 方式再次发送数据）。

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/3256445a9fe38bd16f769623f5733c83-0)

由此能够看出，在 UDP 传输方式中，如果将接收数据的一方称为服务端，发送数据一方称为客户端的话，那么在第一次数据传输完成后，接收数据一方回复对方数据时，双方的身份就发生了互换。（当然可以将率先 `bind()` 端口的一方固定看作服务端也是没问题的，在开始客户端发送数据时是随机使用端口号进行发送的，但是开始使用 UDP 套接字后，随机分配的端口号在这个过程中是保持不变的。）

## 3 UDP 协议的数据传输特性和 connect 函数解析

通过前面的举例能够简单描述了 UDP 传输方式的过程，那么与 TCP 传输方式比较，UDP 传输方式又有哪些不同呢，下面这个表格列出了其中的几方面差别。

|              | UDP                                        | TCP                                    |
| ------------ | ------------------------------------------ | -------------------------------------- |
| 是否连接     | 无连接                                     | 面向连接                               |
| 是否可靠     | 不可靠传输，不使用流量控制和拥塞控制       | 可靠传输，使用流量控制和拥塞控制       |
| 连接对象个数 | 支持一对一，一对多，多对一和多对多交互通信 | 仅支持一对一通信                       |
| 传输方式     | 面向报文                                   | 面向字节流                             |
| 首部开销     | 首部开销小，仅 8 字节                      | 首部最小 20 字节，最大 60 字节         |
| 适用场景     | 适用于实时应用，视频、直播等               | 适用于要求可靠传输的应用，文件、数据等 |

从表格中能够看出，如果只考虑可靠性的话，那么 TCP 比 UDP 好太多了。但是在使用过程中，UDP 比 TCP 简洁很多，UDP 在传输中不需要 ACK 等应答信号和 SEQ 数据包序号等等，每一次的数据传输都是相对独立的。因此，UDP 在数据传输效率上要远高于 TCP 传输方式。

实际上，UDP 传输方式之所以实现方式和效率都要优于 TCP ，就是因为缺少了 TCP 那样出于可靠数据传输考虑的数据流控制机制。这就是 TCP 方式与 UDP 方式最本质的不同点。

### UDP 内部工作原理

前面已经描述过，在使用 UDP 传输的方式中，只需要将 UDP 套接字标记好接收方的 IP 地址和端口号后，就能够通过 UDP 传输方式将数据发送给对方。具体如前一小节的信件传输示意图类似。

从图中能够看到，IP 协议的作用就是能够将数据根据标记的 IP 地址准确地找到并发给这个 IP 地址的主机，而这之后的操作就需要通过 UDP 套接字来完成，根据标记好的接收方的端口号，UDP 套接字最终将这个数据包准确且完整地送到该端口的 UDP 套接字中去。

### UDP 的应用场景分析

虽然大部分的网络编程都是基于 TCP 实现的，这其实是出于数据可靠性和安全性的考虑，但是也有很多应用是基于 UDP 实现的。尤其是当应用场景对安全性、可靠性要求并不高，但是对传输效率和传输频率有很高要求时，UDP 传输方式就有很大的优势了。

比如网络传输文件、压缩包的时候都会采用 TCP 传输方式，因为要确保数据传输的不丢失。而在网络实时传输视频或音频的时候，数据丢失影响就不是很大，速度是非常重要的因素，这种情况下 UDP 传输方式就是非常好的选择。

## 4 实现基于 UDP 的服务器端和客户端需要注意的事项

通过前面描述 UDP 传输协议的相关概念和特性后，接下来就需要介绍如何通过编程实现使用 UDP 传输方式来进行网络数据的传输。

### UDP 不存在建立连接

前面其实就有提到过，UDP 方式与 TCP 方式在使用过程中最大的不同，就是 UDP 不需要建立连接，也就是说在服务端不需要调用 `listen()` 和 `accept()` 等待客户端请求连接，客户端也不需要调用 `connect()` 去发送建立连接请求。

UDP 传输方式只需要创建套接字后，就能够通过 IP 地址和端口号实现数据的发送和接受。

### UDP 传输双方均只需要创建一个套接字

TCP 方式中，服务端首先创建 socket 套接字用于端口的监听和等待连接请求，在正式建立连接后会返回连接成功后的套接字，而之前的套接字继续作为端口的监听和等待连接。

UDP 方式既然不需要去建立连接，自然也不需要一个用于监听和等待连接请求的套接字存在，所以在使用 UDP 方式时，不论客户端还是服务端均都是只有一个 socket 套接字同时用于接收数据和发送数据使用。

## 5 实现基于 UDP 的服务器实例程序

通过前面关于 UDP 传输方式的详细讲述，大家现在应该对这种传输方式有一定程度上的理解了，接下来就让我们用代码的方式去使用 UDP 传输方式完成一个简单的回声服务器实现（服务器接收到消息后，将消息打印输出并再次将该消息数据返回给客户端）。

### UdpServer.cpp

在 VS Code 平台下，点击左侧新建文件，创建 `UdpServer.cpp` 源文件作为 UDP 服务器的源代码文件，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/217a1e02059e2064667bd5b7e4b865cf-0)

在其中添加如下代码，这段代码完成了最简化的 UDP 服务器创建和基本数据处理操作（具体可以查看其中的注释信息加以理解）。

```C++
#include <iostream>
#include <string>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_SIZE 128

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    int serv_sock;
    char message[BUF_SIZE];
    int str_len;
    socklen_t client_adr_sz;
    struct sockaddr_in serv_adr, client_adr;

    //因为服务器程序使用时通过命令行参数指定接收消息的端口号，
    //所以当参数数量少于需求 2 时，程序结束并报错
    if(argc != 2){
        std::cout << "Usage : " << argv[0] << " <port>" << std::endl;
        exit(0);
    }

    //创建socket 套接字，sock 用于后面发送和接收数据
    serv_sock = socket(PF_INET, SOCK_DGRAM, 0);
    if(serv_sock == -1)
    {
        error_handle("socket", "UDP socket creation error.");
    }

    //初始化套接字结构体，初始化服务器套接字结构体中的 IP 地址和端口号
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    //将socket套接字绑定固定端口进行消息接收
    if(bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr)) == -1)
    {
        error_handle("bind", "bind() error.");
    }

    //UDP 方式不需要进行 listen 和 accept 操作，直接就可以通过 recvfrom 函数去接收套接字中收到的数据。
    while(1)
    {
        //清空接收消息数据的缓存区
        memset(message, 0, BUF_SIZE);
        //计算客户端结构体大小，用于接收客户端数据结构
        client_adr_sz = sizeof(client_adr);
        //recvfrom() 调用不仅收到对方发送来的数据，还通过最后两个参数返回了对方的 IP 地址和端口号信息，用于返回数据使用。
        str_len = recvfrom(serv_sock, message, BUF_SIZE, 0, (struct sockaddr*)&client_adr, &client_adr_sz);
        //将收到的信息打印出来
        std::cout << "Recv Message : " << message << std::endl;
        //通过recvfrom 返回的发送方的IP地址和端口号信息，使用 sendto 将收到的信息再次发送回去。
        sendto(serv_sock, message, str_len, 0, (struct sockaddr*)&client_adr, client_adr_sz);
    }

    //结束程序后关闭socket套接字
    close(serv_sock);

    return 0;
}
```

编写完成代码后，在 VS Code 平台下的终端窗口中输入如下命令完成源文件的编译并构建 UDP 可执行文件 `UdpServer`

```bash
g++ UdpServer.cpp -o UdpServer
```

如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/c427c7ad8eedd8a988cf38dcd01880de-0)

如果编译存在错误，根据编译器报错信息进行修改后完成可执行程序的编译。



### UdpClient.cpp

在完成 UDP 服务器实例程序的创建后，就需要创建与之进行数据交互的客户端程序实例。

在 VS Code 平台下，新建文件 `UdpClient.cpp` 作为 UDP 客户端实例程序的源代码文件，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/1f1c8b70007c3e0676b1e8df2752d80b-0)

在其中添加如下代码实现 UDP 客户端数据的发送和接收，具体可以通过阅读其中的注释信息加以理解。

```C++
#include <iostream>
#include <string>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_SIZE 128

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    int sock;
    char message[BUF_SIZE];
    int str_len;
    socklen_t adr_sz;
    struct sockaddr_in serv_adr, from_adr;

    //因为客户端程序使用时通过命令行参数指定IP地址和端口号，
    //所以当参数数量少于需求 3 时，程序结束并报错
    if(argc < 3)
    {
        std::cout << "Usage : " << argv[0] << " <IP> <port>" << std::endl;
        exit(0);
    }

    //创建socket 套接字，sock 用于后面发送和接收数据
    sock = socket(PF_INET, SOCK_DGRAM, 0);
    if(sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //初始化套接字结构体，初始化服务器套接字结构体中的 IP 地址和端口号
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_adr.sin_port = htons(atoi(argv[2]));

    while(1)
    {
        //从标准输入获取发送数据，保存到message缓存区
        std::cin >> message;
        //判断如果输入是 'q' 或者 'Q' 则表示退出客户端程序
        if(!strcmp(message, "q\n") || !strcmp(message, "Q\n"))
        {
            //退出
            break;
        }

        //发送消息
        sendto(sock, message, strlen(message), 0, (struct sockaddr*)&serv_adr, sizeof(serv_adr));

        //清空接收数组
        memset(message, 0, BUF_SIZE);
        //计算返回套接字结构体字节长度
        adr_sz = sizeof(from_adr);
        //接收UDP返回数据
        str_len = recvfrom(sock, message, BUF_SIZE, 0, (struct sockaddr*)&from_adr, &adr_sz);
        //在接收数据尾部添加'0'结束符
        message[str_len + 1] = 0;
        //将接收数据打印输出
        std::cout << "Recv Message : " << message << std::endl;

    }
}
```

跟构建服务端实例程序相同，在 VS Code 平台中的终端窗口输入下面命令完成客户端可执行程序的编译构建：

```bash
g++ UdpClient.cpp -o UdpClient
```

如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/7b6dfda1768af16db5a5a89485d8a0e0-0)

## 6 程序运行实现

前面完成了 UDP 服务器实例程序和 UDP 客户端实例程序的创建后，在这里就可以进行运行测试了。本文中的实例程序测试同一通过 VS Code 终端窗口实现和显示。

鼠标左键点击 VS Code 终端窗口上的 `+` 符号按键，新建一个窗口出来。一个窗口用于启动服务器，另一个窗口用于启动运行客户端的方式来进行测试验证。新建终端方式如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/9a4c566146592e896966796c4c718444-0)

在服务端测试终端窗口，输入以下操作命令：

```bash
./UdpServer 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/5347831a78523461964ce9d23f3ce271-0)

鼠标点击切换到另一个窗口，输入如下命令启动 UDP 客户端程序：

```bash
./UdpClient
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/99d5f32c168c3c21d0e176daed19246b-0)

此时，UDP 客户端程序终端窗口就会进入读取终端信息输入的步骤，随意输入消息，输入消息就会被发送到 UDP 服务器程序中（在服务器终端中会出现信息打印），并且 UDP 服务器程序会将每次接收到的数据再返送回来，这样客户端程序也会打印输出一次。效果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/44452fbaf16c4ccefcc7533547b45d39-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/78ed195d0413f037c921cf378e8d3e73-0)

## 7 实验总结

通过本章节的学习和练习，能够对网络编程中的 UDP 传输方式有一定的理解和掌握。UDP 方式与 TCP 传输方式最大的不同就在于缺少流控制需求，也正是因为缺少流控制，UDP 传输方式不需要在传输过程中建立通信连接以及数据 ACK 消息回复和数据包顺序传输等操作，这样大大提高了数据传输的效率。所以，与 TCP 传输方式相比，UDP 在缺少可靠性的同时大大增加的数据传输效率。

通过本章节最后的程序实例实现，也从代码层面了解到了在使用 UDP 传输方式时，服务器端和客户端的基本操作简单明了。当以后续真正运用到 UDP 网络传输方式在代码的开发过程中，相信大家也能够轻松应对了。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code2.zip
```

# 三 UDP 的多播与广播

## 1 实验介绍

通过前一小节的知识，我们了解到 UDP 网络传输协议的相关概念及特点。除了在数据传输过程中不需要建立连接以外，在本章的知识中还能学习到 UDP 传输方式特有的传输方式，那就是多播方式及广播方式，当面对多个数据接收者时，UDP 传输方式就是有这样的可选方案去简单实现数据的一次放松，多个接收者同时都能够接收到数据的。

下面，我们就来一步一步深入了解网络编程中的 UDP 编程吧。

#### 知识点

- UDP 的多播模式
- UDP 的广播模式

## 2 UDP 的多播模式

多播方式的数据传输其实就是基于 UDP 传输方式完成的。所以使用多播方式进行数据传输的实现方式也和 UDP 方式实现几乎一样。不一样的地方就是前面讲解的 UDP 数据传输实现是传输给单一目标进行的，而多播方式是将多个接收方加入（注册）到特定组中，通过向组中的所有主机进行数据传输实现多播方式。

多播数据传输方式的优点：

- 多播服务器端针对特定多播组，每次仅仅需要发送一次数据既可保证向所有组成员完成数据发送；
- 即使只发送一次数据，组内所有的客户端也都会接收到数据；
- 多播组数能够在 IP 地址范围内随意增加；
- 加入特定组后，就能接收所有发往该组中的数据；

多播方式的传输流程如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/64c7065d5e56a1b9e1eff8fcaceb02f9-0)

通过上图中的描述，能够看出，多播方式的实现主要依赖于路由器的帮助。也就是说虽然数据仅执行发送了一次，但是到达该组的路由器的时候，由路由器完成了数据的复制和多次发送操作。而接收者其实是通过加入到路由器的组中完成了特定组的加入。这样的操作其实是为了尽可能使得相同数据在网络中所占用的资源最小。

### TTL(生存模式)

前面说到了真正完成数据的多次复制发送的其实是路由器，但是为了使多播数据尽可能少地影响网络环境，在具体实现使必须要设置 TTL （生存时间），TTL 是 Time to Live 的简写，该参数是决定“数据包传输距离”的主要因素。TTL 参数是由整数表示，当传输过程中国没经过一个路由器时，TTL 数值都会减一，当 TTL 为 0 时，该数据包就不会继续传递下去，只能被销毁掉。所以，TTL 的值如果设置太大会导致数据影响网络范围过多，而太小则会导致传递不到需要传送的目标位置。具体如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/d7191753bf5cda503ad8a5d010f658a2-0)

TTL 设置方法其实是通过 socket 套接字的可选项完成的。设置 TTL 相关的协议层是 `IPPROTO_IP` ，选项名为 `IP_MULTICAST_TTL`，所以可以用如下代码将 TTL 值设置为 64 ：

```C++
    int send_sock;
    int time_live = 64;

    ...
    send_sock = socket(PF_INET, SOCK_DGRAM, 0);
    setsockopt(send_sock, IPPROTO_IP, IP_MULTICAST_TTL, (void*)&time_live, sizeof(time_live));
    ...
```

### 加入多播组

作为多播数据的接收者，如果要想正常接收到多播的数据，需要提前加入到多播组中。加入多播组的设置也是通过设置套接字可选项完成，加入多播组相关的协议层为 `IPPROTO_IP` ，选项名为 `IP_ADD_MEMBERSHIP`，可以通过下面代码使当前 socket 套接字加入多播组：

```C++
int recv_sock;
struct ip_mreq join_adr;
...
recv_sock = socket(PF_INET, SOCK_DGRAM, 0);
...
join_adr.imr_multiaddr.s_addr = "多播组地址信息";
join_adr.imr_interface.s_addr = "加入多播组的主机地址信息";
setsockopt(recv_sock, IPPROTO_IP, IP_ADD_MEMBERSHIP, (void*)&join_adr, sizeof(join_adr));
···
```

## 3 UDP 多播模式代码实现

接下来就来具体用代码实现多播数据传输的实例程序。在多播方式中不再简单使用服务器端和客户端来描述数据传输双方，而是用“发送者”和“接收者”，也就是 sender 和 recviver 来指代双方。

- sender ：指向多播组中发送指定文件中保存的文本信息；
- recviver ：指接收多播组中数据的接收者；

### sender 发送者实例程序实现

在 VS Code 平台下创建新的源代码文件 `sender.cpp` ，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/8dd3a4b589f3f36398f152197608956f-0)

在 `sender.cpp` 文件中添加如下源代码：

```C++
#include <iostream>
#include <fstream>
#include <string>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define TTL     64
#define BUFFSIZE    64

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    //声明变量
    int send_sock;
    struct sockaddr_in mul_adr;
    int time_live = TTL;
    FILE *fp;
    char buf[BUFFSIZE];

    if(argc < 3)
    {
        std::cout << "Usage : " << argv[0] << "Group IP <Port>" << std::endl;
        exit(1);
    }

    //创建 UDP 套接字
    send_sock = socket(PF_INET, SOCK_DGRAM, 0);

    //套接字结构体初始化
    memset(&mul_adr, 0, sizeof(mul_adr));
    mul_adr.sin_family = AF_INET;
    mul_adr.sin_addr.s_addr = inet_addr(argv[1]);
    mul_adr.sin_port = htons(atoi(argv[2]));

    //设置 socketopt 选项为多播模式
    setsockopt(send_sock, IPPROTO_IP, IP_MULTICAST_TTL, (void*)&time_live, sizeof(time_live));

    //以文本模式打开in.txt备读
    std::ifstream srcFile("news.t", std::ios::in);
    if(!srcFile) { //打开失败
        error_handle("ifstream", "ifstream error.");
    }

    //可以像用cin那样用ifstream对象
    memset(buf, 0, BUFFSIZE);
    while(srcFile >> buf)
    {
        sendto(send_sock, buf, strlen(buf), 0, (struct sockaddr*)&mul_adr, sizeof(mul_adr));
        memset(buf, 0, BUFFSIZE);
    }


    //关闭套接字
    close(send_sock);

    std::cout << "数据发送完毕！" << std::endl;
    return 0;
}
```

完成代码编写后，在 VS Code 中的终端窗口输入如下终端命令完成代码的编译和可执行程序的构建：

```bash
g++ sender.cpp -o sender
```

如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/c1ecb5991c0871e5d9358fde585940e7-0)

### receiver 接收者实例程序实现

在 VS Code 平台下创建新的源代码文件 `receiver.cpp` ，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/e93bef570e52269f6e822cf356fe795e-0)

在 `receiver.cpp` 文件中添加如下源代码：

```C++
#include <iostream>
#include <fstream>
#include <string>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#define BUFFSIZE    64

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    //声明变量
    int recv_sock;
    struct sockaddr_in adr;
    struct ip_mreq join_adr;
    int str_len;
    char buf[BUFFSIZE];

    //判断当前命令行参数是否有效
    if(argc < 3)
    {
        std::cout << "Usage : " << argv[0] << "Group IP <PORT>" << std::endl;
        exit(1);
    }

    //创建 UDP socket套接字
    recv_sock = socket(PF_INET, SOCK_DGRAM, 0);

    //初始化UDP 套接字及端口号
    memset(&adr, 0, sizeof(adr));
    adr.sin_family = AF_INET;
    adr.sin_addr.s_addr = htonl(INADDR_ANY);
    adr.sin_port = htons(atoi(argv[2]));

    //绑定接收数据的 UDP 端口
    if(bind(recv_sock, (struct sockaddr*)&adr, sizeof(adr)) == 1)
    {
        error_handle("bind", "bind() error.");
    }

    join_adr.imr_multiaddr.s_addr = inet_addr(argv[1]);
    join_adr.imr_interface.s_addr = htonl(INADDR_ANY);

    setsockopt(recv_sock, IPPROTO_IP, IP_ADD_MEMBERSHIP, (void*)&join_adr, sizeof(join_adr));

    //循环接收多播数据并打印输出
    while(1)
    {
        memset(buf, 0, sizeof(buf));
        str_len = recvfrom(recv_sock, buf, BUFFSIZE -1, 0, NULL, 0);
        if(str_len < 0)
        {
            break;
        }
        buf[str_len] = 0;
        std::cout << buf << std::endl;
    }

    close(recv_sock);
    return 0;
}
```

完成代码编写后，在 VS Code 中的终端窗口输入如下终端命令完成代码的编译和可执行程序的构建：

```bash
g++ receiver.cpp -o receiver
```

编译显示如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/c557ef0d624297023f708c899d443d77-0)

### 测试验证

在 VS Code 窗口中新建两个终端窗口，通过两个终端窗口运行不同的接收者程序 `receiver` ，另一个终端窗口中运行发送者程序 `sender` 完成多播实例效果测试验证。

1. 创建测试传输文件 `news.t` ，创建文件步骤类似之前创建源文件的方式，在新建的数据文件中随意添加用于测试传输的文本数据，具体如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/a1e368cf118ae42c7fcbc8b7303edeb3-0)

1. 使用一个终端窗口启动 `receiver` 接收者程序，命令为：

```bash
./receiver 127.0.0.1 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/f78885025a0502b9c1556bc89a6547ea-0)

1. 在 VS Code 终端窗口中点击 `+` 按键新建一个终端窗口，在其中使用命令启动 `sender` 发送者程序：

```bash
./sender 127.0.0.1 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/beb6c9fa614ccf6e73b2b584d31208e2-0)

1. 在本机演示的情况下，测试后接收者程序将接收到数据并打印出来。（由于本机只有一个用于接收消息的端口，所以本机演示只能显示一个接收者接收数据，实际应用过程中只要加入多播组并存在多播范围内就能接收到发送者发出的数据）

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/beb6c9fa614ccf6e73b2b584d31208e2-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/8fdba944b9fd0da5ea9388e1157a813f-0)

## 4 UDP的广播模式

广播也是一种“一次性向多个主机发送数据”的发送方式，跟前面介绍的多播类似，区别是广播的数据传输范围是一个指定网络中的全部主机，而多播方式中只要接收者加入了多播组就能收到数据。

广播方式和多播方式一样，也是基于 UDP 完成的，并且根据传输数据时使用的 IP 地址形式不同，广播分为直接广播和本地广播两种。这两种方式的区别主语是 IP 地址不同。

- 直接广播的 IP 地址中除了网络地址，其余地址全部设置为 1 。
- 本地广播中使用的 IP 地址限定为 `255.255.255.255` 。所以数据将发生给本机所在局域网络中的所有主机。

在具体实现中，广播方式的发送者需要调用 `setsockopt()` 函数将 `SO_BROADCAST` 选项设置为 1 。而接收者在代码实现上与普通的 UDP 服务器类似吗，只需要监听指定 UDP 端口等待数据即可，不需要像多播方式一样去加入多播组。

## 5 UDP 广播模式代码实现

接下来，将具体通过编写代码实现一个简单的广播例程，跟多播方式一样，也是需要创建发送者和接收者。（由于是本机演示，所以广播方式同时间也只能有一个接收者运行接收数据，因为要绑定监听指定的 UDP 端口。）

### 广播方式发送者代码实现

1. 在 VS Code 平台中新建广播方式发送者文件 `news_sender_brd.cpp` ，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/5fb39fca5a29f1c350575901032cf217-0)

1. 在 `news_sender_brd.cpp` 源文件中添加如下代码：

```C++
#include <iostream>
#include <fstream>
#include <string>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUFFSIZE    64

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    //声明变量
    int send_sock;
    struct sockaddr_in broad_adr;
    int str_len;
    char buf[BUFFSIZE];
    FILE *fp;
    int so_brd = 1; //初始化 SO_BROADCAST 选项为 1

    if(argc < 3)
    {
        std::cout << "Usage : " << argv[0] << "Broadcast IP <Port>" << std::endl;
        exit(1);
    }

    //创建 UDP 套接字
    send_sock = socket(PF_INET, SOCK_DGRAM, 0);

    //套接字结构体初始化
    memset(&broad_adr, 0, sizeof(broad_adr));
    broad_adr.sin_family = AF_INET;
    broad_adr.sin_addr.s_addr = inet_addr(argv[1]);
    broad_adr.sin_port = htons(atoi(argv[2]));

    //设置 socketopt 选项为广播模式
    setsockopt(send_sock, SOL_SOCKET, SO_BROADCAST, (void*)&so_brd, sizeof(so_brd));

    //以文本模式打开in.txt备读
    std::ifstream srcFile("news.t", std::ios::in);
    if(!srcFile) { //打开失败
        error_handle("ifstream", "ifstream error.");
    }

    //可以像用cin那样用ifstream对象
    memset(buf, 0, BUFFSIZE);
    while(srcFile >> buf)
    {
        sendto(send_sock, buf, strlen(buf), 0, (struct sockaddr*)&broad_adr, sizeof(broad_adr));
        memset(buf, 0, BUFFSIZE);
    }

    //关闭套接字
    close(send_sock);
    return 0;
}
```

1. 完成代码添加后，在 VS Code 平台中的终端窗口使用如下命令完成可执行文件的编译构建：

```bash
g++ news_sender_brd.cpp -o news_sender_brd
```

如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/a3bc113218ba1586848a121e4510fbbc-0)

### 广播方式接收者代码实现

1. 在 VS Code 平台中新建广播方式发送者文件 `news_receiver_brd.cpp` ，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/5e8de364561b59ade779d965f509bbd1-0)

1. 在 `news_receiver_brd.cpp` 源文件中添加如下代码：

```C++
#include <iostream>
#include <fstream>
#include <string>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUFFSIZE    64

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    //声明变量
    int recv_sock;
    struct sockaddr_in adr;
    int str_len;
    char buf[BUFFSIZE];

    //判断当前命令行参数是否有效
    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << "<PORT>" << std::endl;
        exit(1);
    }

    //创建 UDP socket套接字
    recv_sock = socket(PF_INET, SOCK_DGRAM, 0);

    //初始化UDP 套接字及端口号
    memset(&adr, 0, sizeof(adr));
    adr.sin_family = AF_INET;
    adr.sin_addr.s_addr = htonl(INADDR_ANY);
    adr.sin_port = htons(atoi(argv[1]));

    //绑定接收数据的 UDP 端口
    if(bind(recv_sock, (struct sockaddr*)&adr, sizeof(adr)) == 1)
    {
        error_handle("bind", "bind() error.");
    }

    //循环接收广播数据并打印输出
    while(1)
    {
        memset(buf, 0, sizeof(buf));
        str_len = recvfrom(recv_sock, buf, BUFFSIZE -1, 0, NULL, 0);
        if(str_len < 0)
        {
            break;
        }
        buf[str_len] = 0;
        std::cout << buf << std::endl;
    }

    close(recv_sock);
    return 0;
}
```

1. 完成代码添加后，在 VS Code 平台中的终端窗口使用如下命令完成可执行文件的编译构建：

```bash
g++ news_receiver_brd.cpp -o news_receiver_brd
```

如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/2fdaa1b909458e1ad84f9542194e25d4-0)

### 广播方式实例程序演示

1. 启动服务器，指定接收消息端口号为 18888。在终端窗口中输入如下命令：

```bash
./news_receiver_brd 18888
```

如图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/b1c24a83d927183acf10f8054980769e-0)

1. 使用另一个终端窗口启动客户端，指定服务器 IP 地址和端口号（`127.0.0.1` 表示本地环回地址，`18888` 指前面服务器启动指定端口号）

```bash
./news_sender_brd 127.0.0.1 18888
```

新建终端窗口，只需要点击原来终端窗口上的 `+` 号键，然后中间就会显示 `2:zsh` 表示第二个终端：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/f8dbfd00cd01b987573f1cac0467deef-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/350a269e3f69cb2108fbec313d6e8378-0)

## 6 实验总结

通过本章节的学习和练习，能够对网络编程中有关于 UDP 传输方式的应用有一定了解和掌握。前一节已经对 UDP 普通数据的传输方式进行了学习，而本章的内容重点是对于在有一定特定场景下的 UDP 传输方式的应用。在需要多播和广播方式的场景下，也就是需要众多接收者同时接收到数据而不需要发送者依次挨个发送的场景下，UDP 的多播和广播方式就很有用，相信大家经过本章内容的学习能够对 UDP 传输有一些特别的理解，有助于将来在项目开发中的使用。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code3.zip
```

# 四 基于 TCP 的服务器和客户端实现

## 1 实验介绍

前面几个章节已经讲述了关于 UDP 传输协议的相关概念和实例程序，从本章开始将详细讲述在 Linux 下的网络编程的重点 ———— TCP 传输方式及实现。

通过本章实现的学习，能够了解到关于 TCP 传输方式的原理及过程，并且通过代码编写掌握在 Linux 平台下使用 C++ 实现简单的 TCP 服务器和客户端的具体例程。

#### 知识点

- 理解 TCP 传输协议、TCP 传输过程
- TCP 传输方式原理
- 实现基于 TCP 的基本服务器端、客户端
- 构建 C++ 基本项目目录及编写 Makefile 文件
- 实现基于 TCP 方式的回声服务器实例程序

## 2 TCP 传输协议概述

TCP 协议的全称为传输控制协议，而 TCP 其实就是 Transmission Control Protocol 的简写。结合前面学习 UDP 协议时的了解，我们知道 TCP 协议是面向连接和流控制的。

前面在讲解 UDP 传输协议时已经讲述郭关于 IP 层的作用了，IP 层主要是解决数据传输过程中路径选择问题。在使用 TCP 传输协议时，虽然也是基于 IP 链路层之上实现了数据传输，但是为了保证数据传输的可靠性，TCP 协议在实际传输过程中添加了关于数据的接收反馈及数据校验等等操作来确保网络数据传输过程中的稳定可靠。（当然，这样做必然会损失部分效率）

### TCP 数据传输过程

与 UDP 传输方式最大的不同，TCP 传输方式需要建立连接，除此之外，还有对数据包传输的顺序要求（既对每次接受到的数据都有顺序要求，顺序不对会向对方发出正确序号数据包的接收请求，确保数据不丢失）

下面的几幅图形象描述的 TCP 传输方式在建立连接、数据传以及断开连接这几部分的具体操作过程：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/ab62da9b4e0c5439a815eeb958649850-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/fd377b75234846225fbe6f062e587af8-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/4fed0065f47da7df3c0404f18103dfd2-0)

## 3 构建 C++ 基本项目结构

1. 构建项目目录

按照前面分析的，我们首先创建一个项目根目录文件，也就是项目的名称 `TCP-ServerDemo` 。在 VS Code 平台左侧的窗口中点击 `新建文件夹` 按钮，命名新文件夹的名称为 `TCP-ServerDemo` ，具体如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/aa655fae1c3f4accaec2e52d610955e3-0)

完成后，根据前面所描述的关于项目的基本结构设计，在新创建的文件夹 `TCP-ServerDemo` 内创建对应的文件夹以及对应的文件：

- `src`（源代码文件目录)
- `include` (头文件目录)
- `tools`（工具文件目录）
- `test`（测试文件目录）
- `Makefile`（项目构建文件）

具体如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/c35f8404c313f2a3a9a23dae84ecafdd-0)

1. 创建 `main.cpp` 主文件

在完成项目目录结构创建后，在对应的源文件目录中新建主函数文件 `main.cpp` ，如下图示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/1ec1d204b05d4ea9de28c8c3039169ca-0)

创建完成后，先在 `main.cpp` 文件中添加用于测试验证的源代码内容，具体如下：

```C++
#include <iostream>
#include "base.h"

int main(int argc, char *argv[])
{
    char username[BUFFSIZE] = {0};
    std::cout << "Hello world" << std::endl;

    std::cout << "Please Input Your Name :" <<std::endl;

    std::cin >> username;

    std::cout << "Hi! " << username << " , Welcome China !" << std::endl;

    return 0;
}
```

在 `main.cpp` 中引用了自定义实现的头文件 `base.h` ，这里就需要在项目的头文件目录中创建它。

在项目头文件目录 `include` 中创建头文件 `base.h` ，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/d7711278d8b072ac5ec48de923c68276-0)

创建完成后，添加如下头文件代码：

```C++
#ifndef __BASE__H
#define __BASE__H

#define BUFFSIZE    64

#endif
```

1. 创建 Makefile 文件

在项目的根路径创建用于实现项目编译构建的工具文件 `Makefile` 文件并向其中添加基本构建内容：

```bash
pwd=${shell pwd}
src_dir=$(pwd)/src
include_dir=$(pwd)/include/
tool_dir=$(pwd)/tools/
test_dir=$(pwd)/test/

out_dir=.
target=TcpServerDemo



########################## src section #################################
obj_src_dir=$(src_dir)
obj_src=${shell find $(obj_src_dir) -name "*.cpp"}
objs=${patsubst %.cpp,%.o,$(obj_src)}
deps=${patsubst %.cpp,%.d,$(obj_src)}


#########################################################################

########################## tools #######################################

########################################################################

####################### flag section ########################################
VER ?= 0.1

OF=-g
CFLAGS=-I${include_dir}

CC=g++

COMMIT_ID ?= ${shell git rev-parse --short HEAD}


libs=
LDFLAGS=

#############################################################################


.PHONY: all
all: $(target)
$(target): $(objs)
    $(CC) $(CFLAGS)  -o $@ $(objs)

%.o:%.cpp
    $(CC) ${CFLAGS} -c -o $@ $<


clean:
    -rm $(target)
    -rm $(objs)
```

对于 Makefile 文件内容，如果大家是从左侧实验文档 copy 过来的话，需要注意修改一下缩进问题。文档中的缩进使用的是空格，而 Makefile 语言默认的是制表符缩进。点击环境右下角的“制表符长度”：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/c0767ba3a171531624f98133e2fa38d1-0)

然后选择“将缩进转换为制表符”：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/0fb59dd2411f013a25ff5f3c23505673-0)

注意每一次对于 Makefile 文件内容进行了修改，就需要进行一次转换，否则将会提示“缺失分隔符。停止。”的报错信息。

1. 项目构建验证

在完成 Makefile 文件创建和内容添加后，就可以在 VS Code 的终端窗口中使用 `make` 命令实现项目的编译构建了。这里，我们来尝试编译验证一下，看项目是否构造完成：

```bash
cd /home/project/code4/TCP-ServerDemo
make
```

完成操作后，终端窗口显示如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/a9c3011d5655293316a731850a4c7d36-0)

## 4 实现基于 TCP 服务器实例程序

前面，我们完成了基本的 C++ 项目构建，网络编程的主要内容其实就是在服务器端，所以项目命名为 `TCP-ServerDemo` ，接下来就要在项目中实现最基础的 TCP 服务器。

在 `TCP-ServerDemo` 目录下的 `src` 目录中创建新的主函数文件 `main.cpp` （之前用于项目构建测试的 `main.cpp` 文件需要删除掉）。创建完成如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/1ec1d204b05d4ea9de28c8c3039169ca-0)

创建完成后，在 `main.cpp` 文件中添加如下代码：

```C++
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <cstring>
#include <sys/socket.h>
#include "base.h"

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    int serv_sock, client_sock;
    char message[BUFFSIZE];
    int str_len, ret;

    struct sockaddr_in serv_adr, client_adr;
    socklen_t client_adr_size;

    //判断命令行参数合法性
    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << " <port>" << std::endl;
        exit(1);
    }

    //创建socket套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //初始化服务器套接字参数，设置网卡IP 和端口号
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    //绑定端口
    ret = bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr));
    if(ret < 0)
    {
        error_handle("bind", "bind() error.");
    }

    //监听TCP端口号
    ret = listen(serv_sock, 5);
    if(ret < 0)
    {
        error_handle("listen", "listen() error.");
    }

    client_adr_size = sizeof(client_adr);

    while(1)
    {
        //使用accept接收客户端连接请求
        client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &client_adr_size);
        if(client_sock < 0)
        {
            //接收客户端连接请求失败，
            continue;
        }
        //接收新的客户端请求，进行客户端数据处理
        std::cout << "Accept New Client : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
        std::cout << "Start Recv Client Data..." << std::endl;

        //清空缓存区
        memset((void *)&message, 0, BUFFSIZE);
        while((str_len = read(client_sock, message, BUFFSIZE)) != 0)
        {
            //成功读取客户端发送来的数据消息
            //打印输出
            std::cout << "Recv Message : " << message << std::endl;
            //将消息回传给客户端，作为回声服务器，类似 echo 命令
            write(client_sock, message, str_len);
            //清空缓存区，等待再次读取数据
            memset(message, 0, BUFFSIZE);
        }

        //客户端断开连接，关闭套接字
        close(client_sock);
    }

    //服务器关闭，关闭服务器套接字
    close(serv_sock);
    return 0;
}
```

接下来通过实现 TCP 客户端实例程序进行具体理解，在我们构建的 C++ 项目中，客户端程序主要作为我们的测试程序编写，所以存放在 `TCP-ServerDemo` 项目中的 `test` 目录下，具体如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/90fa7b6b5601ee0c5b1e98112a5994a3-0)

创建完成 `TcpClient.cpp` 文件后，在其中添加如下实现代码：

```C++
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <cstring>
#include <sys/socket.h>
#include "base.h"

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

int main(int argc, char *argv[])
{
    int sock, ret;
    char message[BUFFSIZE];
    int str_len;
    struct sockaddr_in serv_adr;

    if(argc < 3)
    {
        std::cout << "Usage : " << argv[0] << " <IP> <port>" << std::endl;
        exit(1);
    }

    sock = socket(PF_INET, SOCK_STREAM, 0);
    if(sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_adr.sin_port = htons(atoi(argv[2]));

    ret = connect(sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr));
    if(ret < 0)
    {
        error_handle("connect", "connect() error.");
    }

    std::cout << "Connect Success..." << std::endl;

    //进入数据处理
    while(1)
    {
        std::cout << "Please Input Message(Q to quit) : " << std::endl;
        std::cin >> message;
        if((!strcmp(message, "q")) || (!strcmp(message, "Q")))
        {
            //退出客户端
            break;
        }

        //将数据发送给服务端
        write(sock, message, strlen(message));
        //读取服务端回传的数据
        str_len = read(sock, message, BUFFSIZE-1);
        message[str_len] = 0;
        //打印输出
        std::cout << "Echo Message : " << message << std::endl;
    }

    //关闭套接字
    close(sock);
    return 0;
}
```

完成代码添加后，需要修改项目的 Makefile 文件以方便实现测试程序的编译构建，具体修改如下：

```bash
pwd=${shell pwd}
src_dir=$(pwd)/src
include_dir=$(pwd)/include/
tool_dir=$(pwd)/tools/
test_dir=$(pwd)/test/

out_dir=.
target=TcpServerDemo



########################## src section #################################
obj_src_dir=$(src_dir)
obj_src=${shell find $(obj_src_dir) -name "*.cpp"}
objs=${patsubst %.cpp,%.o,$(obj_src)}
deps=${patsubst %.cpp,%.d,$(obj_src)}

#添加测试文件的内容
test_src=${shell find ${test_dir} -name "*.cpp"}
test_obj=${patsubst %.cpp,%.o,${test_src}}
tests=${patsubst %.cpp,%,${test_src}}
#########################################################################

########################## tools #######################################

########################################################################

####################### flag section ########################################
VER ?= 0.1

OF=-g
CFLAGS=-I${include_dir}

CC=g++

COMMIT_ID ?= ${shell git rev-parse --short HEAD}


libs=
LDFLAGS=

#############################################################################


.PHONY: all
all: $(target)
$(target): $(objs)
    $(CC) $(CFLAGS)  -o $@ $(objs)

%.o:%.cpp
    $(CC) ${CFLAGS} -c -o $@ $<

#添加编译测试文件的规则使用
.PHONY: test
test: $(tests)
$(tests): $(test_obj)
    $(CC) $(CFLAGS)  -o $@ $(test_obj)

$(test_obj): $(test_src)
    $(CC) ${CFLAGS} -c -o $@ $<


clean:
    -rm $(target)
    -rm $(objs)
    -rm ${test_obj}
    -rm ${tests}
```

## 5 测试验证

将 TCP 服务器与客户端的函数构建流程对照起来看，就能明白双方通信建立连接和进行数据传输的过程，具体如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/776faeffffbeb7c88072b50a423c3053-0)

完成代码添加后，通过执行 `make` 命令，借助项目的 Makefile 文件就能够完成可执行程序的编译构建，具体如下图所示：

```bash
cd /home/project/code4/TCP-ServerDemo
make
make test
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/71a1c1b8e6c6461f17a43c2cde9d6893-0)

可执行程序编译完成后，通过在 VS Code 平台下的终端窗口输入命令执行程序来进行 TCP 数据通信测试，具体如下图：

```bash
cd /home/project/code4/TCP-ServerDemo
./TcpServerDemo 18888

# 新开一个终端
cd /home/project/code4/TCP-ServerDemo/test
./TcpClient 127.0.0.1 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/e21a23894e88db33c325632c2f61e112-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/4ed2a2c37eb1e2e4e012d6ab9d3fdc5d-0)

测试验证的方式跟之前类似，一个终端窗口先启动服务端程序运行，然后新建一个终端窗口运行客户端与服务器通信以验证我们实现的程序是否运行正常。

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/61da736f15212527ed114c34c50dbdfc-0)

## 6 实验总结

通过本章节的学习，能够从中掌握基础的 C++ 项目构建和 Makefile 文件编写，结合前面已经学习到的内容完成最基础的 TCP 服务器和客户端例程代码实现，方便后面章节的讲解和新内容迭代。作为后面章节的基础，本章节的内容十分重要，请大家勤加练习并加以理解。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code4.zip
```

# 五 TCP 半关闭及 socket 多种可选项

## 1 实验介绍

通过前一章节的内容，我们学习到了关于 TCP 通信和基于 TCP 的服务器实现相关内容。事实上，在实际应用中，还需要注意对 TCP 通信中的具体操作步骤和实现细节进行设定和控制，从而才能使得开发出的应用程序在实际使用过程中不会出现太多不可控的情况。本章内容就是通过对 TCP 套接字参数进行设置从而使得通信过程能够与设计实现的现象一致。

#### 知识点

- 基于 TCP 的半关闭操作
- 套接字可选项及 I/O 缓冲大小设置
- SO_REUSEADDR 操作
- TCP_NODELAY 操作

## 2 基于 TCP 的半关闭

通过前面的学习肯定理解并掌握了 TCP 建立连接的过程（三次握手），本节将要详细介绍更为重要的 TCP 断开连接的过程。在 TCP 建立连接后基本处于稳定状态，但是当发生连接断开时，由于断开的过程不同产生的结果也是不一样的，接下来我们就来详细研究一下 TCP 连接断开的相关问题。

### 2.1 单方面断开连接的问题

TCP 断开过程发生问题往往是由于客户端或者服务端一方单方面关闭了连接导致的，前面实现的例程也都是直接调用 `close()` 函数来实现断开连接的，这样的操作会导致目前处于连接状态的套接字直接完全关闭（既不能接收数据也不能发送数据）。单方面断开连接的最大的问题就是对方并不知道此时 TCP 连接发送断开，从而导致对方措手不及发送一系列问题。

单方面断开连接的图示如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/7fc1fb6d9c01eb0812e7c37a81f48f87-0)

从图中能够看到，当连接双方处于正常双向通信时。A 主机在完成自己的数据发送后直接调用 `close()` 函数断开连接，断开连接后也就读取不到最后 B 主机发送回来的信息，所以这部分的数据由于连接断开最终只能销毁掉。但是作为被迫断开连接的 B 主机并不知道此时已经断开连接了，它还是正常接收到 A 发送来的最后的数据并且也正常发送了一包被销毁的数据，直到 B 再次调用接收数据函数才会发现 TCP 连接断开了，这导致 B 主机的状态十分被动并不可控。

为了解决这样的问题，Linux 下对 TCP 连接提出了半关闭的方式，也就是将双向传输改变成单向传输的方式（可以发送数据不能接受数据、可以接受数据不能发送数据）。

### 2.2 套接字和 I/O 流

当 TCP 双方建立连接后就进入可交换数据的状态，此时可以将建立连接后可交换数据的状态看作是一种数据流。

由于 TCP 建立连接后，双方是能够进行数据的双向通信的，也就是每一方都同时能够读取数据和发送数据，因此可以将此时的连接看成是两个 I/O 流，如下图所示；

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/1eb33156ee57d90ed52a25f8e6db4ff3-0)

从图中能看到在 TCP 建立连接后正常是会有两个 I/O 流在进行数据的传输，而前面提到的 TCP 的半关闭其实就是将其中一条 I/O 流断开，只保留一条继续进行数据传输。

### 2.3 TCP 半关闭 shutdown 函数

能够仅关闭 TCP 连接中的一条 I/O 数据流的操作函数就是 `shutdown()` 函数，具体如下：

```C++
#include <sys/socket.h>
//成功时返回 0，失败返回 -1
int shutdown(int sock, int howto);
```

- sock：需要断开的套接字文件描述符
- howto：传递断开方式信息

其中第二个参数 `howto` 分别有三种不同的断开方式，如下：

- SHUT_RD：仅断开输入流
- SHUT_WR：仅断开输出流
- SHUT_RDWR：同时断开 I/O 流

举例说明：如果使用 `SHUT_RD` 作为第二个参数使用 `shutdown()` 函数，则将断开当前的输入流导致套接字无法接收数据（无法调用输入相关函数），但是不会影响输出流的相关操作（可以继续调用输出函数发送数据）。

## 3 socket 套接字可选项及 I/O 缓冲大小设置

通常在进行套接字网络编程时往往都重点关注数据通信，所以经常会忽视了套接字的不同特性，其实了解这些不同特性并且在使用时进行相应的更改设置也是十分重要的。

### 套接字多种可选项

前面在创建套接字时都是直接创建并使用，这种情况下都是通过默认的套接字特性进行数据通信的。下面列出一部分套接字可选项：

| 协议层      | 选项名            | 可读取 | 可设置 |
| ----------- | ----------------- | ------ | ------ |
| SOL_SOCKET  | SO_SNDBUF         | √      | √      |
|             | SO_RCVBUF         | √      | √      |
|             | SO_REUSEADDR      | √      | √      |
|             | SO_KEEPALIVE      | √      | √      |
|             | SO_BROADCAST      | √      | √      |
|             | SO_DONTROUTE      | √      | √      |
|             | SO_OOBINLINE      | √      | √      |
|             | SO_ERROR          | √      | ×      |
|             | SO_TYPE           | √      | ×      |
| IPPROTO_IP  | IP_TOS            | √      | √      |
|             | IP_TTL            | √      | √      |
|             | IP_MULTICAST_TTL  | √      | √      |
|             | IP_MULTICAST_LOOP | √      | √      |
|             | IP_MULTICAST_IF   | √      | √      |
| IPPROTO_TCP | TCP_KEEPALIVE     | √      | √      |
|             | TCP_NODELAY       | √      | √      |
|             | TCP_MAXSEG        | √      | √      |

从上面的表中能够看出，套接字可选项是分层的，其中 `IPPROTO_IP` 层可选项是 IP 协议相关事项，`IPPROTO_TCP` 层可选项是 TCP 协议相关事项，`SOL_SOCKET` 层是套接字相关的通用可选项。

通过 `getsockopt()` 和 `setsockopt()` 两个函数能够完成对上面表中可选项的读取和设置操作。具体如下：

```C++
#include <sys/socket.h>

//读取套接字可选项值，成功返回0，失败返回 -1
int getsockopt(int sock, int level, int optname, void *optval, socklen_t *optlen);
/*
- sock 用于查看选项套接字文件描述符
- level 要查看的可选项的协议层
- optname 要查看的可选项名称
- optval 获取查看结果的缓冲地址
- optlen 保存查看结果的缓冲空间占有字节大小
*/

//设置套接字可选项值，成功返回0，失败返回 -1
int setsockopt(int sock, int level, int optname, const void *optval, socklen_t optlen);
/*
- sock 用于修改选项套接字文件描述符
- level 要修改的可选项的协议层
- optname 要修改的可选项名称
- optval 保存要更改的可选项信息的缓冲地址
- optlen 第四个参数值的字节大小
*/
```

接下来，通过演示一个实例程序让我们进一步了解如何查看套接字可选项值及设置部分可选项的值，具体如下：

1. 新建实例程序源代码文件 `sockoptDemo.cpp`，将新建的文件暂时放置到项目 `TCP-ServerDemo/tools/` 目录下，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/73d68159c07067860aa0b0fc6519ba3e-0)

1. 创建文件后，在其中添加如下代码：

```C++
#include <iostream>
#include <unistd.h>
#include <sys/socket.h>

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}


int main(int argc, char *argv[])
{
    int tcpSock, udpSock;
    int sockType;
    socklen_t optlen;
    int state;

    optlen = sizeof(sockType);

    //创建 TCP socket
    tcpSock = socket(PF_INET, SOCK_STREAM, 0);
    //创建 UDP socket
    udpSock = socket(PF_INET, SOCK_DGRAM, 0);

    std::cout << "Tcp Socket : " << tcpSock << std::endl;
    std::cout << "Udp Socket : " << udpSock << std::endl;

    state = getsockopt(tcpSock, SOL_SOCKET, SO_TYPE, (void*)&sockType, &optlen);
    if(state < 0)
    {
        error_handle("getsockopt", "getsockopt() error.");
    }
    std::cout << "Tcp Socket type : " << sockType << std::endl;

    state = getsockopt(udpSock, SOL_SOCKET, SO_TYPE, (void*)&sockType, &optlen);
    if(state < 0)
    {
        error_handle("getsockopt", "getsockopt() error.");
    }
    std::cout << "Udp Socket type : " << sockType << std::endl;

    return 0;
}
```

1. 编译，查看程序运行结果。在 VS Code 平台的终端窗口输入如下命令：

```bash
cd /home/project/code5/TCP-ServerDemo/tools
g++ sockoptDemo.cpp -o sockoptDemo
./sockoptDemo
```

程序运行结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/a44cb7aaf08643718e54197ef540b563-0)

### 套接字 I/O 缓冲大小设置

在前面的套接字可选项中，我们看到在 `SOL_SOCKET` 层有两个可选项 `SO_SNDBUF` 和 `SO_RCVBUF` ，这两个可选项是用于设置套接字 I/O 缓冲大小的。

`SO_RCVBUF` 可选项是输入缓冲大小相关可选项，而 `SO_SNDBUF` 则是输出缓冲大小相关可选项，我们能够通过设置这两个套接字可选项的值能够对套接字的 I/O 换成大小进行修改。

下面通过两个简单的实例程序来掌握套接字 I/O 缓冲大小的设置：

1. 获取当前系统的套接字默认 I/O 缓冲大小值。

- 新建实例程序源代码文件 `tools/getsockBuf.cpp` ，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/61695582fac8fd192ffdaee0a1dd968b-0)

- `getsockBuf.cpp` 文件创建完成后，添加如下代码：

```C++
#include <iostream>
#include <unistd.h>
#include <sys/socket.h>



//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}


int main(int argc, char *argv[])
{
    int sock;
    int sendBuf, recvBuf, state;
    socklen_t optlen;

    //创建socket
    sock = socket(PF_INET, SOCK_STREAM, 0);

    //获取socket可选项 SO_SNDBUF 的值
    optlen = sizeof(sendBuf);
    state = getsockopt(sock, SOL_SOCKET, SO_SNDBUF, (void*)&sendBuf, &optlen);
    if(state < 0)
    {
        error_handle("getsockopt", "getsockopt() error.");
    }
    //打印输出
    std::cout << "Send Buf size : " << sendBuf << std::endl;

    //获取socket可选项 SO_RCVBUF 的值
    optlen = sizeof(recvBuf);
    state = getsockopt(sock, SOL_SOCKET, SO_RCVBUF, (void*)&recvBuf, &optlen);
    if(state < 0)
    {
        error_handle("getsockopt", "getsockopt() error.");
    }
    //打印输出
    std::cout << "Recv Buf size : " << recvBuf << std::endl;

    return 0;
}
```

- 添加代码后，在 VS Code 平台的终端窗口执行如下命令进行程序运行：

```bash
cd /home/project/code5/TCP-ServerDemo/tools
g++ getsockBuf.cpp -o getsockBuf
./getsockBuf
```

- 执行显示结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/dc4f91b64548abdeea039c5cabfdaa8c-0)

1. 修改套接字 I/O 缓冲大小。

- 新建修改套接字 I/O 缓冲大小实例程序源代码文件 `tools/setsockBuf.cpp` ，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/18e594c1ddab43b9f0ec6e282b08ee6a-0)

- `setsockBuf.cpp` 文件创建完成后，添加如下代码：

```C++
#include <iostream>
#include <unistd.h>
#include <sys/socket.h>



//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}


int main(int argc, char *argv[])
{
    int sock;
    int sendBuf = 1024, recvBuf = 4096;
    int state;

    socklen_t optlen;

    //创建socket
    sock = socket(PF_INET, SOCK_STREAM, 0);

    //设置socket 可选项 SO_SNDBUF SO_RCVBUF
    state = setsockopt(sock, SOL_SOCKET, SO_SNDBUF, (void*)&sendBuf, sizeof(sendBuf));
    if(state < 0)
    {
        error_handle("setsockopt", "setsockopt() error.");
    }
    state = setsockopt(sock, SOL_SOCKET, SO_RCVBUF, (void*)&recvBuf, sizeof(recvBuf));
    if(state < 0)
    {
        error_handle("setsockopt", "setsockopt() error.");
    }

    std::cout << "setsockopt end..." <<std::endl;

    //获取socket 可选项 SO_SNDBUF SO_RCVBUF
    optlen = sizeof(sendBuf);
    state = getsockopt(sock, SOL_SOCKET, SO_SNDBUF, (void*)&sendBuf, &optlen);
    if(state < 0)
    {
        error_handle("getsockopt", "getsockopt() error.");
    }
    //打印输出
    std::cout << "Send Buf size : " << sendBuf << std::endl;

    optlen = sizeof(recvBuf);
    state = getsockopt(sock, SOL_SOCKET, SO_RCVBUF, (void*)&recvBuf, &optlen);
    if(state < 0)
    {
        error_handle("getsockopt", "getsockopt() error.");
    }
    //打印输出
    std::cout << "Recv Buf size : " << recvBuf << std::endl;

    return 0;
}
```

- 添加代码后，在 VS Code 平台的终端窗口执行如下命令进行程序运行：

```bash
cd /home/project/code5/TCP-ServerDemo/tools
g++ setsockBuf.cpp -o setsockBuf
./setsockBuf
```

- 执行显示结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/b4c82c7ed410fe79da951dff168d77a7-0)

从设置例程的执行结果中能看到，虽然 I/O 缓冲大小被修改了，但是并没有修改成我们设置的值，这是因为系统在执行设置时会优先选择最佳方案，而不会直接设置成我们给定的值（避免因为我们的设置导致套接字数据传递发生更多问题）

## 4 SO_REUSEADDR

在套接字可选项中除了需要注意 I/O 缓冲区大小外，最应该注意的就是 `SO_REUSEADDR` 这个可选项，在学习这个可选项之前，必须理解套接字 Time-wait 状态。在之前的实例过程中最先调用 `close()` 断开连接的都是客户端，这本身没有什么问题，但是如果先让服务端调用 `close()` 断开连接的话，短时间内再次启动服务端绑定同一端口号就会发生 `bind() error` 的错误消息，默认等待 2 到 3 分钟再次启动服务器就正常了。造成这样的情况的根本原因就是哪一方最先传输出的 `FIN` 消息。

而之所以服务端和客户端发送 `FIN` 消息造成的现象的不一样，其实最重要的情况是要对 TCP 连接的正常断开连接的四次握手有一定理解，关于 TCP 断开连接的四次握手如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/a4a75516c25f9f5738fd96326c218a3f-0)

从图中能够看出，Time-wait 状态其实就是最早发出 `FIN` 的一方在最后发出 `ACK` 消息后的时间等待，也就是在接收到对方 `FIN` 消息后反馈一个 `ACK` 消息并等待一定时间再关闭，这段时间的状态就是 Time-wait 状态。不论是客户端还是服务端，只要是先发送 `FIN` 消息就会有这个状态产生，但是只有服务端会绑定端口导致在关闭之后这个状态还占用网络端口，所以在这个状态下再次启动服务器会报 `bind() error` 的端口号绑定错误。

其实 Time-wait 状态就是为了确保前一次发送的 `ACK` 消息能够让对方准确接收到，如果对方没有成功收到 `ACK` 消息，那么 Time-wait 状态就会重新传输 `ACK` 信号确保对方能够正常终止。这种现象说白了是 TCP 保证数据可靠传输的一种方法，但是在实际应用中经常会因为这样的状态导致服务器在发生问题后短期内无法快速重新启动，而既能够使得 TCP 数据传输可靠性不丢又能够让服务器在 `close()` 后短期内能够快速启动不会发生端口占用的问题，其实只需要修改套接字可选项 `SO_REUSEADDR` 即可。

套接字可选项 `SO_REUSEADDR` 默认值为 0 ，表示不会重新分配 Time-wait 状态下的套接字端口，也就是在 TCP 连接进入 Time-wait 状态后依然占用之前的 TCP 端口号。如果将 `SO_REUSEADDR` 设置为 1 ，那么当 TCP 连接进入 Time-wait 状态后就会重新分配端口号，而不会占用之前端口，这样就不会影响新的服务再次启动了。

具体修改方式如下：

```C++
int option;
socklen_t optlen;

optlen = sizeof(option);
option = TRUE;
setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, (void*)&option, optlen);
```

我们在 `TCP-ServerDemo` 项目的服务器中添加这一修改来试一下，具体修改 `src/main.cpp` 文件中：

```C++
int main(int argc, char *argv[])
{
    ...
    struct sockaddr_in serv_adr, client_adr;
    socklen_t client_adr_size;
    // 添加 optlen 声明
    int option;
    socklen_t optlen;

    ...
    //创建socket套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //添加 Time-wait 状态重新分配断开设置
    optlen = sizeof(option);
    option = 1;
    setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, (void*)&option, optlen);

    //初始化服务器套接字参数，设置网卡IP 和端口号
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));
    ...
```

修改完成后，在 VS Code 平台的终端窗口中完成项目的编译构建，如下图：

```bash
cd /home/project/code5/TCP-ServerDemo
make
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/e3f8cb23d7e85b321e510d475328c75a-0)

完成编译后，测试一下我们的修改是否有效。

修改之前的服务器断开后再次启动如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/3d900e5cdcd46780aa7c97b3881fd421-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/30b73534bebacabf0dbb1c60128c6f0d-0)

修改后的服务器断开再次启动如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/563f55292e2d282ea178455efe016e29-0)



## 5 TCP_NODELAY

还需要提到的一个套接字可选项就是 `TCP_NODELAY` ，这个可选项其实就是防止 TCP 数据在传输过程中因为 `Nagle` 算法导致的效率低下问题， `Nagle` 算法其实就是一个 TCP 数据传输中的数据包等待算法，具体示意如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/266a619a8db2176a338c74fb0d43e00f-0)

简单描述就是如果在 TCP 传输中不禁用 `Nagle` 算法的话，那么每次发送的数据包都会等待一定时间再发送出去，其实就是为了能够等待更多的数据再一次性发送，节省发包次数，但这样做会导致传输速率下降很多。尤其是在大文件数据传输的场景，需要尽快将每次缓冲区中的数据及时发送出去，这时候就需要禁用 `Nagle` 算法。其实禁用这样的方式很简单，就是设置套接字可选项 `TCP_NODELAY` 为 1 （真） 即可。具体修改方式如下：

```C++
int opt_val = 1;
int opt_get_val;
socklen_t opt_len;
opt_len = sizeof(opt_get_val);

//获取TCP_NODELAY可选项的值
getsockopt(sock, IPPROTO_TCP, TCP_NODELAY, (void*)&opt_get_val, &opt_len);

//设置TCP_NODELAY可选项为真（1）
setsockopt(sock, IPPROTO_TCP, TCP_NODELAY, (void*)&opt_val, sizeof(opt_val));
```

这个参数设置略作了解即可，通常情况下不会经常用到的，所以这里就不再多说了。

## 6 实验总结

通过本节实验的学习，能够对 TCP 连接的具体细节部分深入了解，并且了解到关于 socket 套接字可选性的设置能够对套接字的种种行为进行设定和修改。相信大家经过本节的学习后能够对 TCP 网络通信方式的理解更加深刻。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code5.zip
```

# 六 多进程服务端实现

## 1 实验介绍

到目前为止，前面章节完成的都是基于单任务单线程的 TCP 服务端实现，虽然能够完成对客户端请求的任务响应，但是当需要同时响应多个甚至上万个客户端的请求响应时，肯定不可能使用单线程的 TCP 服务端来作为网络服务器使用。

现有大部分的 TCP 服务器都是基于以下三种模式实现的：

- 多进程服务器：通过创建多个进程来响应并发服务。
- 多线程服务器：通过生成与客户端请求等量的线程来提供服务。
- 多路复用服务器：通过统一管理 I/O 对象来优化服务响应效率。

以上三种模式也可以混合使用已达到高并发服务器的最佳效果。

本章将先重点讲解多进程服务器的实现，实际应用中的多进程服务器大部分是混合多线程服务器以达到单机高并发的效果。

#### 知识点

- Unix 平台下进程的概念及应用
- Unix 平台下的信号响应机制
- 基于多任务的多进程并发服务器

## 2 Unix 平台下的信号响应机制

前面在介绍 Linux 下使用 `fork()` 函数创建子进程中讲述了关于僵尸进程的产生原因和危害，而要避免产生僵尸进程，只需要父进程在恰当的时候将子进程的资源进行回收就可以了。大家可能都知道可以使用 `wait()` 系列函数去回收子进程，但是当用于服务器程序的过程中，最好避免使用会发生阻塞效果的函数，所以推荐使用 Unix 平台下的信号响应机制进行处理。

其实在 Linux 系统中运行的程序是肯定会与信号发生关系的，比如在运行程序的过程中，使用键盘按下 `Ctrl+C` 或者 `Ctrl+\` 就会向当前终端运行的进程发生 `SIGINT` 和 `SIGQUIT` 信号。还有通常使用 `kill` 去杀死一个进程时，我们使用 `kill -9 <pid>` 的方式也是通过向 `<pid>` 进场发送 `SIGKILL` 信号来 `杀死` 进程的。

除了系统默认的信号处理方式，我们在编写程序时也可以自定义部分信号的处理实现过程。使用系统函数 `signal()` 就可以完成自定义信号处理函数的注册步骤，该函数描述如下：

```C++
#include <signal.h>

//注册新的信号处理函数，返回之前注册的函数指针
void (*signal(int signo, void(*func)(int)))(int);
```

跟 `signal()` 函数相同用法的还有一个函数就是 `sigaction()` ，这里推荐大家还是使用 `sigaction()` 来注册信号处理函数，原因主要是在多种 Unix 平台下的 `signal()` 函数用法可能不一样，但是 `sigaction()` 函数的使用方法都是一致的，这样我们的程序在平台移植过程中不需要太多修改。具体描述如下：

```C++
#include <signal.h>

//成功返回 0， 失败返回 -1
int sigaction(int signo, const struct sigaction *act, struct sigaction *oldact);
/*
- signo ：与 signal 函数相同，传递信号序号
- act ：使用 sigaction 结构体注册新号处理函数
- oldact ：返回之前注册的新号处理函数指针，不需要传递 0 即可
*/
```

其中 `struct sigaction` 结构体描述如下：

```C++
struct sigaction{
    void (*sa_handler)(int);    //新号处理函数
    sigset_t sa_mask;   //通常不需要，设置 0 即可
    int sa_flags;   //同 sa_mask 一样，暂时不用
}
```

创建一个信号处理实例程序来了解简单的信号处理函数的注册使用。

1. 创建信号处理实例程序源代码文件 `signalDemo.cpp` ，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/8b6d99d853bc6fab16f8bfbccd8b25e1-0)

1. 创建文件完成后，在其中添加如下源代码：

```C++
#include <iostream>
#include <unistd.h>
#include <signal.h>
#include <sys/wait.h>

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}


void child_process_handle(int sig)
{
    int re;
    int pid;
    //非阻塞循环  因为信号不排队所以可能有几个子进程退出信号叠加在一起所以要循环
    while((pid = waitpid(-1,&re, WNOHANG)) > 0)
    {
        if(WIFEXITED(re))
        {
            std::cout << pid << " exit " << WEXITSTATUS(re) << std::endl;
        }
        if(WIFSIGNALED(re))
        {
            std::cout << pid << " exit " << WTERMSIG(re) << std::endl;
        }
    }
}

int main(int argc, char *argv[])
{
    int pid;
    int i = 10;
    while(i--)
    {
        if((pid = fork()) < 0)  //错误
        {
            error_handle("fork", "fork() error.");
        }
        else if(pid == 0)   //子进程
        {
            break;
        }
        else if(pid > 0)
        {
            continue;
        }
    }
    if(pid > 0)
    {
        struct sigaction act;
        act.sa_handler = child_process_handle;//设置信号处理函数
        sigemptyset(&act.sa_mask);
        act.sa_flags = 0;
        if(sigaction(SIGCHLD, &act, NULL) < 0)
        {
            error_handle("sigaction", "sigaction() error.");
        }
        while(1)
        {
            //父进程循环等待处理回收子进程
        }

    }
    if(pid == 0)
    {
        sleep(10);
    }
    return 0;
}
```

1. 在 VS Code 平台的终端窗口通过命令完成代码编译：

```bash
g++ signalDemo.cpp -o signalDemo
./signalDemo
```

1. 运行程序结果如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/ffcbf261cff724dc786f2a8fb9415502-0)

## 3 基于多任务的多进程并发服务器

通过基本介绍 Unix 平台下的进程概念，现在就开始利用多进程方式改造我们的 TCP 服务器实现服务的并发功能。目前为止，我们实现的 TCP 服务器还是基于单进程单线程模式的，同时间只能为一个客户端提供服务，如果在服务器调用 `accept()` 函数接收到客户端请求连接后，创建一个新的子进程去处理这个新连接的客户端请求服务。具体如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/950c86bc9fcf508f7888fa2d8002ab4f-0)

复制 `/home/project/code5` 文件夹，并命名为 `/home/project/code6` 文件夹。接下来的内容统一在 `code6` 中进行修改。

修改 `TCP-ServerDemo` 项目代码，在其中添加处理客户端消息的函数封装及多进程服务相关函数封装处理：

1. 打开 `TCP-ServerDemo/src/main.cpp` 源文件，在其中添加如下代码：

```C++
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <cstring>
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/wait.h>
#include "base.h"

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}


//子进程处理客户端请求函数封装
int ChildProcessWork(int ServSocket, int ChildSocket)
{
    char buf[BUFFSIZE] = {0};
    int str_len;
    //首先在子进程中关闭服务端套接字
    close(ServSocket);

    while(1)
    {
        //接收客户端数据
        str_len = read(ChildSocket, buf, BUFFSIZE);
        if(str_len == 0)
        {
            //客户端断开连接，处理结束
            break;
        }
        //多进程服务中不再将受到数据打印输出
        //将客户端数据返回给客户端
        write(ChildSocket, buf, str_len);
        memset(buf, 0, BUFFSIZE);
    }

    //关闭客户端套接字
    close(ChildSocket);
    return 0;
}

//处理子进程结束后，父进程回收资源的信号回调函数
void read_childproc(int signum)
{
    pid_t pid;
    int status;
    pid = waitpid(-1, &status, WNOHANG);
    std::cout << "Removed proc id : " << pid << std::endl;
}

//多进程工作
int fork_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    pid_t pid;

    //循环等待客户端请求
    while (1)
    {
        adr_size = sizeof(client_adr);
        client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
        if(client_sock < 0)
        {
            //接收请求失败，继续工作
            continue;
        }
        else{
            //接收到新的客户端，打印输出
            std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
            //创建子进程进行处理
            pid = fork();
            if(pid < 0)
            {
                //创建子进程失败，关闭连接
                std::cout << "fork error, close client" << std::endl;
                close(client_sock);
                continue;
            }
            if(pid == 0)
            {
                //进入子进程,
                ret = ChildProcessWork(serv_sock, client_sock);
                //调用结束后直接结束子进程
                exit(ret);
            }
            else{
                //父进程关闭客户端套接字，继续等待新客户端请求
                close(client_sock);
            }
        }
    }
    return 0;
}

//修改主函数
int main(int argc, char *argv[])
{
    int serv_sock, ret;
    struct sockaddr_in serv_adr;
    struct sigaction act;

    int str_len, state;
    int option;
    socklen_t optlen;
    char buf[BUFFSIZE];

    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << "<port> " << std::endl;
        exit(1);
    }

    act.sa_handler = read_childproc;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    state = sigaction(SIGCHLD, &act, 0);

    //创建服务socket套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //添加 Time-wait 状态重新分配断开设置
    optlen = sizeof(option);
    option = 1;
    setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, (void*)&option, optlen);


    //初始化服务套接字
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    //绑定套接字
    ret = bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr));
    if(ret < 0)
    {
        error_handle("bind", "bind() error.");
    }

    //监听端口
    ret = listen(serv_sock, 5);
    if(ret < 0)
    {
        error_handle("listen", "listen() error");
    }

    fork_workproc(serv_sock);

    //关闭套接字
    close(serv_sock);
    return 0;
}
```

## 4 测试验证

通过前面对我们的 TCP 服务项目代码的修改，增加了并发功能。现在就来构建编译进行验证。

1. 编译构建服务器，在 VS Code 平台中的终端窗口通过命令构建编译服务器：

```bash
cd /home/project/code6/TCP-ServerDemo
make
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/1e08abaf855f5a85ecbfc22721801dd2-0)

1. 编译客户端测试程序，客户端测试程序代码不需要修改，直接通过命令编译测试程序即可：

```bash
make test
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/a086bf899d583edfbebbc4fde9ddf8a5-0)

1. 运行验证

在 VS Code 平台下的终端窗口中先启动新编译好的 TCP 服务器：

```bash
./TcpServerDemo 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/086fc34951a2557fc5ac76a04def8fd3-0)

然后点击终端窗口上的 `+` 按键新建一个新的终端窗口，在其中启动一个客户端程序与服务器进行连接：

```bash
cd test
./TcpClient 127.0.0.1 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/728c075f9d548e4c86088054b0ac3ea3-0)

为了测试并发，我们再次新建一个终端窗口，再启动一个客户端，使得同时有两个客户端对服务器进行连接请求：

```bash
cd test
./TcpClient 127.0.0.1 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/728c075f9d548e4c86088054b0ac3ea3-0)

此时运行服务器的终端窗口会打印输出两个客户端连接，并且在这两个运行客户端程序的终端中都能实现 `echo` 的服务器功能运行：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/3b5a426c4ae2280c5c76768fe2464896-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/6ca9564aa757c630981f58ccb86ae4c1-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/58e5fd57dcfb49762359d7178f2dafbd-0)

## 5 实验总结

通过本节实验的学习，对 Linux 平台下的进程创建和多进程操作以及信号处理有了一定了解，结合到网络编程上，我们通过使用简单的多进程方式实现了并发服务器的创建，此时就能够使得服务器能够同时对多个客户端进行任务处理和请求响应。虽然采用多进程的方式会使得程序资源占用很大，但是基本功能是实现了，后面的章节就会针对资源占用问题提出更好的并发服务器实现方案。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code6.zip
```

# 七 IO 复用在服务器端的使用

## 1 实验介绍

通过本小节的内容讲解及部分示例程序演示，我们能够在 Linux 平台下不使用多进程或者多线程，甚至不是并发模式来实现多客户端任务处理的服务器。这就用到了 Linux 下的 I/O 复用，通过 `select` 函数来达到对众多客户端套接字的监控，在数据事件发生后去轮询处理，虽然细看客户端在发出数据后不会使得服务器在第一时间处理消息，但是也不会使得效率太低，并且相比多进程服务器来说占用系统资源很少，所以在很多应用场合中，I/O 复用的方式实现对服务器来说很有用，接下来我们就来学习本章关于 I/O 复用的知识内容。

#### 知识点

- 基于 I/O 复用函数的讲解
- 基于 I/O 复用的服务器端代码实现
- 理解 select 函数使用并修改 TCP 服务器项目实现 I/O 复用操作



## 2 基于 I/O 复用函数的讲解

前一章节的实验介绍了通过多进程的方式实现了服务器的并发响应操作，但是采用多进程的方式会导致随着客户端连接数的上升，程序所占内存和资源会急剧变大，这很明显对于服务器来说不是一个很好地解决办法。如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/e8114fb753f2ffcda871841261f6bf72-0)

其实在 Linux 系统下，还有一种采用 I/O 复用的方式实现同样的并发服务器的操作。 如下图所示，本质上同一时间服务器也只是会对一个客户端的请求数据进行处理，但是能够使得其他客户端在这个时间等待，等服务器处理完一个任务再继续处理别的请求任务，这个过程中所有的客户端都与服务器保持连接，由于处理时间很快，所以客户端感觉上都能够正常与服务器通信。所以这种方式其实严格上来说是一种“假并发”。类似 CPU 的时间片轮转处理方式。

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/1394efcc70f48746c34530063f85c6a3-0)



## 3 select 函数理解

在 Linux 系统下运用 `select()` 函数可以将多个文件描述符（套接字描述符）集中到一个集合中并统一监视，当发现在这个集合中的文件描述符有事件发生就会使得程序去得到当前有数据接收到的文件描述符，然后再去执行数据读取，不再像前面操作的那样阻塞等待数据接收。下图是 `select()` 函数详细的调用过程：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/3e0c6fe2523843de87827d02ce7b9ade-0)

接下来，我们就来一步一步详细介绍 `select()` 函数的使用。

1. 设置文件描述符

前面已经说过，`select()` 函数是通过监控一个文件描述符集合来获取有事件发生的文件描述符，所以最先需要设置文件描述符。使用 `fd_set` 数组变量执行设置文件描述符的操作，如下图所示，这个数组中是保存有 0 和 1 的位数组。

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/b8fba3fdabe7e8df9c356b1b0e54e104-0)

在这个结构中，最左端的位表示文件描述符 0 的位置，如果这一位设置为 1 ，则表示文件描述符 0 作为 `select()` 的监视对象。像上图中现在的状态就是说明现在监视文件描述符 1 和 3 的状态。

要设置 `fd_set` 变量的方式其实是要依赖一系列 Linux 下的宏：

- `FD_ZERO(fd_set *fdset)` ：将 fd_set 变量所有位清零。
- `FD_SET(int fd, fd_set *fdset)` ：在参数 fd_set 中注册文件描述符 fd 的信息。
- `FD_CLR(int fd, fd_set *fdset)` ：在参数 fd_set 中清除文件描述符 fd 的信息。
- `FD_ISSET(int fd, fd_set *fdset)` ：如果参数 fd_set 中已经注册了文件描述符 fd ，则返回 “真”。

1. 设置监视文件描述符范围和超时

`select()` 函数的详细描述如下：

```C++
#include <sys/select.h>
#include <sys/time.h>

//成功返回大于0的发生事件的文件描述符数量，失败返回-1
int select(int maxfd, fd_set *readset, fd_set *writeset, fd_set *exceptset, const struct tieval *timeout);

/*
- maxfd 监视文件描述符数量
- readset 关注是否存在待读数据的 fd_set 数组指针
- writeset 关注是否存在待发送无阻塞的 fd_set 数组指针
- exceptset 将所有检测到发生关注事件的文件描述符存放到 fd_set 数组中
- timeout 设置超时时间，放置 select 函数阻塞等待
*/
```

从上面的 `select()` 函数介绍中可以看出，文件描述符的监视范围设置与第一个参数 `maxfd` 有关，而超时设置与 `timeout` 参数有关。

- `maxfd` 参数设置

计算当前程序需要监视的文件描述符数量，其实在设置 `fd_set` 的时候进行数量统计就可以了，如果当前所有打开的文件描述符都需要监控，那么只需要将最大的文件描述符数值加一即可。

- `timeout` 参数设置

`timeout` 参数的结构如下：

```C++
struct timeval
{
    long tv_sec;    //秒
    long tv_usec;   //微秒
}
```

将超时时间传递给声明好的 `timeout` 变量即可完成对 `select()` 函数超时设置。

1. 调用 `select()` 函数后查看结果

在设置好 `select()` 调用的参数后，直接调用 `select()` 函数去查看当前监控的文件描述符的状态。在 `select()` 函数返回正整数后，它能够给查看其中第二个参数到第四个参数的数值变化就能得到对应位置的文件描述符发生了事件。如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/45fcaf9e792af89265cbe94bd854651d-0)

由图可知，在函数调用完成后，传递的 `fd_set` 参数将会发生变化，其中只保留发生事件的文件描述符对应位，其余全部会被清空。

1. select 函数调用例程

复制 `/home/project/code6` 文件夹，并命名为 `/home/project/code7` 文件夹。接下来的内容统一在 `code7` 中进行修改。

下面，通过实现一个简单的演示例程来熟悉一下 `select()` 函数的具体使用方法，在我们的项目 `TCP-ServerDemo` 下的 `tools` 目录中创建例程源代码文件 `selectDemo.cpp` ，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/9424729b78eadf43f6cff5bb4121d157-0)

文件创建完成后，在其中添加以下代码：

```C++
#include <iostream>
#include <unistd.h>
#include <sys/time.h>
#include <sys/select.h>
#include <cstring>

#define BUFFSIZE    1024

int main(int argc, char *argv[])
{
    fd_set reads, temps;
    int result, str_len;
    char buf[BUFFSIZE];
    struct timeval timeout;
    FD_ZERO(&reads);
    FD_SET(0, &reads);


    while(1)
    {
        temps = reads;
        //设置超时时间 5.5 s
        timeout.tv_sec = 5;
        timeout.tv_usec = 5000;
        //清空缓冲区内容
        memset(buf, 0, BUFFSIZE);

        result = select(1, &temps, 0, 0, &timeout);
        if(result < 0)
        {
            std::cout << "select() error. " << std::endl;
            break;
        }
        else if(result == 0)
        {
            //超时
            std::cout << "Time-out!" << std::endl;
            continue;
        }
        //判断是否是标准输入有数据到
        if(FD_ISSET(0, &temps))
        {
            str_len = read(0, buf, BUFFSIZE);
            buf[str_len] = 0;
            std::cout << "Message from console : " << buf << std::endl;
        }
    }

    return 0;
}
```

代码添加完毕后，在 VS Code 平台的终端窗口中使用命令编译源代码。

```bash
cd /home/project/code7/TCP-ServerDemo/tools
g++ selectDemo.cpp -o selectDemo
./selectDemo
```

执行结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/f891cae3517bda00cdb098f6ed54abc6-0)

从终端输出可以看到，当我们向终端输入传输数据时，程序就能够通过 `select()` 函数观察到有数据待读取，于是去读取数据然后打印输出。当程序长时间没有消息进入，就会进入超时响应，会打印输出 `Time-out!` 的超时消息。

## 4 修改 TCP 服务器项目进行 I/O 复用优化

通过前面的例程代码，我们基本上了解到如何使用 `select()` 来监听当前进程所等待数据的文件描述符信息，接下来就将 I/O 复用的方式用在我们一直维护的 TCP 服务器项目 `TCP-ServerDemo` 中。

前一章节已经将项目代码修改为多进程方式的服务器，类似前面图示模型的情况。基于目前我们学习到的内容，如果使用 I/O 复用的方式实现多客户端任务响应，只能暂时将代码修改回单进程模式。

打开项目 `TCP-ServerDemo` 源代码文件 `src/main.cpp` ，在其中修改代码成如下内容：

```C++
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <cstring>
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/select.h>
#include "base.h"

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}


//子进程处理客户端请求函数封装
int ChildProcessWork(int ServSocket, int ChildSocket)
{
    char buf[BUFFSIZE] = {0};
    int str_len;
    //首先在子进程中关闭服务端套接字
    close(ServSocket);

    while(1)
    {
        //接收客户端数据
        str_len = read(ChildSocket, buf, BUFFSIZE);
        if(str_len == 0)
        {
            //客户端断开连接，处理结束
            break;
        }
        //多进程服务中不再将受到数据打印输出
        //将客户端数据返回给客户端
        write(ChildSocket, buf, str_len);
        memset(buf, 0, BUFFSIZE);
    }

    //关闭客户端套接字
    close(ChildSocket);
    return 0;
}

//处理子进程结束后，父进程回收资源的信号回调函数
void read_childproc(int signum)
{
    pid_t pid;
    int status;
    pid = waitpid(-1, &status, WNOHANG);
    std::cout << "Removed proc id : " << pid << std::endl;
}

//多进程工作
int fork_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    pid_t pid;

    //循环等待客户端请求
    while (1)
    {
        adr_size = sizeof(client_adr);
        client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
        if(client_sock < 0)
        {
            //接收请求失败，继续工作
            continue;
        }
        else{
            //接收到新的客户端，打印输出
            std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
            //创建子进程进行处理
            pid = fork();
            if(pid < 0)
            {
                //创建子进程失败，关闭连接
                std::cout << "fork error, close client" << std::endl;
                close(client_sock);
                continue;
            }
            if(pid == 0)
            {
                //进入子进程,
                ret = ChildProcessWork(serv_sock, client_sock);
                //调用结束后直接结束子进程
                exit(ret);
            }
            else{
                //父进程关闭客户端套接字，继续等待新客户端请求
                close(client_sock);
            }
        }
    }
    return 0;
}

//select I/O复用
int select_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    int fd_max, fd_num;
    fd_set reads, cpy_reads;
    struct timeval timeout;
    char buf[BUFFSIZE];
    int str_len;

    FD_ZERO(&reads);
    FD_SET(serv_sock, &reads);
    fd_max = serv_sock;

    while(1)
    {
        cpy_reads = reads;
        timeout.tv_sec = 5;
        timeout.tv_usec = 5000;

        fd_num = select(fd_max + 1, &cpy_reads, 0, 0, &timeout);
        if(fd_num < 0)
        {
            //调用select 失败
            std::cout << "select error." << std::endl;
            break;
        }
        else if(fd_num == 0)
        {
            //目前没有数据事件发生
            continue;
        }
        //循环处理当前所有可读事件
        for(int i = 0; i < fd_max + 1; i++)
        {
            if(FD_ISSET(i, &cpy_reads))
            {
                //如果当前要处理的文件描述符是 serv_sock 服务器套接字，则表示需要处理客户端请求连接事件
                if(i == serv_sock)
                {
                    adr_size = sizeof(client_adr);
                    client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
                    //打印新连接客户端信息
                    std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
                    //将新连接的客户端注册到select处理队列中
                    FD_SET(client_sock, &reads);
                    //如果当前最大值小于新客户端套接字，则改变
                    if(fd_max < client_sock)
                    {
                        fd_max = client_sock;
                    }
                }
                else
                {
                    //处理客户端发来的消息，echo服务器负责将数据返回
                    str_len = read(i, buf, BUFFSIZE);
                    if(str_len == 0)
                    {
                        //客户端连接断开
                        //将客户端套接字从select监控队列中清除
                        FD_CLR(i, &reads);
                        close(i);
                        //输出打印
                        std::cout << "Closed Client : " << i << std::endl;
                    }
                    else
                    {
                        //正常处理数据，将读取到的数据返回给客户端
                        write(i, buf, str_len);
                    }

                }
            }
        }
    }

    return 0;
}

//修改主函数
int main(int argc, char *argv[])
{
    int serv_sock, ret;
    struct sockaddr_in serv_adr;
    struct sigaction act;

    int str_len, state;
    int option;
    socklen_t optlen;

    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << "<port> " << std::endl;
        exit(1);
    }

    act.sa_handler = read_childproc;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    state = sigaction(SIGCHLD, &act, 0);

    //创建服务socket套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //添加 Time-wait 状态重新分配断开设置
    optlen = sizeof(option);
    option = 1;
    setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, (void*)&option, optlen);


    //初始化服务套接字
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    //绑定套接字
    ret = bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr));
    if(ret < 0)
    {
        error_handle("bind", "bind() error.");
    }

    //监听端口
    ret = listen(serv_sock, 5);
    if(ret < 0)
    {
        error_handle("listen", "listen() error");
    }

    // fork_workproc(serv_sock);    //多进程方式实现
    //使用select I/O复用方式
    select_workproc(serv_sock);

    //关闭套接字
    close(serv_sock);
    return 0;
}
```

代码修改完成后，在 VS Code 平台中的终端窗口下执行 `make` 命令进行项目编译：

```bash
cd /home/project/code7/TCP-ServerDemo
make
```

编译截图如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/cc920b1eadde582b11d2b1b0617592a8-0)

验证修改后的服务器运行效果如下：

```bash
./TcpServerDemo 18888
# 新空一个终端
cd /home/project/code7/TCP-ServerDemo/test
./TcpClient 127.0.0.1 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/f3c6fb1947238d03c0c523bb98867616-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/6acdf90d6dc701335d0aa68c49952e6a-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/3efce8364c0e73775211b92ff75b5cce-0)

关闭客户端后：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/3943cf0ab83d510ede47735be1295390-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/b4614fde1ac60c21b466968eed163258-0)

## 5 实验总结

通过本节实验的学习，我们了解能够在单线程模式下使用 `select` 进行 I/O 复用的方式实现多客户端任务处理的服务器实现。对于 Linux 平台下的网络编程的知识掌握更深入了一层，在后面的章节中会将目前所学到的各个部分的内容组合起来，这样就构成了在商业上经常使用的服务器实现方式。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code7.zip
```

# 八 epoll 方式优化 IO 复用

## 1 实验介绍

在 Unix 平台下的网络编程中，实现 I/O 复用的方式其实是有 `select` 和 `epoll` 、 `poll` 几种方式，前面已经介绍并使用了 `select` 方式使用 I/O 复用实现了服务器的多客户端任务处理，本章将介绍优于 `select` 的 `epoll` 方式。通过本章的学习，能够加深对 Linux 下的 I/O 复用相关知识的理解和掌握，使用 `epoll` 方式修改前面的服务器项目能够从代码层面上更好地理解与 `select` 方式的不同和有点。

#### 知识点

- 优于 `select` 的 `epoll` 方式介绍
- 条件触发和边缘触发
- 使用 `epoll` 方式修改服务器项目代码



## 2 epoll 方式介绍

前一章节已经学习了 `select` 方式实现 I/O 复用，其实 `select` 方式是通过依次去查看当前所监控的文件描述符（套接字）组中的成员是否有可执行的操作来获得当前能够去进行操作的文件描述符（套接字），但是这样的操作在运行时速度会很慢的，主要原因是：

- 调用 `select` 函数后必然会执行针对所以可用文件描述符的循环语句；
- 每次调用 `select` 函数时都需要向该函数传递监视对象信息，需要反复初始化参数进行传递；

因此，通常在服务器开发中，尤其是 web 服务器开发中，`select` 方式通常并不会采用，而是采用速度更快的 `epoll` 方式来实现 I/O 复用操作。但是，`select` 方式还是有很大的用处的，`epoll` 方法虽然好却只有 Linux 平台下支持，而 `select` 方式是大部分操作系统都支持的，所以在很多场景下还是需要使用到 `select` 方式，所以大家还是需要对此有一定掌握和理解的。

现在就正式介绍 `epoll` 方式的概念和理解。`epoll` 之所以要优于 `select` 方式，其实主要是有以下优点：

- 无需编写用来监控状态变化为目的的针对所有文件描述符的循环语句；
- 调用对应于 `select` 函数的 `epoll_wait` 函数不需要反复传递监控对象的信息；

使用 `epoll` 方式的主要操作如下：

```C++
//创建保存epoll文件描述符的空间，类似创建select方式下的fd_set变量
int epoll_create(int size);

//注册或者注销监视的文件描述符，类似select方式下的FD_SET、FD_CLR操作
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
/*
epfd ：由epoll调用产生的文件描述符
op ：控制操作参数，有 EPOLL_CTL_ADD EPOLL_CTL_MOD EPOLL_CTL_DEL，
fd ：操作对象
event ：由以下宏组成：
    EPOLLIN：表示对应文件描述符可读
    EPOLLOUT：表示对应文件描述符可写
    EPOLLPRI：表示对应文件描述符有紧急书籍可读
    EPOLLERR：表示对应文件描述符发生错误
    EPOLLHUP：表示对应文件描述符被挂断
    EPOLLET：设置边缘触发模式
    EPOLLONESHOT：表示仅对该描述符实施一次监听操作
*/


//epoll函数的特点处，不同于select方式去轮询探查监视对象，epoll是等待事件触发
int epoll_wait(int epfd, struct epoll_event *event, int maxevents, int timeout);
```

需要注意，在 `epoll` 下需要使用到特别的 `epoll_event` 结构体反映发送变化的文件描述符，具体内容如下：

```C++
typedef union epoll_data
{
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
}epoll_data_t;

struct epoll_event
{
    __uint32_t events;
    epoll_data_t data;
}
```

其实，`epoll` 方式的学习在理解了 `select` 基础上并不难，主要是对其中的三个函数的使用清楚和理解后就很简单了。

```C++
#include <sys/epoll.h>

//成功返回 epoll 文件描述符， 失败返回 -1
int epoll_create(int size);
/*
size ：epoll 实例大小
*/
```

这里虽然说 `size` 参数是 `epoll` 实例大小，但是也只是向操作系统提出的建立，具体创建的大小还是由操作系统来决定的。`epoll_create` 函数创建的 `epoll` 文件描述符其实类似套接字资源一样，都是由系统内核来管理的，所以在最后不再使用时也需要调用 `close()` 函数来结束资源的使用。

```C++
#include <sys/epoll.h>

//成功返回0，失败返回-1
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
/*
epfd：用于注册监视对象的epoll文件描述符，前面epoll_create函数的返回值
op：操作参数，传递监视对象的添加、删除、更改的操作
    EPOLL_CTL_ADD：将文件描述符注册到epoll例程中
    EPOLL_CTL_DEL：从epoll例程中删除文件描述符
    EPOLL_CTL_MOD：更改注册的文件描述符的关注事件发生情况
fd：需要操作的监视对象文件描述符
event：监视对象的事件类型。通过初始化一个 epoll_event 结构体的变量完成注册事件的传递，其中event.events成员变量由下面内容组成：
    EPOLLIN：需要读取数据的情况
    EPOLLOUT：输出缓冲为空，可以立即发送数据的情况
    EPOLLPRI：收到 OOB 数据的情况
    EPOLLRDHUP：断开连接或者半关闭的情况，在边缘触发方式下经常使用
    EPOLLERR：发送错误的情况
    EPOLLET：以边缘触发的方式得到事件通知
    EPOLLONESHOT：发送一次事件后，对应的文件描述符不再收到事件通知，如果还需要获取事件通知，需要使用 epoll_ctl 再次设置事件
*/
#include <sys/epoll.h>

//成功返回发生事件的文件描述符，失败返回-1
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
/*
epfd：epoll例程的文件描述符
events：保存发生事件的文件描述符集合的结构体地址，该参数所指向的缓冲需要动态分配
maxevents：第二个参数中科院保存的最大事件数量
timeout：以微秒为单位的等待时间，设置为-1则表示持续等待
*/
```



## 3 epoll 函数调用例程

复制 `/home/project/code7` 文件夹，并命名为 `/home/project/code8` 文件夹。接下来的内容统一在 `code8` 中进行修改。

下面，通过类似之前 `select` 调用例程来演示使用 `epoll` 方式实现相同例程。在我们的项目 `TCP-ServerDemo` 下的 `tools` 目录中创建例程源代码文件 `epollDemo.cpp` ，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/72039a38321985421ac17e3fadfcf1c8-0)

文件创建完成后，在其中添加以下代码：

```C++
#include <iostream>
#include <unistd.h>
#include <cstring>
#include <sys/time.h>
#include <sys/epoll.h>

#define BUFFSIZE    1024

int main(int argc, char *argv[])
{

    char buf[BUFFSIZE];
    int timeout;
    int epfd, epoll_size, event_cnt;
    struct epoll_event *ep_events;
    struct epoll_event event;
    int str_len;

    //设置超时时间 5.5 s , 5500 微秒
    timeout = 5500;

    //创建epoll 例程，size = 1
    epoll_size = 2;
    epfd = epoll_create(epoll_size);
    if(epfd < 0)
    {
        //创建失败
        std::cout << "epoll_create error." << std::endl;
        return -1;
    }

    //创建用于epoll_wait事件发生的事件结构体缓冲区
    ep_events = (struct epoll_event *)malloc(sizeof(struct epoll_event)*epoll_size);

    //初始化文件描述符事件注册参数
    event.events = EPOLLIN; //监视事件为有数据输入/可读取状态
    event.data.fd = 0;  //监视文件描述符为标准输入
    //设置事件注册
    epoll_ctl(epfd, EPOLL_CTL_ADD, 0, &event);

    while(1)
    {
        //清空缓冲区内容
        memset(buf, 0, BUFFSIZE);

        event_cnt = epoll_wait(epfd, ep_events, epoll_size, timeout);
        if(event_cnt < 0)
        {
            std::cout << "epoll_wait() error!" << std::endl;
            break;
        }
        else if(event_cnt == 0)
        {
            std::cout << "epoll_wait() timeout!" << std::endl;
            continue;
        }
        else{
            //这里只注册了一个文件描述符，所以不需要循环处理，通常还是需要循环处理所有返回的事件消息
            //基础判断,发生事件的文件描述符是标准输入
            if(ep_events[0].data.fd == 0)
            {
                //读取数据并打印输出
                str_len = read(0, buf, BUFFSIZE);
                buf[str_len] = 0;
                std::cout << "Message from console : " << buf << std::endl;
            }

        }
    }
    //关闭epfd
    close(epfd);

    return 0;
}
```

代码添加完毕后，在 VS Code 平台的终端窗口中使用命令编译源代码。

```bash
cd /home/project/code8/TCP-ServerDemo/tools
g++ epollDemo.cpp -o epollDemo
./epollDemo
```

执行结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/d8cf5179bf2cebff2bbfc6936aceee59-0)

从终端输出可以看到，当我们向终端输入传输数据时，程序就能够通过 `epoll_wait()` 函数等待事件发生，当函数返回后就会去处理返回的结构体中的文件描述符，这里判断有数据可以读取，所以执行读取数据然后打印输出。当程序长时间没有消息进入，就会进入超时响应，会打印输出 `Time-out!` 的超时消息。



## 4 条件触发和边缘触发

`epoll` 方式主要是依靠系统内核对事件触发后的消息返回，但是这里需要明白条件触发和边缘触发的关系。条件触发表示当前监视的文件描述符满足注册条件就会发送通知消息（epoll_wait 会得到该文件描述符），而边缘触发意味着仅会在事件发生时注册消息（仅注册一次）。

可以将前面的例程修改读取数据缓冲大小后，发现如果仅使用条件触发后，只要输入缓冲区中还有数据就会在 `epoll_wait` 函数中返回这个文件描述符（实际它并没有什么状态变化，只不过是上次数据处理不够完全），新建 `tools/epollNOletDemo.cpp` 文件，向其中添加如下代码内容：

```C++
#include <iostream>
#include <unistd.h>
#include <sys/time.h>
#include <sys/epoll.h>
#include <cstring>

#define BUFFSIZE    3

int main(int argc, char *argv[])
{

    char buf[BUFFSIZE];
    int timeout;
    int epfd, epoll_size, event_cnt;
    struct epoll_event *ep_events;
    struct epoll_event event;
    int str_len;

    //设置超时时间 5.5 s , 5500 微秒
    timeout = 5500;

    //创建epoll 例程，size = 1
    epoll_size = 2;
    epfd = epoll_create(epoll_size);
    if(epfd < 0)
    {
        //创建失败
        std::cout << "epoll_create error." << std::endl;
        return -1;
    }

    //创建用于epoll_wait事件发生的事件结构体缓冲区
    ep_events = (struct epoll_event *)malloc(sizeof(struct epoll_event)*epoll_size);

    //初始化文件描述符事件注册参数
    event.events = EPOLLIN; //监视事件为有数据输入/可读取状态
    event.data.fd = 0;  //监视文件描述符为标准输入
    //设置事件注册
    epoll_ctl(epfd, EPOLL_CTL_ADD, 0, &event);

    while(1)
    {
        //清空缓冲区内容
        memset(buf, 0, BUFFSIZE);

        event_cnt = epoll_wait(epfd, ep_events, epoll_size, timeout);
        if(event_cnt < 0)
        {
            std::cout << "epoll_wait() error!" << std::endl;
            break;
        }
        else if(event_cnt == 0)
        {
            std::cout << "epoll_wait() timeout!" << std::endl;
            continue;
        }
        else{
            //添加正常调用打印
            std::cout << "epoll_wait() sucess" << std::endl;
            //这里只注册了一个文件描述符，所以不需要循环处理，通常还是需要循环处理所有返回的事件消息
            //基础判断,发生事件的文件描述符是标准输入
            if(ep_events[0].data.fd == 0)
            {
                //读取数据并打印输出
                str_len = read(0, buf, BUFFSIZE);
                buf[str_len] = 0;
                std::cout << "Message from console : " << buf << std::endl;
            }

        }
    }
    //关闭epfd
    close(epfd);

    return 0;
}
```

如果在这里使用边缘触发设置，也就是所监视的文件描述符状态变化后仅执行事件注册一次。新建 `tools/epollLETDemo.cpp` 文件，并向其中写入如下代码内容：

```C++
#include <iostream>
#include <unistd.h>
#include <sys/time.h>
#include <sys/epoll.h>
#include <cstring>

#define BUFFSIZE    3

int main(int argc, char *argv[])
{

    char buf[BUFFSIZE];
    int timeout;
    int epfd, epoll_size, event_cnt;
    struct epoll_event *ep_events;
    struct epoll_event event;
    int str_len;

    //设置超时时间 5.5 s , 5500 微秒
    timeout = 5500;

    //创建epoll 例程，size = 1
    epoll_size = 2;
    epfd = epoll_create(epoll_size);
    if(epfd < 0)
    {
        //创建失败
        std::cout << "epoll_create error." << std::endl;
        return -1;
    }

    //创建用于epoll_wait事件发生的事件结构体缓冲区
    ep_events = (struct epoll_event *)malloc(sizeof(struct epoll_event)*epoll_size);

    //初始化文件描述符事件注册参数
    event.events = EPOLLIN|EPOLLET; //监视事件为有数据输入/可读取状态 ,添加边缘触发设置
    event.data.fd = 0;  //监视文件描述符为标准输入
    //设置事件注册
    epoll_ctl(epfd, EPOLL_CTL_ADD, 0, &event);

    while(1)
    {
        //清空缓冲区内容
        memset(buf, 0, BUFFSIZE);

        event_cnt = epoll_wait(epfd, ep_events, epoll_size, timeout);
        if(event_cnt < 0)
        {
            std::cout << "epoll_wait() error!" << std::endl;
            break;
        }
        else if(event_cnt == 0)
        {
            std::cout << "epoll_wait() timeout!" << std::endl;
            continue;
        }
        else{
            //添加正常调用打印
            std::cout << "epoll_wait() sucess" << std::endl;
            //这里只注册了一个文件描述符，所以不需要循环处理，通常还是需要循环处理所有返回的事件消息
            //基础判断,发生事件的文件描述符是标准输入
            if(ep_events[0].data.fd == 0)
            {
                //读取数据并打印输出
                str_len = read(0, buf, BUFFSIZE);
                buf[str_len] = 0;
                std::cout << "Message from console : " << buf << std::endl;
            }

        }
    }
    //关闭epfd
    close(epfd);

    return 0;
}
```

编译两个修改后的例程代码：

```bash
cd /home/project/code8/TCP-ServerDemo/tools
g++ epollNOletDemo.cpp -o epollNOletDemo
g++ epollLETDemo.cpp -o epollLETDemo
./epollNOletDemo
./epollLETDemo
```

执行结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/b847986c97b5c8e31e5ab3c21d29cf4e-0)

从结果中可以看出，由于每次数据处理都没能将待读取数据完全读取完全，所以条件触发会持续注册事件使程序进行继续的处理；而添加了边缘触发参数后，仅在状态改变时注册一次事件，所以 `epollLETDemo` 执行后每次仅会在有新数据时处理消息，所以只会打印输出 3 个字节的数据。通过这样的对比，大家应该对条件触发和边缘触发有了清楚的认识了吧。（`select` 方式是以条件触发的方式工作的）

## 5 使用 epoll 方式修改服务器项目代码

通过前面的例程代码，我们基本上了解到如何使用 `epoll()` 来监听当前进程所等待数据的文件描述符信息，接下来就优化厚的 I/O 复用的方式用在我们一直维护的 TCP 服务器项目 `TCP-ServerDemo` 中。

前一章节使用了 `select` 方式实现了 I/O 复用，这里直接使用 `epoll` 方式进行替换。

打开项目 `TCP-ServerDemo` 源代码文件 `src/main.cpp` ，在其中修改代码成如下内容：

```C++
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <cstring>
#include <fcntl.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/select.h>
#include <sys/epoll.h>
#include "base.h"

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

//设置套接字非阻塞
void setnonblockingmode(int fd)
{
    int flag = fcntl(fd, F_GETFL, 0);
    fcntl(fd, F_SETFL, flag|O_NONBLOCK);
}

//epoll I/O 复用方式
int epoll_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    int epoll_size, epfd, event_cnt;
    struct epoll_event *ep_events;
    struct epoll_event event;
    int timeout;
    int str_len;
    char buf[BUFFSIZE];

    //初始化部分参数，创建epoll 例程
    epoll_size = 50;    //默认定义50作为监视文件描述符数量，可修改
    epfd = epoll_create(epoll_size);
    if(epfd < 0)
    {
        //创建失败，退出程序
        std::cout << "epoll_create error." << std::endl;
        return -1;
    }
    //申请epoll_wait 返回缓冲区
    ep_events = (struct epoll_event *)malloc(sizeof(struct epoll_event) * epoll_size);

    //设置服务器套接字非阻塞
    setnonblockingmode(serv_sock);
    //优先将服务器套接字注册到epoll监视例程中
    event.events = EPOLLIN; //服务器套接字不使用边缘触发
    event.data.fd = serv_sock;
    //添加文件描述符及事件
    epoll_ctl(epfd, EPOLL_CTL_ADD, serv_sock, &event);

    //初始化超时时间为 5.5 s
    timeout = 5500;

    while(1)
    {
        //调用 epoll_wait 获取当前文件描述符注册事件
        event_cnt = epoll_wait(epfd, ep_events, epoll_size, timeout);
        if(event_cnt < 0)
        {
            //调用select 失败
            std::cout << "epoll_wait error." << std::endl;
            break;
        }
        else if(event_cnt == 0)
        {
            //目前没有数据事件发生，超时
            continue;
        }
        //循环处理当前所有待处理事件
        for(int i = 0; i < event_cnt; i++)
        {
            if(ep_events[i].data.fd == serv_sock)//判断事件文件描述符是服务器套接字
            {
                //如果当前要处理的文件描述符是 serv_sock 服务器套接字，则表示需要处理客户端请求连接事件
                adr_size = sizeof(client_adr);
                client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
                //打印新连接客户端信息
                std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
                //初始化 event 变量，注册到 epoll 监视列表中
                event.events = EPOLLIN | EPOLLET;  //添加边缘触发条件
                event.data.fd = client_sock;
                setnonblockingmode(client_sock);    //设置文件描述符非阻塞
                epoll_ctl(epfd, EPOLL_CTL_ADD, client_sock, &event);
            }
            else
            {
                //由于添加了边缘触发，所以一次操作就需要将输入缓冲区中数据全部处理完成
                while(1)
                {
                    //清除缓存区
                    memset(buf, 0, BUFFSIZE);
                    //处理客户端发来的消息，echo服务器负责将数据返回
                    str_len = read(ep_events[i].data.fd, buf, BUFFSIZE);
                    if(str_len == 0)
                    {
                        //客户端断开连接，将该客户端套接字从监视列表中删除
                        epoll_ctl(epfd, EPOLL_CTL_DEL, ep_events[i].data.fd, NULL);
                        close(ep_events[i].data.fd);
                        //输出打印
                        std::cout << "Closed Client : " << ep_events[i].data.fd << std::endl;
                        break;
                    }
                    else if(str_len < 0)
                    {
                        if(errno == EAGAIN)
                        {
                            //数据读取完毕
                            break;
                        }
                    }
                    else
                    {
                        //正常处理数据，将读取到的数据返回给客户端
                        write(ep_events[i].data.fd, buf, str_len);
                    }


                }
            }
        }
    }

    //关闭epoll 例程
    close(epfd);

    return 0;
}

//处理子进程结束后，父进程回收资源的信号回调函数
void read_childproc(int signum)
{
    pid_t pid;
    int status;
    pid = waitpid(-1, &status, WNOHANG);
    std::cout << "Removed proc id : " << pid << std::endl;
}

//修改主函数
int main(int argc, char *argv[])
{
    int serv_sock, ret;
    struct sockaddr_in serv_adr;
    struct sigaction act;

    int str_len, state;
    int option;
    socklen_t optlen;

    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << "<port> " << std::endl;
        exit(1);
    }

    act.sa_handler = read_childproc;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    state = sigaction(SIGCHLD, &act, 0);

    //创建服务socket套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //添加 Time-wait 状态重新分配断开设置
    optlen = sizeof(option);
    option = 1;
    setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, (void*)&option, optlen);


    //初始化服务套接字
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    //绑定套接字
    ret = bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr));
    if(ret < 0)
    {
        error_handle("bind", "bind() error.");
    }

    //监听端口
    ret = listen(serv_sock, 5);
    if(ret < 0)
    {
        error_handle("listen", "listen() error");
    }

    // //多进程方式实现
    // fork_workproc(serv_sock);
    // //select I/O 复用方式
    // select_workproc(serv_sock);

    //epoll I/O 复用方式
    epoll_workproc(serv_sock);

    //关闭套接字
    close(serv_sock);
    return 0;
}
```

代码修改完成后，在 VS Code 平台中的终端窗口下执行 `make` 命令进行项目编译：

```bash
cd /home/project/code8/TCP-ServerDemo
make
make test
```

编译截图如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/003e2d9e27e5a31c75c2842f9b04aab7-0)

验证修改后的服务器运行效果如下：

```bash
cd /home/project/code8/TCP-ServerDemo
./TcpServerDemo 18888
# 新开一个终端
cd /home/project/code8/TCP-ServerDemo/test
./TcpClient 127.0.0.1 18888
# 再新开一个终端，输入不同的内容
cd /home/project/code8/TCP-ServerDemo/test
./TcpClient 127.0.0.1 18888
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/523b1f10a9081d445d8ccda09a323d83-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/079e80f084ec035b3ac777cf577d9166-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/f61322d29861afce09ad7ca1aed61f38-0)

客户端断开连接后：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/aa396f8f6b67639da55db53899150005-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/3afff65d2249c78ecd26f9ce2b27f773-0)



## 6 实验总结

通过本节实验的学习，我们学习到了比 `select` 更好实现 I/O 复用的 `epoll` 方式，通过该方式的修改使得服务器处理速度能够进一步提升。后面，会在此基础上将各个内容组合起来，这样就构成了在商业上经常使用的服务器实现方式。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code8.zip
```

# 九 多线程服务器实现

## 1 实验介绍

通过多进程方式已经能够实现服务器的并发处理，但是多进程天生占用计算机资源较大，于是在 Linux 平台下引进了多线程操作来解决系统同步处理的任务而又不会像进程那样占用资源过大。本章就通过引入 Linux 下的多线程操作来实现服务器的并发处理功能。

#### 知识点

- Linux 线程概念
- Linux 下线程创建及运行
- 多线程存在的问题
- 线程同步及线程销毁
- 多线程并发服务器的实现



## 2 Linux 线程概念

在前面的学习中，我们已经学习并掌握了 Linux 下的进程操作和多进程实现并发服务器的方式，然而在 Linux 平台下除了通过进程方式实现并发效果外，还可以通过线程的方式实现。并且与进程相比，线程会占有资源更小。我们都知道现在的计算机处理器芯片（CPU）中其实会有很多个 CPU 核，并且每个 CPU 核同时能够实现一个线程到多个线程的工作，因此，为了能够使得程序在 CPU 中的处理速度更快速，我们经常会使用线程操作。

线程相比进程具有以下优点：

- 线程的创建和上下文切换比进程的创建和上下文切换更快。
- 线程间交换数据时不需要类似进程的 IPC 操作。

在使用多进程方式时，每个进程都会有独立的 PCB 空间，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/66e3b85d0eefa2933d9f155149b8657f-0)

而在一个单个进程中使用多线程操作时，其内存状态如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/87b213cdd8c72b6984d89460612b1539-0)

由此可以看出，在使用多线程操作时对系统的资源占用没有进程多，但是真正的程序执行流却比进程只多不少。



## 3 Linux 下线程创建及运行

为了方便 Unix 平台之间的程序移植诞生了 POSIX （适用于计算机环境可移植操作系统接口），它规定了一系列标准的操作系统使用 API 接口，其中就有关于线程创建和操作的相关函数接口。

每个线程的创建和执行都需要一个独立的执行入口，所以需要定义对应的线程入口函数，而创建并执行线程的操作如下：

```C++
#include <pthread.h>

// 成功返回0， 失败返回其他值
int pthread_create(pthread_t *restrict thread, const pthread_attr_t *restrict attr, void *(* start_routine)(void *), void *restrict arg);

/*
- thread 保存新创建线程 ID 的变量地址值，线程与进程相同，需要用于区分不同线程的 ID
- attr 用于传递线程属性参数，NULL 时表示创建默认属性线程
- start_routine 传递线程入口函数指针
- arg 传递给线程入口函数的参数信息的变量地址
*/
```

接下来，通过一个简单的线程创建例程来了解并掌握线程创建的函数接口使用。

复制 `/home/project/code8` 文件夹，并命名为 `/home/project/code9` 文件夹。接下来的内容统一在 `code9` 中进行修改。

创建 `tools/pthreadCreateDemo.cpp` 文件，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/b3f6241496109b231995e2c8cc07a4b2-0)

创建文件完成后，在其中添加如下代码：

```C++
#include <iostream>
#include <unistd.h>
#include <pthread.h>

void *thread_routine1(void *arg);
void *thread_routine2(void *arg);

int main(int argc, char *argv[])
{
    pthread_t t_id1, t_id2;
    int thread_param = 0;

    //创建线程1
    if(pthread_create(&t_id1, NULL, thread_routine1, (void*)&thread_param) != 0)
    {
        std::cout << "pthread create error" << std::endl;
        return -1;
    }
    //创建线程2
    if(pthread_create(&t_id2, NULL, thread_routine2, (void*)&thread_param) != 0)
    {
        std::cout << "pthread create error" << std::endl;
        return -1;
    }
    //等待1秒
    sleep(1);
    std::cout << "Main Start ..." << std::endl;
    //给参数赋值，使线程正式工作
    thread_param = 5;
    sleep(5);

    std::cout << "Main End ..." << std::endl;

    return 0;
}

void *thread_routine1(void *arg)
{
    int *cnt = (int *)arg;
    //当参数为0时，循环等待
    while(*cnt == 0)

    for(int i = 0; i < *cnt; ++i)
    {
        sleep(1);
        std::cout << "Thread 1 Runing..." << std::endl;
    }

    return NULL;
}

void *thread_routine2(void *arg)
{
    int *cnt = (int *)arg;
    //当参数为0时，循环等待
    while(*cnt == 0)

    for(int i = 0; i < *cnt; ++i)
    {
        sleep(1);
        std::cout << "Thread 2 Runing..." << std::endl;
    }

    return NULL;
}
```

在程序编译的时候需要注意添加编译参数，因为我们用到了多线程操作，所以在使用编译器编译时需要添加 `-pthread` 编译参数，具体如下：

```bash
cd /home/project/code9/TCP-ServerDemo/tools
g++ pthreadCreateDemo.cpp -pthread -o pthreadCreateDemo
./pthreadCreateDemo
```

编译完成后，运行结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/dbc5a37bd815d35aef010b4aa176f5db-0)



## 4 多线程存在的问题

由于一个进程中的多个线程是都有权限去访问全局变量的，当多个线程同时访问同一个全局变量时就让容易导致结果不可预测，接下来通过一个实例程序来观察一下这个现象。

创建新的源文件 `threadBugDemo.cpp` ，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/fd4335a8be4f9aa066d48866b992e686-0)

创建完成后，在其中添加如下代码：

```C++
#include <iostream>
#include <pthread.h>

#define NUM_THREAD 100

//自增线程
void *thread_inc(void *arg);
//自减线程
void *thread_des(void *arg);
//全局变量
long long num = 0;

int main(int argc, char *argv[])
{
    pthread_t thread_id[NUM_THREAD];

    //创建50个加线程，创建50个减线程
    for(int i = 0; i < NUM_THREAD; ++i)
    {
        if(i%2)
            pthread_create(&(thread_id[i]), NULL, thread_inc, NULL);
        else
            pthread_create(&(thread_id[i]), NULL, thread_des, NULL);
    }

    //回收这100个线程资源
    for(int i = 0; i< NUM_THREAD; ++i)
    {
        pthread_join(thread_id[i], NULL);
    }

    //打印输出结果
    std::cout << "Result : " << num << std::endl;
    return 0;
}

//自增线程
void *thread_inc(void *arg)
{
    for(int i = 0; i < 3000; ++i)
    {
        num++;
    }
    return NULL;
}
//自减线程
void *thread_des(void *arg)
{
    for(int i = 0; i < 3000; ++i)
    {
        num--;
    }
    return NULL;
}
```

代码添加完成后，使用如下命令编译：

```bash
cd /home/project/code9/TCP-ServerDemo/tools
g++ threadBugDemo.cpp -lpthread -o threadBugDemo
./threadBugDemo
```

程序运行结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/a492f68b184de06ba9b7ace7b71f8252-0)

本来，按照我们的想法，这个程序最终结果应该是 0 ，因为执行同样数量的加法操作和减法操作后结果应该是不会变的，但是执行多次程序发现执行结果不但不会是 0 ，而且还不是一个确定的值。这是为什么呢？

其实主要原因就是两个线程同时操作一个全局变量导致的，其实是类似竞争。如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/e1be5a778b83faeaf3c9c90d285e62a9-0)

然后，在两个线程同时操作时，就会发生如下情况：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/5b401afb4c9998e5ecf24228dc8cf265-0)

从图中能够看出，线程 1 虽然完成了计算，但是在还没有将结果放回到内存中时，线程 2 就又执行了操作。这种操作由于每次发生的次序不一致也就导致最终结果每次都不一样的情况。

为了在程序中使用多线程方式而避免发生这种问题，我们需要使得线程在操作临界区（全局变量）时能够进行同步操作。比如：线程 1 在操作时，线程 2 暂时停止操作，等待线程 1 操作完成后再操作。



## 5 线程同步及线程销毁

通过线程同步是肯定能够解决前面的线程存在的问题。常用的线程同步技术有 “互斥量” 和 “信号量” 。二者概念十分接近，使用只有掌握其中一种就能够对两者都有一定掌握。

### 互斥量

互斥量（ mutex ）主要就是用来解决线程同步问题的。

```C++
#include <pthread.h>

//成功返回 0，失败返回其他值
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
int pthread_mutex_destroy(pthread_mutex_t *mutex);

/*
mutex 创建互斥量时传递保存互斥量的变量地址
attr  传递即将创建的互斥量属性，默认属性传递NULL
*/

//保护临界区而需要添加的上锁和解锁操作
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

接下来我们修改前面的 `threadBugDemo.cpp` 代码，添加互斥量操作，具体修改后代码如下图：

```C++
#include <iostream>
#include <pthread.h>

#define NUM_THREAD 100

//自增线程
void *thread_inc(void *arg);
//自减线程
void *thread_des(void *arg);
//全局变量
long long num = 0;

//声明使用的互斥量
pthread_mutex_t mutex;

int main(int argc, char *argv[])
{
    pthread_t thread_id[NUM_THREAD];

    //初始化互斥量
    pthread_mutex_init(&mutex, NULL);

    //创建50个加线程，创建50个减线程
    for(int i = 0; i < NUM_THREAD; ++i)
    {
        if(i%2)
            pthread_create(&(thread_id[i]), NULL, thread_inc, NULL);
        else
            pthread_create(&(thread_id[i]), NULL, thread_des, NULL);
    }

    //回收这100个线程资源
    for(int i = 0; i< NUM_THREAD; ++i)
    {
        pthread_join(thread_id[i], NULL);
    }

    //打印输出结果
    std::cout << "Result : " << num << std::endl;

    //释放互斥量
    pthread_mutex_destroy(&mutex);
    return 0;
}

//自增线程
void *thread_inc(void *arg)
{
    pthread_mutex_lock(&mutex);
    for(int i = 0; i < 3000; ++i)
    {
        num++;
    }
    pthread_mutex_unlock(&mutex);
    return NULL;
}
//自减线程
void *thread_des(void *arg)
{
    pthread_mutex_lock(&mutex);
    for(int i = 0; i < 3000; ++i)
    {
        num--;
    }
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```

修改完成后，再次编译并执行，结果就会如我们所想：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/91b09f24610b0b78cae94469f0cee5b7-0)

## 6 多线程并发服务器的实现



通过学习多线程的使用及同步操作后，接下来我们修改一直维护的项目 `TCP-ServerDemo` 代码，使得 TCP 服务器通过多线程方式实现并发操作，修改项目中的 `src/main.cpp` 文件代码，具体如下：

```C++
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <cstring>
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/select.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <pthread.h>
#include "base.h"

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}


//子进程处理客户端请求函数封装
int ChildProcessWork(int ServSocket, int ChildSocket)
{
    char buf[BUFFSIZE] = {0};
    int str_len;
    //首先在子进程中关闭服务端套接字
    close(ServSocket);

    while(1)
    {
        //接收客户端数据
        str_len = read(ChildSocket, buf, BUFFSIZE);
        if(str_len == 0)
        {
            //客户端断开连接，处理结束
            break;
        }
        //多进程服务中不再将受到数据打印输出
        //将客户端数据返回给客户端
        write(ChildSocket, buf, str_len);
        memset(buf, 0, BUFFSIZE);
    }

    //关闭客户端套接字
    close(ChildSocket);
    return 0;
}

//处理子进程结束后，父进程回收资源的信号回调函数
void read_childproc(int signum)
{
    pid_t pid;
    int status;
    pid = waitpid(-1, &status, WNOHANG);
    std::cout << "Removed proc id : " << pid << std::endl;
}

//多进程工作
int fork_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    pid_t pid;

    //循环等待客户端请求
    while (1)
    {
        adr_size = sizeof(client_adr);
        client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
        if(client_sock < 0)
        {
            //接收请求失败，继续工作
            continue;
        }
        else{
            //接收到新的客户端，打印输出
            std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
            //创建子进程进行处理
            pid = fork();
            if(pid < 0)
            {
                //创建子进程失败，关闭连接
                std::cout << "fork error, close client" << std::endl;
                close(client_sock);
                continue;
            }
            if(pid == 0)
            {
                //进入子进程,
                ret = ChildProcessWork(serv_sock, client_sock);
                //调用结束后直接结束子进程
                exit(ret);
            }
            else{
                //父进程关闭客户端套接字，继续等待新客户端请求
                close(client_sock);
            }
        }
    }
    return 0;
}

//select I/O复用
int select_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    int fd_max, fd_num;
    fd_set reads, cpy_reads;
    struct timeval timeout;
    char buf[BUFFSIZE];

    FD_ZERO(&reads);
    FD_SET(serv_sock, &reads);
    fd_max = serv_sock;
    int str_len;

    while(1)
    {
        cpy_reads = reads;
        timeout.tv_sec = 5;
        timeout.tv_usec = 5000;

        fd_num = select(fd_max + 1, &cpy_reads, 0, 0, &timeout);
        if(fd_num < 0)
        {
            //调用select 失败
            std::cout << "select error." << std::endl;
            break;
        }
        else if(fd_num == 0)
        {
            //目前没有数据事件发生
            continue;
        }
        //循环处理当前所有可读事件
        for(int i = 0; i < fd_num; i++)
        {
            if(FD_ISSET(i, &cpy_reads))
            {
                //如果当前要处理的文件描述符是 serv_sock 服务器套接字，则表示需要处理客户端请求连接事件
                if(i == serv_sock)
                {
                    adr_size = sizeof(client_adr);
                    client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
                    //打印新连接客户端信息
                    std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
                    //将新连接的客户端注册到select处理队列中
                    FD_SET(client_sock, &reads);
                    //如果当前最大值小于新客户端套接字，则改变
                    if(fd_max < client_sock)
                    {
                        fd_max = client_sock;
                    }
                }
                else
                {
                    //处理客户端发来的消息，echo服务器负责将数据返回
                    str_len = read(i, buf, BUFFSIZE);
                    if(str_len == 0)
                    {
                        //客户端连接断开
                        //将客户端套接字从select监控队列中清除
                        FD_CLR(i, &reads);
                        close(i);
                        //输出打印
                        std::cout << "Closed Client : " << i << std::endl;
                    }
                    else
                    {
                        //正常处理数据，将读取到的数据返回给客户端
                        write(i, buf, str_len);
                    }

                }
            }
        }
    }

    return 0;
}

//设置套接字非阻塞
void setnonblockingmode(int fd)
{
    int flag = fcntl(fd, F_GETFL, 0);
    fcntl(fd, F_SETFL, flag | O_NONBLOCK);
}

//epoll I/O 复用方式
int epoll_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    int epoll_size, epfd, event_cnt;
    struct epoll_event *ep_events;
    struct epoll_event event;
    int timeout;
    int str_len;
    char buf[BUFFSIZE];

    //初始化部分参数，创建epoll 例程
    epoll_size = 50;    //默认定义50作为监视文件描述符数量，可修改
    epfd = epoll_create(epoll_size);
    if(epfd < 0)
    {
        //创建失败，退出程序
        std::cout << "epoll_create error." << std::endl;
        return -1;
    }
    //申请epoll_wait 返回缓冲区
    ep_events = (struct epoll_event *)malloc(sizeof(struct epoll_event) * epoll_size);

    //设置服务器套接字非阻塞
    setnonblockingmode(serv_sock);
    //优先将服务器套接字注册到epoll监视例程中
    event.events = EPOLLIN; //服务器套接字不使用边缘触发
    event.data.fd = serv_sock;
    //添加文件描述符及事件
    epoll_ctl(epfd, EPOLL_CTL_ADD, serv_sock, &event);

    //初始化超时时间为 5.5 s
    timeout = 5500;

    while(1)
    {
        //调用 epoll_wait 获取当前文件描述符注册事件
        event_cnt = epoll_wait(epfd, ep_events, epoll_size, timeout);
        if(event_cnt < 0)
        {
            //调用select 失败
            std::cout << "epoll_wait error." << std::endl;
            break;
        }
        else if(event_cnt == 0)
        {
            //目前没有数据事件发生，超时
            continue;
        }
        //循环处理当前所有待处理事件
        for(int i = 0; i < event_cnt; i++)
        {
            if(ep_events[i].data.fd == serv_sock)//判断事件文件描述符是服务器套接字
            {
                //如果当前要处理的文件描述符是 serv_sock 服务器套接字，则表示需要处理客户端请求连接事件
                adr_size = sizeof(client_adr);
                client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
                //打印新连接客户端信息
                std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
                //初始化 event 变量，注册到 epoll 监视列表中
                event.events = EPOLLIN | EPOLLET;  //添加边缘触发条件
                event.data.fd = client_sock;
                epoll_ctl(epfd, EPOLL_CTL_ADD, client_sock, &event);
            }
            else
            {
                //由于添加了边缘触发，所以一次操作就需要将输入缓冲区中数据全部处理完成
                while(1)
                {
                    //处理客户端发来的消息，echo服务器负责将数据返回
                    str_len = read(i, buf, BUFFSIZE);
                    if(str_len == 0)
                    {
                        //客户端断开连接，将该客户端套接字从监视列表中删除
                        epoll_ctl(epfd, EPOLL_CTL_DEL, ep_events[i].data.fd, NULL);
                        close(ep_events[i].data.fd);
                        //输出打印
                        std::cout << "Closed Client : " << ep_events[i].data.fd << std::endl;
                        break;
                    }
                    else if(str_len < 0)
                    {
                        if(errno == EAGAIN)
                        {
                            //数据读取完毕
                            break;
                        }
                    }
                    else
                    {
                        //正常处理数据，将读取到的数据返回给客户端
                        write(ep_events[i].data.fd, buf, str_len);
                    }


                }
            }
        }
    }

    //关闭epoll 例程
    close(epfd);

    return 0;
}

void *handle_client(void *arg)
{
    int client_sock = *(int *)arg;
    int str_len = 0;
    char buf[BUFFSIZE];

    while(1)
    {
        //处理客户端发来的消息，echo服务器负责将数据返回
        str_len = read(client_sock, buf, BUFFSIZE);
        if(str_len == 0)
        {
            //客户端连接断开
            close(client_sock);
            //输出打印
            std::cout << "Closed Client : " << client_sock << std::endl;
        }
        else
        {
            //正常处理数据，将读取到的数据返回给客户端
            write(client_sock, buf, str_len);
        }

    }
}

int pthread_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    pthread_t t_id;


    //循环等待客户端请求
    while (1)
    {
        adr_size = sizeof(client_adr);
        client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);

        pthread_create(&t_id, NULL, handle_client, (void *)&client_sock);
        //分离线程，使其单独执行，执行完成后自动结束，不需要去执行 pthread_join回收资源
         pthread_detach(t_id);
        //打印新连接客户端信息
        std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
    }

    return 0;
}


//修改主函数
int main(int argc, char *argv[])
{
    int serv_sock, ret;
    struct sockaddr_in serv_adr;
    struct sigaction act;

    int str_len, state;
    int option;
    socklen_t optlen;

    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << "<port> " << std::endl;
        exit(1);
    }

    act.sa_handler = read_childproc;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    state = sigaction(SIGCHLD, &act, 0);

    //创建服务socket套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //添加 Time-wait 状态重新分配断开设置
    optlen = sizeof(option);
    option = 1;
    setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, (void*)&option, optlen);


    //初始化服务套接字
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    //绑定套接字
    ret = bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr));
    if(ret < 0)
    {
        error_handle("bind", "bind() error.");
    }

    //监听端口
    ret = listen(serv_sock, 5);
    if(ret < 0)
    {
        error_handle("listen", "listen() error");
    }

    // //多进程方式实现
    // fork_workproc(serv_sock);

    // //select I/O 复用方式
    // select_workproc(serv_sock);

    //epoll I/O 复用方式
    // epoll_workproc(serv_sock);

    //多线程方式
    pthread_workproc(serv_sock);

    //关闭套接字
    close(serv_sock);
    return 0;
}
```

代码修改完成后，在 VS Code 平台中的终端窗口下执行 `make` 命令进行项目编译：

```bash
cd /home/project/code9/TCP-ServerDemo
make
```

此时应该会有如下报错，因为我们在代码中添加了使用 `pthread` 相关的函数，所以需要在 `Makefile` 的编译参数中添加 `-lpthread`。

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/4c7db5e4912f93c750cc5b6fa1df6ac3-0)

```makefile
.PHONY: all
all: $(target)
$(target): $(objs)
    $(CC) $(CFLAGS)  -o $@ $(objs) -lpthread

%.o:%.cpp
    $(CC) ${CFLAGS} -c -o $@ $<
```

编译截图如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/4c33b3028e1b6e564c29e98c1817e648-0)

验证修改后的服务器运行效果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/567cbc261e516bac8077696bc8a0bc28-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/db31087e1b5e800f163458faa04ea4e5-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/515ad9eb8a15286037b36fa5522c5a7a-0)

关闭客户端后：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/bd02203a7747e8444c258c68edd74634-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/fa1231cd23c60103c2e643edbec1be27-0)



## 7 实验总结

通过本节实验的学习，我们了解能够通过多线程方式实现在 Linux 平台下的服务器并发操作，并且与前面多进程并发服务器实现相比对计算机资源占用更少，且实现其阿里更加方便。对于 Linux 平台下的网络编程的知识掌握更深入了一层，在后面的章节中会将目前所学到的各个部分的内容组合起来，这样就构成了在商业上经常使用的服务器实现方式。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code9.zip
```

# 十 初步制作 HTTP 服务器

## 1 实验介绍

本章实验的重点就是带大家认识并初步制作 HTTP 服务器。前面经过对 TCP 服务器的多方面的讲解和学习，相信大家已经对 TCP 传输方式及各方面功能实现有了很深的理解和掌握，接下来就在基于 TCP 传输协议的基础上引入应用层网络协议 HTTP 协议的实现和基于 TCP 服务器如何实现 Web 服务器的功能。HTTP 协议是目前网络编程中经常要用到的应用层协议，尤其是在 B/S 架构的项目开发中。通过本章实验的学习和练习，相信大家能够对这部分的知识有一定的理解和掌握。

#### 知识点

- HTTP 协议
- HTTP 协议请求消息包和响应消息包结构
- 实现简单的 Web 服务器
- 修改 TCP 项目代码提供简单的 HTTP 响应



## 2 HTTP 无状态连接特性

接下来详细介绍一下 HTTP 协议，首先要说明的就是 HTTP 协议虽然是基于 TCP 传输方式实现，但是 HTTP 却是采用短连接方式————无状态 `Stateless` 协议，由于 HTTP 协议的规定，服务器在与客户端建立连接后仅处理一次请求然后就会断开连接，即使同一个客户端再次访问也作为新的客户端进行处理，由此产生了无状态连接，无论什么时候的请求都作为新请求处理，不会针对客户端保存它的状态。具体如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/3ec82a7ef033dbda9e691a00e078779c-0)

当然，虽然 HTTP 协议本身是不会保存状态和维持短连接请求和断开连接的，但是我们在使用过程中可以通过采用 `Cookie` 和 `Session` 的方式记录和保持对应的客户端状态，这个后面会讲到。



## 3 HTTP 请求消息包

HTTP 协议的重点就在于它的请求和响应消息包的格式是有规定的。下面我们先来看一下 HTTP 请求消息包的格式，具体如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/1fa29cbb7a31f931976eac081b176550-0)

从上图中可以看出，请求消息包从头到尾可以定义为：

- 请求行：包含有具体的请求方式信息，HTTP 协议的请求方式主要有 `GET` 和 `POST` , 其中：
  - `GET` ：主要用于请求数据
  - `POST`：主要用于传输数据 而后面的数据表示该 HTTP 数据包采用的是 HTTP/1.1 的版本，也就是说在 HTTP 请求包的请求行中除了表示请求方式外还需要携带版本信息。
- 消息头：消息头会包含很多信息，可以将这部分看成一大堆的 JSON 数据串，其中会包含浏览器信息、用户认证信息以及最重要的消息主体字节长度、字符编码等等，只要中间没有用空行隔开，就都涵盖在消息头结构中。
- 空行：这里的空行就是为了区分消息头和 HTTP 消息主体数据采用的分割符号。
- 消息主体：通过空行分割，HTTP 剩下的内容就是请求消息包的主体消息了。

HTTP 协议的特点就是每次客户端发送请求消息包给 Web 服务器后，服务器都会针对请求的消息返回一个有着固定格式的响应消息包。这个响应消息的包结构如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/da27d4d9351c954d6c01e0a2295d2ccf-0)

HTTP 响应包也分为以下几部分：

- 状态行：包含着请求消息返回的响应状态以及同样的 HTTP 版本号，消息状态有着一定的固定编号，如下：
  - 200 OK：成功处理了请求
  - 404 Not Found：请求的文件不存在
  - 400 Bad Request：请求方式错误，请检查
- 消息头：类似请求包的消息头，结构类似 JSON ，内容包含了传输的数据类型、数据长度、编码格式等等。
- 空行：同请求包类似，用来分割消息头和消息主体。
- 消息主体：剩下的部分就是响应消息包的具体消息内容，长度一般在消息头中会有说明。

## 4 实现简单的 web 服务器



接下来，我们在最简化的 TCP 服务器上实现 Web 服务器功能，主要就是在服务器消息处理上需要针对 HTTP 协议的请求消息包格式进行解析，并且能够正确返回对应的 HTTP 协议格式的响应消息。因为最终实现的是 Web 服务器功能，所以最后在测试时不需要再编写对应的客户端程序，直接通过通用浏览器进行测试即可。

复制 `/home/project/code9` 文件夹，并命名为 `/home/project/code10` 文件夹。接下来的内容统一在 `code10` 中进行修改。

1. 新建源代码文件 `tools/httpDemo.cpp` ，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/20b8cd1ff1d03e30a71d02168357d44c-0)

1. 文件新建完成后，添加最基本的 TCP 服务器部分代码，这部分可以参考实验一章节中的代码内容，具体如下：

```C++
#include <errno.h>
#include <ctype.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/stat.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <arpa/inet.h>
#include <iostream>

#define BUFFSIZE    1024

//出错调用函数
void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}

//读取一行数据，返回数据长度
int getFdLine(int fd, char *buf, int size);

//回复客户端400错误
void Response400(int client_sock);
//回复客户端404错误
void Response404(int client_sock);
//回复客户端501错误
void Response501(int client_sock);
//回复客户端500错误
void Response500(int client_sock);
//回复客户端正常 200 状态
void Response200(int client_sock);
//返回客户端请求的文件内容
void FileResponse(int client_sock, const char *FileNamePath);

//http 协议客户端请求数据处理
int http_request(int client_sock);



int main(int argc, char *argv[])
{
    int serv_sock;
    int client_sock;

    struct sockaddr_in serv_addr;
    struct sockaddr_in client_addr;

    socklen_t client_addr_size;
    char message[] = "hello world";

    //判断参数数量，Usage: <port>， 需要在命令行输入服务器接收消息的端口号
    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << " <prot>" << std::endl;
        exit(1);
    }

    //创建socket 套接字
    serv_sock = socket(AF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //初始化套接字结构体
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);//选择当前任意网卡
    serv_addr.sin_port = htons(atoi(argv[1]));//设置接收消息的端口号

    //绑定端口
    if(bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0)
    {
        error_handle("bind", "bind() error.");
    }

    //监听端口，设置等待队列数量为5
    if(listen(serv_sock, 5) < 0)
    {
        error_handle("listen", "listen() error.");
    }
    int ret = 0;
    client_sock = -1;

    while(1)
    {
        //打印输出等待连接
        std::cout << "Waiting Client.... server : " << serv_sock << " client_socke" << client_sock << std::endl;

        client_addr_size = sizeof(client_addr);
        //等待接收客户端建立连接
        client_sock = accept(serv_sock, (struct sockaddr*)&client_addr, &client_addr_size);
        if(client_sock < 0)
        {
            error_handle("accept", "accept() error.");
        }
        //accept() 成功建立连接后，服务器就会得到客户端的 IP 地址和端口号。
        //打印客户端 IP 和端口号
        std::cout << "Client IP : " << inet_ntoa(client_addr.sin_addr) << " , port : " << ntohs(client_addr.sin_port) << std::endl;

        //http 请求解析处理
        ret = http_request(client_sock);

        std::cout << "ret = " << ret << " client :" << client_sock << std::endl;
        //关闭TCP连接
        close(client_sock);
        client_sock = -1;
    }

    //关闭socket套接字
    close(serv_sock);

    return 0;
}
```

1. 对 TCP 服务器代码进行修改，增加 HTTP 协议支持，添加 HTTP 数据包解析和响应部分代码，继续向 `httpDemo.cpp` 文件末尾添加如下代码内容：

```C++
//HTTP 数据请求解析函数, 成功回复返回 0, 否则返回 -1
int http_request(int client_sock)
{
    char req_line[BUFFSIZE];

    char method[10];
    char ct[15];
    char file_name[30];


    //请求错误，直接返回结果
    if(getFdLine(client_sock, req_line, sizeof(req_line)) == 0){
        Response501(client_sock);
        return -1;
    }

    if(strstr(req_line, "HTTP/")==NULL){
        //请求消息头信息不正确
        Response501(client_sock);
        return -1;
    }

    //判断方式是否正确
    if(strstr(req_line, "GET") != NULL || strstr(req_line, "POST") != NULL)
    {
        //请求方式正确
        // //判断是 post还是 get
        // if(strstr(req_line, "GET") != NULL)
        // {
        //     //请求方式是 GET

        // }
        // else{
        //     //请求方式是 POST

        // }
        //不论是 POST 还是 GET ，处理方式都一样
        Response200(client_sock);
    }
    else{
        //HTTP 请求方式不是 GET ，有问题
        Response501(client_sock);
        return -1;
    }

    return 0;
}

//读取一行数据，返回数据长度
int getFdLine(int fd, char *buf, int size)
{
    char c, *top = buf;
    int ret = 0;

    while((top - buf) <= size)
    {
        if(read(fd, &c, 1) <= 0)
            break;
        else
            *top++ = c;
        if(c == '\r'){
            //读取到了 '\r' 数据，判断下一个数据是否是 '\n'
            read(fd, &c, 1);
            if(c == '\n')
                *top++ = c;
            else
                *top++ = '\n';
            break;
        }
    }
    //追加结束字符
    *top++ = '\0';
    ret = top - buf;

    return ret;
}

//回复客户端400错误
void Response400(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 400 BAD REQUEST\r\n"
    "Server: wz simple httpd 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<p>你的请求有问题,请检查语法!</p>\r\n";

    write(client_sock, estr, strlen(estr));
}

//回复客户端404错误
void Response404(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 404 NOT FOUND\r\n"
    "Server: wz simple httpd 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<html>"
    "<head><title>你请求的界面被查水表了!</title></head>\r\n"
    "<body><p>404: 估计是回不来了</p></body>"
    "</html>";

    write(client_sock, estr, strlen(estr));
}

//回复客户端501错误
void Response501(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 501 Method Not Implemented\r\n"
    "Server: wz simple httpd 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<html>"
    "<head><title>小伙子不要乱请求</title></head>\r\n"
    "<body><p>too young too simple, 年轻人别总想弄出个大新闻.</p></body>"
    "</html>";

    write(client_sock, estr, strlen(estr));
}

//回复客户端500错误
void Response500(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 500 Internal Server Error\r\n"
    "Server: wz simple httpd 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<html>"
    "<head><title>Sorry </title></head>\r\n"
    "<body><p>最近有点方了!</p></body>"
    "</html>";

    write(client_sock, estr, strlen(estr));
}

//回复客户端正常 200 状态
void Response200(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 200 OK\r\n"
    "Server: simple http server 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n";

    const char html[] =
    "HTTP/1.0 200 OK\r\n"
    "Server: simple http server 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<!DOCTYPE html>\r\n"
    "<html>\r\n"
    "<body>\r\n"
    "<script type=\"text/javascript\">\r\n"
    "document.write(\"<h1>Hello World!</h1>\")\r\n"
    "</script>\r\n"
    "</body>\r\n"
    "</html>\r\n";

    write(client_sock, html, strlen(html));
}


//返回客户端请求的文件内容
void FileResponse(int client_sock, const char *FileNamePath)
{
    char buf[BUFSIZ];
    printf("path = [%s]\n", FileNamePath);
    FILE *txt = fopen(FileNamePath, "r");

    // 如果文件解析错误, 给它个404
    if(NULL == txt)
        Response404(client_sock);
    else{
        //发送给200的报文头过去, 并发送文件内容过去
        Response200(client_sock);
        while(!feof(txt) && fgets(buf, sizeof(buf), txt))
            write(client_sock, buf, strlen(buf));
        fclose(txt);
    }
}
```

1. 编译代码，在 VS Code 平台下的终端窗口通过命令编译代码：

```bash
cd /home/project/code10/TCP-ServerDemo/tools
g++ httpDemo.cpp -o httpDemo
```

如下图：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/cfee2b0f5f247804b095851eeb0827e9-0)

1. 运行 Web 服务器并测试，先通过命令在终端执行服务器：

```bash
cd /home/project/code10/TCP-ServerDemo/tools
./httpDemo 8080
```

当程序运行起来后，点击右侧工具栏的“Web 服务”会自动打开浏览器访问对应的端口：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/fd5a585541ace3bbe377484474e36079-0)

可以看到，能够在浏览器中正常访问我们构建的 Web 服务器并显示出最基本的网页页面。

## 5 修改项目代码，实现 Web 服务器功能



通过前面的小例子，相信大家已经对 HTTP 协议有了一定了理解和掌握，现在就来修改我们一直维护的项目代码，在其中添加对应的 HTTP 功能，使得我们的服务器能够具有最基本的 Web 服务器功能。

为了后续针对应用层协议有很好地支持，这里将 HTTP 数据处理和响应的功能代码放置到一个新的文件中，在项目目录 `src` 下新建 HTTP 协议使用源文件 `mhttp.cpp` 以及在 `include` 目录下添加对应的头文件 `mhttp.h`。

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/71f36613387ff8b8091b0b48b1e4538f-0)

在 `mhttp.h` 头文件中添加对应的功能函数声明，具体如下：

```C++
#ifndef MHTTP__H
#define MHTTP__H


//读取一行数据，返回数据长度
int getFdLine(int fd, char *buf, int size);

//回复客户端400错误
void Response400(int client_sock);
//回复客户端404错误
void Response404(int client_sock);
//回复客户端501错误
void Response501(int client_sock);
//回复客户端500错误
void Response500(int client_sock);
//回复客户端正常 200 状态
void Response200(int client_sock);
//返回客户端请求的文件内容
void FileResponse(int client_sock, const char *FileNamePath);

//http 协议客户端请求数据处理
int http_request(int client_sock);


#endif
```

在 `mhttp.cpp` 源文件中添加函数具体实现，如下：

```C++
#include "mhttp.h"
#include "base.h"
#include <cstdio>
#include <string>
#include <cstring>
#include <unistd.h>



//HTTP 数据请求解析函数, 成功回复返回 0, 否则返回 -1
int http_request(int client_sock)
{
    char req_line[BUFFSIZE];

    char method[10];
    char ct[15];
    char file_name[30];


    //请求错误，直接返回结果
    if(getFdLine(client_sock, req_line, sizeof(req_line)) == 0){
        Response501(client_sock);
        return -1;
    }

    if(strstr(req_line, "HTTP/")==NULL){
        //请求消息头信息不正确
        Response501(client_sock);
        return -1;
    }

    //判断方式是否正确
    if(strstr(req_line, "GET") != NULL || strstr(req_line, "POST") != NULL)
    {
        //请求方式正确
        // //判断是 post还是 get
        // if(strstr(req_line, "GET") != NULL)
        // {
        //     //请求方式是 GET

        // }
        // else{
        //     //请求方式是 POST

        // }
        //不论是 POST 还是 GET ，处理方式都一样
        Response200(client_sock);
    }
    else{
        //HTTP 请求方式不是 GET ，有问题
        Response501(client_sock);
        return -1;
    }

    return 0;
}

//读取一行数据，返回数据长度
int getFdLine(int fd, char *buf, int size)
{
    char c, *top = buf;
    int ret = 0;

    while((top - buf) <= size)
    {
        if(read(fd, &c, 1) <= 0)
            break;
        else
            *top++ = c;
        if(c == '\r'){
            //读取到了 '\r' 数据，判断下一个数据是否是 '\n'
            read(fd, &c, 1);
            if(c == '\n')
                *top++ = c;
            else
                *top++ = '\n';
            break;
        }
    }
    //追加结束字符
    *top++ = '\0';
    ret = top - buf;

    return ret;
}

//回复客户端400错误
void Response400(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 400 BAD REQUEST\r\n"
    "Server: wz simple httpd 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<p>你的请求有问题,请检查语法!</p>\r\n";

    write(client_sock, estr, strlen(estr));
}

//回复客户端404错误
void Response404(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 404 NOT FOUND\r\n"
    "Server: wz simple httpd 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<html>"
    "<head><title>你请求的界面被查水表了!</title></head>\r\n"
    "<body><p>404: 估计是回不来了</p></body>"
    "</html>";

    write(client_sock, estr, strlen(estr));
}

//回复客户端501错误
void Response501(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 501 Method Not Implemented\r\n"
    "Server: wz simple httpd 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<html>"
    "<head><title>小伙子不要乱请求</title></head>\r\n"
    "<body><p>too young too simple, 年轻人别总想弄出个大新闻.</p></body>"
    "</html>";

    write(client_sock, estr, strlen(estr));
}

//回复客户端500错误
void Response500(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 500 Internal Server Error\r\n"
    "Server: wz simple httpd 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<html>"
    "<head><title>Sorry </title></head>\r\n"
    "<body><p>最近有点方了!</p></body>"
    "</html>";

    write(client_sock, estr, strlen(estr));
}

//回复客户端正常 200 状态
void Response200(int client_sock)
{
    const char estr[] =
    "HTTP/1.0 200 OK\r\n"
    "Server: simple http server 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n";

    const char html[] =
    "HTTP/1.0 200 OK\r\n"
    "Server: simple http server 1.0\r\n"
    "Content-Type: text/html\r\n"
    "\r\n"
    "<!DOCTYPE html>\r\n"
    "<html>\r\n"
    "<body>\r\n"
    "<script type=\"text/javascript\">\r\n"
    "document.write(\"<h1>Hello World!</h1>\")\r\n"
    "</script>\r\n"
    "</body>\r\n"
    "</html>\r\n";

    write(client_sock, html, strlen(html));
}


//返回客户端请求的文件内容
void FileResponse(int client_sock, const char *FileNamePath)
{
    char buf[BUFSIZ];
    printf("path = [%s]\n", FileNamePath);
    FILE *txt = fopen(FileNamePath, "r");

    // 如果文件解析错误, 给它个404
    if(NULL == txt)
        Response404(client_sock);
    else{
        //发送给200的报文头过去, 并发送文件内容过去
        Response200(client_sock);
        while(!feof(txt) && fgets(buf, sizeof(buf), txt))
            write(client_sock, buf, strlen(buf));
        fclose(txt);
    }
}
```

完成 HTTP 协议功能代码的添加后，在项目的主文件 `main.cpp` 中添加对应的函数使用和头文件引用。通过前面的学习，目前考虑实现并发的最好的方式其实是采用多线程的方式实现，所以我们修改项目中的 `src/main.cpp` 文件代码，具体如下：

```C++
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <cstring>
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/select.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <pthread.h>
#include "base.h"
#include "mhttp.h"

//出错调用函数
static void error_handle(std::string opt, std::string message)
{
    //根据errno值获取失败原因并打印到终端
    perror(opt.c_str());
    std::cout << message << std::endl;
    exit(1);
}


//子进程处理客户端请求函数封装
int ChildProcessWork(int ServSocket, int ChildSocket)
{
    char buf[BUFFSIZE] = {0};
    int str_len;
    //首先在子进程中关闭服务端套接字
    close(ServSocket);

    while(1)
    {
        //接收客户端数据
        str_len = read(ChildSocket, buf, BUFFSIZE);
        if(str_len == 0)
        {
            //客户端断开连接，处理结束
            break;
        }
        //多进程服务中不再将受到数据打印输出
        //将客户端数据返回给客户端
        write(ChildSocket, buf, str_len);
        memset(buf, 0, BUFFSIZE);
    }

    //关闭客户端套接字
    close(ChildSocket);
    return 0;
}

//处理子进程结束后，父进程回收资源的信号回调函数
void read_childproc(int signum)
{
    pid_t pid;
    int status;
    pid = waitpid(-1, &status, WNOHANG);
    std::cout << "Removed proc id : " << pid << std::endl;
}

//多进程工作
int fork_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    pid_t pid;

    //循环等待客户端请求
    while (1)
    {
        adr_size = sizeof(client_adr);
        client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
        if(client_sock < 0)
        {
            //接收请求失败，继续工作
            continue;
        }
        else{
            //接收到新的客户端，打印输出
            std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
            //创建子进程进行处理
            pid = fork();
            if(pid < 0)
            {
                //创建子进程失败，关闭连接
                std::cout << "fork error, close client" << std::endl;
                close(client_sock);
                continue;
            }
            if(pid == 0)
            {
                //进入子进程,
                ret = ChildProcessWork(serv_sock, client_sock);
                //调用结束后直接结束子进程
                exit(ret);
            }
            else{
                //父进程关闭客户端套接字，继续等待新客户端请求
                close(client_sock);
            }
        }
    }
    return 0;
}

//select I/O复用
int select_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    int fd_max, fd_num;
    fd_set reads, cpy_reads;
    struct timeval timeout;
    char buf[BUFFSIZE];

    FD_ZERO(&reads);
    FD_SET(serv_sock, &reads);
    fd_max = serv_sock;
    int str_len;

    while(1)
    {
        cpy_reads = reads;
        timeout.tv_sec = 5;
        timeout.tv_usec = 5000;

        fd_num = select(fd_max + 1, &cpy_reads, 0, 0, &timeout);
        if(fd_num < 0)
        {
            //调用select 失败
            std::cout << "select error." << std::endl;
            break;
        }
        else if(fd_num == 0)
        {
            //目前没有数据事件发生
            continue;
        }
        //循环处理当前所有可读事件
        for(int i = 0; i < fd_num; i++)
        {
            if(FD_ISSET(i, &cpy_reads))
            {
                //如果当前要处理的文件描述符是 serv_sock 服务器套接字，则表示需要处理客户端请求连接事件
                if(i == serv_sock)
                {
                    adr_size = sizeof(client_adr);
                    client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
                    //打印新连接客户端信息
                    std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
                    //将新连接的客户端注册到select处理队列中
                    FD_SET(client_sock, &reads);
                    //如果当前最大值小于新客户端套接字，则改变
                    if(fd_max < client_sock)
                    {
                        fd_max = client_sock;
                    }
                }
                else
                {
                    //处理客户端发来的消息，echo服务器负责将数据返回
                    str_len = read(i, buf, BUFFSIZE);
                    if(str_len == 0)
                    {
                        //客户端连接断开
                        //将客户端套接字从select监控队列中清除
                        FD_CLR(i, &reads);
                        close(i);
                        //输出打印
                        std::cout << "Closed Client : " << i << std::endl;
                    }
                    else
                    {
                        //正常处理数据，将读取到的数据返回给客户端
                        write(i, buf, str_len);
                    }

                }
            }
        }
    }

    return 0;
}

//设置套接字非阻塞
void setnonblockingmode(int fd)
{
    int flag = fcntl(fd, F_GETFL, 0);
    fcntl(fd, F_SETFL, flag | O_NONBLOCK);
}

//epoll I/O 复用方式
int epoll_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    int epoll_size, epfd, event_cnt;
    struct epoll_event *ep_events;
    struct epoll_event event;
    int timeout;
    int str_len;
    char buf[BUFFSIZE];

    //初始化部分参数，创建epoll 例程
    epoll_size = 50;    //默认定义50作为监视文件描述符数量，可修改
    epfd = epoll_create(epoll_size);
    if(epfd < 0)
    {
        //创建失败，退出程序
        std::cout << "epoll_create error." << std::endl;
        return -1;
    }
    //申请epoll_wait 返回缓冲区
    ep_events = (struct epoll_event *)malloc(sizeof(struct epoll_event) * epoll_size);

    //设置服务器套接字非阻塞
    setnonblockingmode(serv_sock);
    //优先将服务器套接字注册到epoll监视例程中
    event.events = EPOLLIN; //服务器套接字不使用边缘触发
    event.data.fd = serv_sock;
    //添加文件描述符及事件
    epoll_ctl(epfd, EPOLL_CTL_ADD, serv_sock, &event);

    //初始化超时时间为 5.5 s
    timeout = 5500;

    while(1)
    {
        //调用 epoll_wait 获取当前文件描述符注册事件
        event_cnt = epoll_wait(epfd, ep_events, epoll_size, timeout);
        if(event_cnt < 0)
        {
            //调用select 失败
            std::cout << "epoll_wait error." << std::endl;
            break;
        }
        else if(event_cnt == 0)
        {
            //目前没有数据事件发生，超时
            continue;
        }
        //循环处理当前所有待处理事件
        for(int i = 0; i < event_cnt; i++)
        {
            if(ep_events[i].data.fd == serv_sock)//判断事件文件描述符是服务器套接字
            {
                //如果当前要处理的文件描述符是 serv_sock 服务器套接字，则表示需要处理客户端请求连接事件
                adr_size = sizeof(client_adr);
                client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);
                //打印新连接客户端信息
                std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
                //初始化 event 变量，注册到 epoll 监视列表中
                event.events = EPOLLIN | EPOLLET;  //添加边缘触发条件
                event.data.fd = client_sock;
                epoll_ctl(epfd, EPOLL_CTL_ADD, client_sock, &event);
            }
            else
            {
                //由于添加了边缘触发，所以一次操作就需要将输入缓冲区中数据全部处理完成
                while(1)
                {
                    //处理客户端发来的消息，echo服务器负责将数据返回
                    str_len = read(i, buf, BUFFSIZE);
                    if(str_len == 0)
                    {
                        //客户端断开连接，将该客户端套接字从监视列表中删除
                        epoll_ctl(epfd, EPOLL_CTL_DEL, ep_events[i].data.fd, NULL);
                        close(ep_events[i].data.fd);
                        //输出打印
                        std::cout << "Closed Client : " << ep_events[i].data.fd << std::endl;
                        break;
                    }
                    else if(str_len < 0)
                    {
                        if(errno == EAGAIN)
                        {
                            //数据读取完毕
                            break;
                        }
                    }
                    else
                    {
                        //正常处理数据，将读取到的数据返回给客户端
                        write(ep_events[i].data.fd, buf, str_len);
                    }


                }
            }
        }
    }

    //关闭epoll 例程
    close(epfd);

    return 0;
}

void *handle_client(void *arg)
{
    int client_sock = *(int *)arg;
    int str_len = 0;
    char buf[BUFFSIZE];

    while(1)
    {
        //处理客户端发来的消息，echo服务器负责将数据返回
        str_len = read(client_sock, buf, BUFFSIZE);
        if(str_len == 0)
        {
            //客户端连接断开
            close(client_sock);
            //输出打印
            std::cout << "Closed Client : " << client_sock << std::endl;
        }
        else
        {
            //正常处理数据，将读取到的数据返回给客户端
            write(client_sock, buf, str_len);
        }

    }
}


void *handle_http_client(void *arg)
{
    int client_sock = *(int *)arg;
    int str_len = 0;
    char buf[BUFFSIZE];

    http_request(client_sock);

    close(client_sock);

    return NULL;
}


int pthread_workproc(int serv_sock)
{
    struct sockaddr_in client_adr;
    socklen_t adr_size;
    int client_sock, ret;
    pthread_t t_id;


    //循环等待客户端请求
    while (1)
    {
        adr_size = sizeof(client_adr);
        client_sock = accept(serv_sock, (struct sockaddr*)&client_adr, &adr_size);

        // pthread_create(&t_id, NULL, handle_client, (void *)&client_sock);
        //使用http请求处理线程
        pthread_create(&t_id, NULL, handle_http_client, (void *)&client_sock);
        //分离线程，使其单独执行，执行完成后自动结束，不需要去执行 pthread_join回收资源
        pthread_detach(t_id);
        //打印新连接客户端信息
        std::cout << "New Client IP : " << inet_ntoa(client_adr.sin_addr) << " , port : " << ntohs(client_adr.sin_port) << std::endl;
    }

    return 0;
}


//修改主函数
int main(int argc, char *argv[])
{
    int serv_sock, ret;
    struct sockaddr_in serv_adr;
    struct sigaction act;

    int str_len, state;
    int option;
    socklen_t optlen;

    if(argc < 2)
    {
        std::cout << "Usage : " << argv[0] << "<port> " << std::endl;
        exit(1);
    }

    act.sa_handler = read_childproc;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    state = sigaction(SIGCHLD, &act, 0);

    //创建服务socket套接字
    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock < 0)
    {
        error_handle("socket", "socket() error.");
    }

    //添加 Time-wait 状态重新分配断开设置
    optlen = sizeof(option);
    option = 1;
    setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, (void*)&option, optlen);


    //初始化服务套接字
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    //绑定套接字
    ret = bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr));
    if(ret < 0)
    {
        error_handle("bind", "bind() error.");
    }

    //监听端口
    ret = listen(serv_sock, 5);
    if(ret < 0)
    {
        error_handle("listen", "listen() error");
    }

    // //多进程方式实现
    // fork_workproc(serv_sock);

    // //select I/O 复用方式
    // select_workproc(serv_sock);

    //epoll I/O 复用方式
    // epoll_workproc(serv_sock);

    //多线程方式
    pthread_workproc(serv_sock);

    //关闭套接字
    close(serv_sock);
    return 0;
}
```

代码添加完成后，在 VS Code 平台下的终端窗口中使用 `make` 命令进行项目编译，如下：

```bash
cd /home/project/code10/TCP-ServerDemo
make
make test
```

正常编译完成后，结果如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/600404/ad3371d83ac60f0862fe8df6bf48aa80-0)

运行并测试实现结果，由于是 Web 服务器，这里也是不采用客户端进行测试，通过浏览器进行测试验证即可，具体如下：

```bash
cd /home/project/code10/TCP-ServerDemo
./TcpServerDemo 8080
```

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/ca58c62eae573591294ce17e4c15fcbf-0)

使用测试环境默认的 8080 端口进行测试，具体如下：

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/5d9599c4d74f6fbd15d3430236c04b84-0)

![图片描述](https://doc.shiyanlou.com/courses/3496/1398198/5d08e0a77a9628a1daec99e1b640f517-0)

从测试结果能够看出，对 TCP 服务器项目的 HTTP 功能添加已经完成，后续如果需要修改项目支持其他的应用层协议，也可以参考这样的方式来实现。



## 6 实验总结

通过本节实验的学习，我们学习到网络编程中的 HTTP 协议以及 Web 服务器是如何实现的。

从第一章实验内容学习到本章内容后，我们已经学习到了最基本的网络编程中的传输协议、并发服务器实现、I/O 复用操作、应用层的 HTTP 协议支持等等。相信通过这段时间的学习，大家应该已经对 Linux 下的网络编程有了很深刻的认识和理解了。由于我们一直重点讲解 Linux 网络编程中的原理和基础操作实现，所以虽然代码语言使用 C++ 语言，但是编程方式还是采用面对过程编程而不是面向对象的方式，这也是为了帮助大家理解和掌握。

目前在 C++ 语言中经常用到的网络库其实有很多，其中最底层的实现原理也都是基于这里讲到的内容，所以经过本课程的基础讲述和学习，相信大家在以后在网络编程这部分能够更轻松和愉快地进行代码的创造和使用。

本实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3496/code10.zip
```
