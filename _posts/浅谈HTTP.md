---
title: 浅谈HTTP
date: 2017-10-30 23:24:46
tags: HTTP
categories: NODEJS
---
## 序言

作为一个WEB开发者，了解一些常用的协议是必须的，本编将总结自己在协议方面的学习。

<!--more-->

## WEB始祖HTTP

HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。伴随着计算机网络和浏览器的诞生，HTTP1.0也随之而来，处于计算机网络中的应用层，HTTP是建立在TCP协议上，所以HTTP协议的瓶颈及其优化技巧都是基于TCP协议本身的特性，例如TCP建立连接的3次握手和断开连接的4次挥手以及每次建立连接带来的RTT延迟时间。

HTTP得以发展是W3C和IETF两个组织和作的结果，他们最终发布了一系列RFC标准，目前最知名的HTTP标准为RFC 2616。

## HTTP的基本优化

影响一个HTTP网络请求的因素主要有两个: 宽带和延迟。

宽带: 如果说我们还停留在拨号上网的阶段，宽带可能成为一个比较严重影响请求的问题，但是现在网络基础建设已经使得宽带得到极大的提升，我们就不再会担心有宽带而影响网速了。

延迟: 
- 浏览器阻塞(HOL blocking):　浏览器会因为一些原因阻塞请求。浏览器对于同一个域名，同时只能有4个连接(这个根据浏览器内核不同可能会有所差异)，超过浏览器最大连接限制，后续请求就会被阻塞。
- DNS查询(DNS Lookup): 浏览器需要知道目标服务器的IP才能建立连接。将域名解析为IP的这个系统就是DNS。这个通常可以利用DNS缓存结果达到减少这个时间的目的。
- 建立连接(initial connection): HTTP是基于TCP协议的，浏览器最快也要在第三次握手时才能捎带HTTP请求报文，达到真正的建立连接，但是这些连接无法复用会导致每次请求都经历三次握手和慢启动。三次握手在高延迟的场景下影响较为明显，慢启动则对文件类大请求影响较大。

## HTTP1.0与HTTP1.1

最早在网页中使用是在1996年，那个时候只是使用一些较为简单的网页上和网络请求上，而HTTP1.1则在1999年才开始广泛应用在现在的各大浏览器网络请求中，同时HTTP1.1也是当前使用最为广泛的HTTP协议。主要区别体现在:

### 区别

1. 缓存处理，HTTP1.0中主要使用header里的If-Modified-Since，Expires来作为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略，例如Entity，If-Unmodified-Since，If-Match，If-None-Match等更多可供选择的缓存头来控制缓存策略。
2. 宽带优化及网络连接的使用，HTTP1.0中，存在浪费宽带的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头中引入了range头域，它允许只请求资源的某个部分，即返回码206(Partial Content)，这样就方便了开发者自由的选择以便于充分选择利用宽带和连接。
3. 错误通知的管理，在HTTP1.1中新增了24个错误状态响应码，如409(Conflict)表示服务器上的某个资源被永久性的删除。
4. 长连接，HTTP1.1支持长连接和请求流水线处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection: keey-alive，一定程度上弥补了HTTP1.O每次请求都要创建连接的缺点。
5. Host头处理，在HTTP1.0中认为每一台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名(hostname)。但随着虚拟机技术的发展，在一台物理服务器上可以存在多个虚拟机，并且他们共享一个IP地址。HTTP1.1中的请求消息和响应消息都应该支持Hose头域，且请求消息中如果没有Host头域会报告一个错误(440 Bad Request)。

### 问题

1. HTTP1.0在传输数据时，每次都要重新建立连接，无疑增加了大量的延迟时间，特别是移动端表现更为突出。
2. HTTP1.x在传输数据时，所有传输的内容都是明文的，无法保证数据的安全性。
3. HTTP1.x在使用时，header里携带的内容过大，在一定程度上增加了传输成本，并且每次请求header基本不怎么变化，尤其在移动端增加用户的流量。
4. 虽然HTTP1.x支持keep-alive，来弥补多次创建连接产生的延迟，但是keep-alive使用同样会给服务器端带来大量的性能压力，并且对于单个文件被不断请求的服务（例如图片存放网站），keep-alive可能会极大的影响性能，因为它在文件被请求之后还保持了不必要的连接很长时间。

## HTTP

