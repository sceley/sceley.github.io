---
title: 浅谈Node-WebSocket
date: 2017-11-03 11:12:45
tags: NODEJS
---

## 序言

WebSocket在客户端与服务器端的通讯中占有重要的地位，基于WebSocket客户端可以随时向服务器端发送数据，服务器也可以随时向客户端发送数据。多人在线聊天室就是基于WebSocket构建的。

<!--more-->

## 构建WebSocket服务

Websocket与Node之间的配合堪称完成完美，其理由有两条:

- WebSocket客户端基于事件的编程模型与Node中自定义事件相差无几。
- WebSocket实现了客户端与服务器端之间的长连接，而Node事件驱动的方式十分擅长与大量客户端保持高并发连接。

### WebSocket的介绍

- 客户端与服务器端只建立一个TCP连接，可以使用更少的连接。
- WebSocket服务器可以推送数据到客户端，这远比HTTP请求响应模式更灵活、更高效。
- 有更轻量级的协议头，减少数据传送量。

在WebSocket之前，网页客户端与服务器端进行通信最高效的是Comet技术。实现Comet技术的细节是采用长轮询(long-polling)或iframe流。

WebSocket与HTTP相比，它更接近于传输层协议，它并没有在HTTP的基础上模拟服务器端的推送，而是在TCP上定义独立的协议。让人迷惑的部分在于它的握手部分是由HTTP完成的，使人觉得它可能是基于HTTP实现的。

WebSocket协议主要分为两个部分: 握手和数据传输。

### WebSocket握手

客户端建立连接时，通过HTTP发起请求报文，如下所示:

```HTTP
GET /chat HTTP/1.1
Host: server.exaple.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGh1IHNhbXBsZSBub25jZQ==
Sec-WebSocket-Protocol: chat，superchat
Sec-WebSocket-Version: 13
```

与普通的HTTP请求协议略有区别的部分在于如下这些协议头:

```HTTP
Upgrade: websocket
Connection: Upgrade
```

上述两个字段表示请求服务器端升级协议头为WebSocket。其中Sec-WebSocket-Key用于安全校验: 

```HTTP
Sec-WebSocket-Key: dGh1IHNhbXBsZSBub25jZQ==
```

Sec-WebSocket-Key的值是随机生成的Base64编码的字符串。服务器端接收到之后将其与与字符串258EAFA5-E914-47DA-95CA-C5ABoDC85B11相连，形成字符串dGh1IHNhbXBsZSBub25jZQ==258EAFA5-E914-47DA-95CA-C5ABoDC85B11，然后通过sha1安全散列算法计算出结果后，在进行Base64编码，最后返回给客户端。这个算法如下所示:

```js
const crypto = require('crypto');
const val = crypto.crateHash('sha1').update(key).digest('base64');
```

另外，下面两个字段指定子协议和版本号: 

```HTTP
Sec-WebSocket-Protocol: chat，superchat
Sec-WebSocket-Version: 13
```

服务器端在处理完请求后，响应如下报文

```HTTP
HTTP/1.1 101 Switching Protocols
Upgrade: webSocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTaxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

上面的报文告之客户端正在更换协议，更新应用层协议为WebSocket协议，并在当前的套接字连接上应用新协议。剩余字段分别表示服务器端基于Sec-WebSocket-Key生成的字符串和选中的子协议。客户端将会校验Sec-WebSocket-Accept的值，如果成功，将开始接下来的数据传输。

#### Node构建WebSocket客户端

```js
function parseUrl (url) {
    let hostportpath = url.split('//')[1];
    let host = hostportpath.split(':')[0];
    let portpath = hostportpath.split(':')[1];
    let port = portpath.split('/')[0];
    return {
        port,
        host
    };
};

const WebSocket = function (url) {
    //伪代码，解析ws://127.0.0.1:12010/updates，用于请求
    this.options = parseUrl(url);
    this.connect();
};
WebSocket.prototype.onopen = function () {
    //TODO
};
WebSocket.prototype.setSocket = function (socket) {
    this.socket = socket;
};
WebSocket.prototype.connect = function () {
    let key = new Buffer(this.options.protocolVersion + '-' + Date.now()).toString('base64');
    let shasum = crypto.createHash('sha1');
    let expected = shasum.update(key + '258EAFA5-E914-47DA-95CA-C5ABoDC85B11').digest('base64');
    let options = {
        port: this.options.port,
        host: this.options.hostname,
        headers: {
            'Connection': 'Upgrade',
            'Upgrade': 'websocket',
            'Sec-WebSocket-Key': key
            'Sec-WebSocket-Version': 13
        }
    };
    let req = http.request(options, (res, socket, upgradeHead) => {
        this.setSocket(socket);
        this.onopen();
    });
    req.end();
};
```

#### Node构建WebSocket服务器

```js
const http = require('http');
const crypto = require('crypto');
const server = http.createServer();
server.on('request', (req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World');
});
server.listen(4000, 'localhost', () => {
    console.log('http://localhost:4000');
});
server.on('upgrade', (req, socket, upgradeHead) => {
    let head = new Buffer(upgradeHead.length);
    upgradeHead.copy(head);
    let key = req.headers['sec-websocket-key'];
    let shasum = crypto.createHash('sha1');
    key = shasum.update(key + '258EAFA5-E914-47DA-95CA-C5AB0DC85B11').digest('base64');
    let headers = [
        'HTTP/1.1 101 Switching Protocols',
        'Upgrade: websocket',
        'Connection: Upgrade',
        'Sec-WebSocket-Accept: ' + key,
    ];
    
    socket.setNoDelay(true);
    socket.write(headers.concat('', '').join('\r\n'));
    let data = '';
    socket.on('data', chunk => {
        data += chunk;
    });
    socket.on('end', () => {
        console.log(data);
        socket.write('nodejs');
    });
});
```

### WebSocket数据传输

在握手顺利完成后，当前连接将不再进行HTTP的交互，而是开始WebSocket的数据帧协议，实现客户端与服务器端的数据交换。下图为协议升级过程示意图。

![](/img/websocketprotocol.png)

为了安全考虑，客户端需要对发送的数据帧进行掩码处理，服务器一旦收到无掩码帧(比如中间拦截破坏)，连接将关闭。而服务器发送到客户端的数据帧则无须做掩码处理，同样，如果客户端收到带掩码的数据帧，连接也将关闭。