---
title: 基于c语言的socket编程
date: 2018-04-02 12:00:46
tags: C语言
---

本编主要是自己在用c语言搭建socket编程的细节 

<!--more-->

## tcp服务器端

代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc, char *argv[]) {
	int serv_sock;
	int clnt_sock;
	struct sockaddr_in serv_addr;
	struct sockaddr_in clnt_addr;
	socklen_t clnt_addr_size;

	char message[] = "Hello World";
	
	if (argc != 2){
		printf("Usage : %s <port>\n", argv[0]);
		exit(1);
	}

	serv_sock=socket(PF_INET, SOCK_STREAM, 0);//创建套接字（成功返回文件描述符，失败返回-1）
	if (serv_sock == -1) 
		error_handling("socket() error");
	memset(&serv_addr, 0, sizeof(serv_addr));//每个字节都用0填充
	serv_addr.sin_family=AF_INET;//使用IPv4地址
	serv_addr.sin_addr.s_addr=htonl(INADDR_ANY);//ip地址
	serv_addr.sin_port=htons(atoi(argv[1]));//端口

	if (bind(serv_sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) == -1)//给创建好的套接字分配地址信息( 1P 地址和端口号 )（成功返回文件描述符，失败返回-1）
			error_handling("bind() error");
	
	if (listen(serv_sock, 5) == -1) {////进入监听状态，等待用户发起请求
		error_handling("listen() error");
	}

	clnt_addr_size=sizeof(clnt_addr);
	//接收客户端请求
	clnt_sock=accept(serv_sock, (struct sockaddr *)&clnt_addr, &clnt_addr_size);
	if (clnt_sock == -1)
		error_handling("accept() error");

	write(clnt_sock, message, sizeof(message)); //向客户端发送数据
	//关闭套接字
	close(clnt_sock);
	close(serv_sock);
	return 0;
}

void error_handling(char *message) {
	fputs(message, stderr);
	fputc('\n', stderr);
	exit(1);
}
```

Internet协议地址结构
```c
struct sockaddr_in
{
	__SOCKADDR_COMMON (sin_);
	in_port_t sin_port;			/* Port number.  */
	struct in_addr sin_addr;		/* Internet address.  */

	/* Pad to size of `struct sockaddr'.  */
	unsigned char sin_zero[sizeof (struct sockaddr) -
			   __SOCKADDR_COMMON_SIZE -
			   sizeof (in_port_t) -
			   sizeof (struct in_addr)];
};
struct sockaddr {
	__uint8_t	sa_len;		/* total length */
	sa_family_t	sa_family;	/* [XSI] address family */
	char		sa_data[14];	/* [XSI] addr value (actually larger) */
};

```

通用地址结构
```c
typedef uint32_t in_addr_t;
struct in_addr
{
    in_addr_t s_addr;
};
```

> AF 表示ADDRESS FAMILY 地址族，PF 表示PROTOCOL FAMILY 协议族，但这两个宏定义是一样的，所以使用哪个都没有关系。AF_INET（又称 PF_INET）是 IPv4 网络协议的套接字类型，AF_INET6 则是 IPv6 的；而 AF_UNIX 则是 Unix 系统本地通信。

> INADDR_ANY就是指定地址为0.0.0.0的地址，这个地址事实上表示不确定地址，或“所有地址”、“任意地址”。 一般来说，在各个系统中均定义成为0值。

> htonl、htons主机字节序转换为网络字节序

## tcp 客户端

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc, char* argv[]) {
	int sock;
	struct sockaddr_in serv_addr;
	char message[30];
	int str_len;
	if (argc != 3) {
		printf("Usage : %s <IP> <port>\n", argv[0]);
		exit(1);
	}

	sock=socket(PF_INET, SOCK_STREAM, 0);//创建套接字
	if (sock == -1)
		error_handling("socket() error");

	memset(&serv_addr, 0, sizeof(serv_addr));//每字节都用0填充
	serv_addr.sin_family=AF_INET;//指定IPv4协议
	serv_addr.sin_addr.s_addr=inet_addr(argv[1]);//ip地址
	serv_addr.sin_port=htons(atoi(argv[2]));//端口号

	if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) == -1)//connect函数向服务器端发送连接请求（成功时返回 0，失败时返回-1）
		error_handling("connect() error");

	str_len=read(sock, message, sizeof(message)-1);//读取服务器传回的数据
	if (str_len == -1)
		error_handling("read() error");

	printf("Message from server : %s \n", message);
	close(sock);//关闭套接字
	return 0;
}

void error_handling(char *message) {
	fputs(message, stderr);
	fputc('\n', stderr);
	exit(1);
}
```