HTTP是无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。

HTTP是媒体独立的：这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型。

HTTP是无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

### HTTP 消息结构

#### 客户端请求消息

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：请求行（request line）、请求头部（header）、空行和请求数据四个部分组成，下图给出了请求报文的一般格式。

![](/img/httprequest.png)

#### 服务器响应消息

HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。

![](/img/httpmessage.jpg)

#### 实例

客户端请求：

```HTTP
> GET / HTTP/1.1
> Host: localhost:3000
> User-Agent: curl/7.47.0
> Accept: */*
> 
```

服务端响应:
```HTTP
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Date: Tue, 31 Oct 2017 14:11:22 GMT
< Connection: keep-alive
< Transfer-Encoding: chunked
< 
Hello World
```

### HTTP请求方法

- GET: 最常用的方法，通常用于请求服务器发送某个资源。
- POST: 起初是用来向服务器输入数据，实际上通常用来把表单数据传输到服务器。
- PUT: 与GET从服务器读取文档相反，PUT方法会向服务器写入文档。
- DELETE: 请求服务器删除指定的资源。

### HTTP状态码

HTTP状态码:

- 200 - 请求成功
- 301 - 资源（网页等）被永久转移到其它URL
- 404 - 请求的资源（网页等）不存在
- 500 - 内部服务器错误

HTTP状态码分类: 

- 1**	信息，服务器收到请求，需要请求者继续执行操作
- 2**	成功，操作被成功接收并处理
- 3**	重定向，需要进一步的操作以完成请求
- 4**	客户端错误，请求包含语法错误或无法完成请求
- 5**	服务器错误，服务器在处理请求的过程中发生了错误

### http模块

Node的http模块包含了对HTTP处理的封装。在Node中，HTTP服务继承了TCP服务器(net模块)，它能够与多个客户端保持连接，由于其采用事件驱动的形式，并不为每一个连接创建额外的线程或进程，保持很低的内存占用，所以能实现高并发。HTTP服务与TCP服务模型有区别的地方在于，在开启keepalive后，一个TCP会话可以用于多次请求和响应。TCP服务以connection为单位进行服务，HTTP服务以request为单位进行服务。http模块即是将connection到request的过程进行了封装。

![](/img/http.png)

http模块将connection到request的过程进行了封装

除此之外，http模块将连接所用套接字的读写抽象为ServerRequest和ServerResponse对象，它们分别对应请求和响应操作。在请求产生的过程中，http模块拿出连接中的传来的数据，调用二进制模块http_parser进行解析，在解析完请求报文的报头，触发request事件，调用用户的业务逻辑。该逻辑的示意图:

![](/img/theprocessofhttp.png)

#### HTTP请求

对TCP连接的读操作，http模块将其封装为ServerRequest对象。请求报文头部将会通过http_parser进行解析。

报头被解析后放置在req.headers属性上传递给业务逻辑以供调用。

报文体部分则抽象为一个只读流对象，如果业务逻辑需要读取报文体中的数据，则要在这个数据流结束后才能进行操作，如下所示: 

```js
(req, res) => {
    //console.log(req.headers);
    let buffers = [];
    req.on('data', chunk => {
        buffers.push(chunk);
    });
    res.on('end', () => {
        let buffer = Buffer.concat(buffers);
        res.end('Hello World');
    });
};
```
HTTP请求对象和HTTP响应对象是相对较底层的封装，现行的Web框架express就是在这两个对象的基础上进行高层封装完成的。

#### HTTP响应

HTTP响应封装了对底层连接的写操作，可以将其看成一个可写的流对象。它影响响应报文头部信息的API为res.setHeader()和res.writeHead()。在上述报文示例中:

```js
res.writeHead(200, {'Content-Type': 'text/plain'});
```

其分为setHeader()和writeHead()两个步骤。它在http模块的封装下，实际生成如下的报文: 

```HTTP
< HTTP/1.1 200 OK
< Content-Type: text/plain
```

我们可以调用setHeader进行多次设置，但只有调用writeHead后，报头才会写入到连接中。除此之外，http模块会自动帮你设置一些头信息，如下所示:

```HTTP
< Date: Tue, 31 Oct 2017 14:11:22 GMT
< Connection: keep-alive
< Transfer-Encoding: chunked
<
```

