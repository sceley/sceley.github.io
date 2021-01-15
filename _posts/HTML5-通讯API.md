---
title: HTML5 通讯API
date: 2019-03-04 20:58:26
tags: Web通讯
---

## 序言

HTML5之前，前后端之间的数据交互比较单一，只能通过ajax技术实现两端之间的数据交互，而且只能由客户端向服务器端发送请求进而携带着数据发往服务器端，然后等待服务器端响应，响应中携带着客户端需要的服务器端数据，而服务器端无法主动向客户端主动发送数据。HTML5现世后，涌现出了两个新的API, WebSocket、SSE，实现了服务器端主动向客户端发往数据。
 
<!-- more -->

## WebSocket

### 客户端实现方式

```js
const webSocket = new WebSocket("ws://localhost:8080"); //与服务器建立连接
//url必须以"ws"或"wss"开头(加密通讯时)文字开头

webSocket.send(data);//向服务器发送数据
//data可以是JSON对象转换为文本数据, 或着blob对象或arraybuffer对像

webSocket.onmassage = function (e) {
    let massage = e.data;
};//监听服务器端发来的数据

websocket.onopen = function () {

};//监听open事件，进而判断是否连接成功

webSocket.onclose = function () {

};//监听close事件，进而判断连接是否关闭

webSocket.close();切断连接

websocket.binaryType = arraybuffer || blob;//设置接受数据类型

```


### 服务器端实现方式

使用Node.js的ws、 socket.io模块来实现

#### WS实现

```js
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

// Broadcast to all.(向所有连接的socket发送数据)
wss.broadcast = (data) => {
     wss.clients.forEach((client) => {
        if (client.readyState === WebSocket.OPEN) {
              client.send(data);
           }
      });
};

wss.on('connection', (ws) => {
    ws.on('message', (data) => {
        // Broadcast to everyone else.(像其他人广播数据)
        wss.clients.forEach((client) => {
            if (client !== ws && client.readyState === WebSocket.OPEN) {
                client.send(data);
            }
        });
    });
});

```

结合Express一起使用

```js
const express = require('express');
const http = require('http');
const url = require('url');
const WebSocket = require('ws');

const app = express();
const server = http.createServer(app);
const wss = new WebSocket.Server({ server });

app.use((req, res) => {
  res.send({ msg: "hello" });
});

wss.on('connection', (ws) => {
  const location = url.parse(ws.upgradeReq.url, true);
  // You might use location.query.access_token to authenticate or share sessions
  // or ws.upgradeReq.headers.cookie
  ws.on('message', (message) => {
    console.log('received: %s', message);
  });
  ws.send('something');
});

server.listen(8080, () => {
  console.log('Listening on %d', server.address().port);
});
```

#### SOCKET.IO实现方式

```js
//服务端建立socket
const io = require("socket.io");//服务器引入socket.io
const socket = io.listen(server);//server为一个http服务器

//客户端建立socket
<script src="/socket.io/socket.io.js"></script>//客户端引入socket.io
const socket = io();

//公共API
socket.on('connection', callback);//监听连接事件

socket.on('disconnection', callback);//监听断开连接事件

socket.on("message", callback);//监听接受消息事件

socket.send();//向对端发送数据

socket.emit(event, data, [callback])；//发布事件，callback由对方进行调用及形参的传进

socket.on(event, function (data, fn) {});//事件监听

socket.once();//监听事件，只触发一次

保存数据:
socket.set(name, value, [callback]);
socket.get(name, callback)
//callback: (err, docs) => {};

广播消息:
io.sockets.
socket.broadcast.

使用命名空间:
io.of(namespace)
const chat = io.of("/chat");
chat.on("connection", callback);//监听连接

const chat = io.connect("http://localhost/chat");//建立连接
```

## SSE

是一种从服务器端发往客户端的单向通信机制，由服务器端发送一些事件, 再由客户端接收这些事件

### 客户端实现方式

```js
const source = new EventSource(url);
source.onmessage = (e) => {e.data};
source.onopen = () => {};
source.onerror = (e) => {};
source.close();
```

### 服务器端实现方式

服务器端发送的数据只能以一个字符串的形式发送，对象必须转为JSON字符串进行发送

```js
res.writeHeader(200, {"Content-Type": "text/event-stream"}, "Cache-Control": "no-cache"); //必须指定浏览器不缓存服务器发送的数据
```

发送事件

```
echo "id: requests \n\n"
echo "data: 数据 \n\n"
echo "retry: 5000\n\n"
```

//每行数据之前必须书写"data: "前缀，同时在行结尾需书写"\n\n"换行标志
//指定客户端每隔多长时间与服务器建立一次连接并获取事件流，不指定由浏览器决定


