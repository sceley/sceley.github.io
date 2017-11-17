---
title: 浅谈Nodejs多进程
date: 2017-11-17 11:57:10
tags: NODEJS
---

## 序言

JavaScript是运行在单进程的单线程上，但是现在的计算机大多是多核CPU的。在nodejs中，为了充分利用多核cpu，引入了child_process和cluster这两个模块来开启多进程，来充分利用多核cpu。

<!--more-->

## 多进程

### 创建子进程

使用child_process模块提供的child_process.fork()函数提供我们实现进程的复制。

worker.js

```js
const http = require('http');
const server = http.createServer();
server.on('request', (req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World\n');
});
server.listen(Math.round((1 + Math.random()) * 1000, '127.0.0.1'));
```

master.js

```js
const fork = require('child_process').fork;
const cpus = require('os').cpus();
for(let i = 0; i < cpus.length; i++) {
    fork('./worker.js');
}
```

上面的代码根据cpu的核数复制出相应Node进程数。

这是著名的Master-Worker模式，又称主从模式。分为两种进程: 主进程、工作进程。这就是典型的分布式架构中用于并行处理业务的模式，具备较好的可伸缩性和稳定性。主进程不负责具体的业务处理，而是负责调度或管理工作进程，它是趋向于稳定的。工作进程负责具体的业务处理，因为业务的多种多样，甚至一项业务有多人开发完成，所以工作进程的稳定性值得开发者关注。

![](/img/worker-master.png)

### 进程间通讯

通过send方法发送数据，通过监听message事件来接受发来的数据。通过消息传递内容，而不是共享或直接操作相关资源，这是较为轻量和无依赖的做法。

example

parent.js

```js
const cp = require('child_process');
const child = cp.fork('child.js');
child.on('message', msg => {
    console.log(`parent receive message: ${msg}`);
});
child.send({
    hello: 'world',
});
```

child.js

```js
process.on('message', msg => {
    console.log(`child receive message: ${msg}`);
});
process.send({
    foo: 'bar'
});
```

通过fork()或者其他API，创建子进程之后，为了实现父子进程之间的通讯，父进程和子进程之间将会创建IPC通道。通过IPC通道，父子进程之间才能通过message和send传递消息。

#### 进程间通信原理

请查看《Node.js深入浅出》

#### 句柄传递

