---
title: TCP、UDP、Socket总结
date: 2017-10-28 22:56:46
tags: HTTP
---

## 序言

本篇将开启我对各种协议的理解总结。

<!--more-->

## UDP与TCP与Socket

### Socket

#### 定义

网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket。

Socket的英文原义是"孔"或"插座"。作为BSD UNIX的进程通信机制，取后一种意思。通常也称作"套接字"，用于描述IP地址和端口，是一个通信链的句柄，可以用来实现不同虚拟机或不同计算机之间的通信。

socket是在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用已实现进程在网络中通信。

![](/img/socket.jpg)

#### Server-Client模型

服务器，使用ServerSocket监听指定的端口，端口可以随意指定（由于1024以下的端口通常属于保留端口，在一些操作系统中不可以随意使用，所以建议使用大于1024的端口），等待客户连接请求，客户连接后，会话产生；在完成会话后，关闭连接。

客户端，使用Socket对网络上某一个服务器的某一个端口发出连接请求，一旦连接成功，打开会话；会话完成后，关闭Socket。客户端不需要指定打开的端口，通常临时的、动态的分配一个1024以上的端口。

Socket接口是TCP/IP网络的API，Socket接口定义了许多函数或例程，程序员可以用它们来开发TCP/IP网络上的应用程序。要学Internet上的TCP/IP网络编程，必须理解Socket接口。Socket接口设计者最先是将接口放在Unix操作系统里面的。如果了解Unix系统的输入和输出的话，就很容易了解Socket了。网络的Socket数据传输是一种特殊的I/O，Socket也是一种文件描述符。Socket也具有一个类似于打开文件的函数调用Socket（），该函数返回一个整型的Socket描述符，随后的连接建立、数据传输等操作都是通过该Socket实现的。

![](/img/socket.png)

#### 常用的套接字类型2中折叠常用的Socket类型

- 流式套接字（SOCK_STREAM）：面向连接的Socket，针对于面向连接的TCP服务应用。
- 数据报式套接字（SOCK_DGRAM）：无连接的Socket，对应于无连接的UDP服务应用。

### TCP

TCP服务在网络应用中十分常见，目前大多数的应用都是基于TCP搭建而成。典型的HTTP、SMTP、IMAP等协议。

TCP(传输控制协议)是面向连接的、传输可靠（保证数据正确性且保证数据顺序）、用于传输大量数据（流模式）、速度快，建立连接需要开销多（时间、系统资源）。

#### TCP服务器端

```js
const net = require('net');
const server = net.createServer(socket => {
    socket.on('data', data => {
        console.log(`接受到客户端发来的数据: ${data}`);
        socket.write('你好');
    });
    socket.on('end', () => {
        console.log('断开链接');
    });
    socket.write('欢迎光临nodejs应用\n');
});
server.listen(8124, () => {
    console.log('server bound');
});
```

#### TCP客户端

```js
const net = require('net');
const client = net.connect({port: 8124}, () => {
    console.log('client connected');
    client.write('Hello World!');
});
client.on('data', data => {
    console.log(data.toString());
    client.end();
});
client.on('end',() => {
    console.log('client disconnected');
});
```

#### TCP服务事件

##### 服务器事件

对于通过net.createServer()创建的服务器而言，它是一个EventEmiter实例，它的自定以事件有如下几种。

- listening: 在调用server.listen()绑定端口或Domain Socket后触发。
- connection: 每个客户端套接字连接到服务器端时触发。
- close:　当服务器关闭时触发, 在调用server.close()后，服务器将停止接受新的套接字连接，但保持当前存在的连接，等待所有连接都断开后，会触发该事件。
- error: 当服务器发生异常时，将会触发该事件。比如侦听一个使用中的端口，将会触发一个异常，如果不侦听error事件，服务器将会抛出异常。

##### 连接事件

服务器可以同时与多个客户端保持连接，对于每一个连接而言是典型的可写可读Stream对象。Stream对象可以用于服务器和客户端之间的通信，既可以通过data事件从一端读取另一端发来的数据，也可以通过write()方法从一端向另一端发送数据。它具有如下的自定义事件。