响应结束后，HTTP服务器将可能会将当前连接用于下一个请求，或者关闭连接。值得注意的是，报头是在报文体发送前发送的，一旦开始了数据的发送，writeHead()和setHeader()将不会再生效。这有协议的特性决定的。

### HTTP服务器

```js
const http = require('http');
const server =  http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World\n');
});
server.listen(8080, '127.0.0.1', () => {
    console.log('server runing at http://localhost:127.0.0.1:1337/');
});
```
NODE获取客户端IP:

```js
req.headers['x-forwarded-for'] || req.connection.remoteAddress || req.socket.remoteAddress;
```

NODE获取客户端PORT:
```js
req.connection.remotePort || req.socket.remotePort;
```

#### HTTP服务器的事件

如同TCP服务一样，HTTP服务器也抽象了一些事件，以供应用层使用，同样典型的是，服务器也是一个EventEmitter实例。

- connection事件: 在开始HTTP请求和响应前，客户端与服务器端需要建立底层的TCP连接，这个连接有可能因为开启了keep-alive，可以在多次请求响应之间使用；当这个连接建立时，服务器触发一次connection事件。

- request事件: 建立TCP连接后，http模块底层将在数据流中抽象出HTTP请求和HTTP响应，当请求数据发送到服务器端，在解析出HTTP请求头后，将会触发该事件；在res.end()后，TCP连接可能将用于下一次请求响应。

- close事件: 与TCP服务器的行为一致，调用server.close()方法停止接受新的连接，当已有的连接都断开时，触发该事件；可以server.close()传递一个回调函数来快速注册该事件。

- connect事件: 当客户端发起CONNECT请求时触发，而发起CONNECT请求通常在HTTP代理时出现；如果不监听该事件，发起请求的连接将会关闭。

- upgrade事件: 当客户端要求升级连接协议时，需要和服务器协商，客户端会在请求头中带上Upgrade字段，服务器端会在接受到这样的请求时触发该事件。如果不监听该事件，发起该请求的连接将会被关闭。

- clientError事件: 连接的客户触发error事件时，这个错误会传递到服务器端，此时触发该事件。

### HTTP客户端

> 从协议的角度来说，现在的应用，如浏览器，其实是一个HTTP的代理，用户的行为将会通过它转化为HTTP请求报文发送给服务器端，服务器端在处理请求后，发送响应报文给代理，代理在解析报文后，将用户需要的内容呈现在界面上。HTTP服务只做两件事情: 处理HTTP请求和发送HTTP响应。

http模块提供了一个底层API: http.request(options, connect)，用于构造HTTP客户端。

```js
const options = {
    hostname: '127.0.0.1',
    port: 1334,
    path: '/',
    method: 'GET'
};
const req = http.request(options, res => {
    console.log('STATUS: ' + res.statusCode);
    console.log('HEADERS: ' + JSON.stringify(res.headers));
    res.setEncoding('uft8');
    res.on('data', chunk => {
        console.log(chunk);
    });
});
req.end();
```

options参数决定了这个HTTP请求头中的内容，它的选项有如下这些。

- host: 服务器的域名或IP地址，默认为localhost。
- hostname: 服务器名称。
- port: 服务器端口，默认为80。
- socketPath: Domain套接字路径。
- method: HTTP请求方法，默认为GET。
- path: 请求路径，默认为/。
- headers: 请求头对象。
- auth: Basic认证，这个值将被计算成请求头中的Authorization部分。

#### 客户端事件:

- response: 与服务端的request事件对应的客户端在请求发出后得到服务器端响应时，会触发该事件。
- socket: 当底层连接池中建立的连接分配给当前请求对象时，触发该事件。
- connect: 当客户端向服务器端发起CONNECT请求时，如果服务器响应了200状态码，客户端将会触发该事件。
- upgrade: 客户端向服务器端发起Upgrade请求时，如果服务器响应了101 Switching Protocols状态，客户端将会触发该事件。
- continue: 客户端向服务器端发起Expect: 100-continue头信息，以试图发送较大数据量，如果服务器端响应100 Continue状态，客户端将触发该事件。 

> 在ClientRequest对象中，它的事件叫做response。ClientRequest在解析响应报文时，一解析完就触发response事件，同时传递一个响应对象以供操作ClientResponse。

> TCP和UDP都属于网络传输协议，如果要构造高效的网络应用，就应该从传输层进行着手。
