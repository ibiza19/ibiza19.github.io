---
title: C语言最基本的套接字编程
tags: []
abstract: C语言最基本的套接字编程
categories:
  - 计算机网络
date: 2023-06-04 16:53:27
---
# 套接字是啥

套接字（socket）就是用来连接网络的工具

套接字可以比喻成电话机
- socket --- 买一个电话机☎️
- bind --- 分配电话号码（分配IP地址和端口号）
- listen --- 连接电话线（将连接转换为可接听状态）
- accept --- 拿起话筒接听（受理连接请求）

## 一个简单的服务端程序

```c
// hello_server.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int serv_sock;
    int clnt_sock;

    struct sockaddr_in serv_addr;
    struct sockaddr_in clnt_addr;
    socklen_t clnt_addr_size;

    char message[] = "Hello world";

    if (argc != 2) {
        printf("Usage: %s <port>\n", argv[0]);
        exit(1);
    }

    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if (serv_sock == -1) {
        error_handling("socket() error");
    }

    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_addr.sin_port = htons(atoi(argv[1]));

    if (bind(serv_sock, (struct sockaddr*) &serv_addr, sizeof(serv_addr)) == -1) {
        error_handling("bind() error");
    }

    if (listen(serv_sock, 5) == -1) {
        error_handling("listen() error");
    }

    clnt_addr_size = sizeof(clnt_addr);
    clnt_sock = accept(serv_sock, (struct sockaddr*) &clnt_addr, &clnt_addr_size);
    if (clnt_sock == -1) {
        error_handling("accept() error");
    }

    write(clnt_sock, message, sizeof(message));
    close(clnt_sock);
    close(serv_sock);
    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```

收到连接请求，给"Hello world"回应

会在 `clnt_sock = accept(serv_sock, (struct sockaddr*) &clnt_addr, &clnt_addr_size);`这里阻塞，等待有链接进来

## 一个简单的客户端

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int sock;
    struct sockaddr_in serv_addr;
    char message[30];
    int str_len;

    if (argc != 3) {
        printf("Usage: %s <IP> <port>\n", argv[0]);
        exit(1);
    }

    sock = socket(PF_INET, SOCK_STREAM, 0);
    if (sock == -1) {
        error_handling("socket() error");
    }

    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_addr.sin_port = htons(atoi(argv[2]));

    if (connect(sock, (struct sockaddr*) &serv_addr, sizeof(serv_addr)) == -1) {
        error_handling("connect() error");
    }

    str_len = read(sock, message, sizeof(message) - 1);
    if (str_len == -1) {
        error_handling("read() error!");
    }

    printf("Message from server : %s \n", message);
    close(sock);
    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```

调用`connect`函数向服务端发出请求

## 运行结果
![image.png](https://s2.loli.net/2023/06/04/6z7G1fHYsBOy5nA.webp)

# 套接字与文件操作

实际上，在Linux里面socket也被认为是文件的一种

已经分配的文件描述符和文件打开模式：

![image.png](https://s2.loli.net/2023/06/04/G3udLaHfMeK5ryW.webp)
![image.png](https://s2.loli.net/2023/06/04/hYg2FqWy35wHBNI.webp)

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/socket.h>

int main(void)
{
    int fd1, fd2, fd3;
    fd1 = socket(PF_INET, SOCK_STREAM, 0);
    fd2 = open("test.dat", O_CREAT | O_WRONLY | O_TRUNC);
    fd3 = socket(PF_INET, SOCK_DGRAM, 0);

    printf("file descriptor 1: %d\n", fd1);
    printf("file descriptor 2: %d\n", fd2);
    printf("file descriptor 3: %d\n", fd3);

    close(fd1);
    close(fd2);
    close(fd3);
    return 0;
}
```

结果：

```
▶ ./fds  
file descriptor 1: 3
file descriptor 2: 4
file descriptor 3: 5
```

除去系统占用的0 1 2，说明socket也是文件描述符，而且编号顺序是从小到大的