- data: 当一端调用write()发送数据时，另一端将会触发data事件，事件传递的数据即是write()发送的数据。
- end: 当连接中的任意一端发送了FIN数据时，将会触发该事件。
- connect: 该事件用于客户端，当套接字与服务器端连接成功时会被触发。
- drain: 当任意一端调用write()发送数据时，当前这段会触发该事件。
- error: 当异常发生时，触发该事件。
- timeout: 当一定事件后连接不再活跃时，该事件将会被触发，通知用户当前该连接已经被闲置。

另外，由于TCP套接字是可读可写的Stream对象，可以利用pipe()方法巧妙地实现管道操作，如下代码实现了一个echo服务器:
```js
const net = require('net');
const server = net.createServer(socket => {
    socket.write('Echo server \r\n');
    socket.pipe(socket);
});
server.listen(1337, '127.0.0.1');
```

### UDP

UDP(用户数据包协议)面向非连接、传输不可靠、用于传输少量数据（数据报模式）、速度快，UDP传输的可靠性由应用层负责。

UDP报头: 

![](https://p1.ssl.qhmsg.com/dr/220__/t013b598635950ce8ea.png)

TCP中连接一旦建立，所有的会话都基于连接完成，客户端如果要与另一个TCP服务通讯，需要另创建一个套接字来完成连接。但在UDP中，一个套接字可以与多个UDP服务通讯，它虽然提供面向事务的简单不可靠的信息传输服务，在网络差的情况下存在丢包严重的问题，但是由于它无须连接，资源消耗低，处理快速且灵活，所以常常应用在那种偶尔丢一两个数据包也不会产生重大影响的场景，如果音频、视频等。UDP目前应用很广泛，DNS服务器是基于它实现的。

#### 创建UDP套接字

创建UDP套接字十分简单，UDP套接字一旦创建，既可以作为客户端发送数据，也可以作为服务器端接受数据。

```js
const dgram = require('dgram');
const socket = dgram.createSocket('udp4');
```

####　创建UDP服务器端

若想让UDP套接字接收网络消息，只要调用dgram.bind(port, [address])方法对网卡和端口进行绑定即可。以下为一个完整的服务器端示例:

```js
const dgram = require('dgram');
const server = dgram.createSocket('udp4');
server.on('message', (msg, rinfo) => {
    console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});
server.on('listening', () => {
    let address = server.address();
    console.log(`server listening ${address.address}:${address.port}`);
});
server.bind(41234);
```

该套接字将接收所有网卡上41234端口上的消息。在绑定完成后，将触发listening事件。

#### 创建UDP客户端

创建一个客户端与服务器端进行对话，代码如下：

```js
const dgram = require('dgram');
const message = new Buffer("深入浅出nodejs");
const client = dgram.createSocket('udp4');
client.send(message, 0, message.length, 41234, 'localhost', (err, bytes) => {
    client.close();
});
```

当套接字对象用在客户端时，可以调用send()方法发送消息到网络中。send()方法的参数如下:

```js
socket.send(buf, offset, length, port, address, [callback]);
```

这些参数分别为要发送的Buffer、Buffer的偏移、Buffer的长度、目标端口、目标地址、发送完成后的回调。与TCP套接字的write相比,send()方法的参数列表相对复杂，但是它更灵活的地方在于可以随意发送数据到网络中的服务器端，而TCP如果要发送数据给另一个服务器端，则需要重新通过套接字构造新的连接。

#### UDP套接字事件

UDP套接字相对TCP套接字使用起来更简单，它只是一个EventEmitter的实例，而非Stream实例。它具备如下自定义事件:

- message: 当UDP套接字侦听网卡端口后，接收到消息时触发该事件，触发携带的数据为消息Buffer对象和一个远程地址消息。
- listening: 当UDP套接字开始侦听时触发该事件。
- close: 调用close()方法时触发该事件，并不在触发message事件。如需再次触发message事件，重新绑定即可。
- error: 当异常发生时触发该事件，如果不侦听，异常将直接抛出，使进程退出。