---
title: 浅谈Web缓存
date: 2017-10-19 08:48:57
tags: cache
categories: Performance
---

## 序言

在开发当中，性能至关重要，然而判断一个网站的性能最直观的就是看到网页打开的速度。其中提高网页反应速度的一个方式就是使用缓存。一个优秀的缓存策略可以缩短网页请求的距离，减少延迟，使缓存文件得到重复利用，减少冠带，降低网络负荷。

<!-- more -->

## 缓存分类

- 数据库缓存
- 代理服务器缓存
- CDN缓存
- 浏览器缓存

### 浏览器缓存

页面的缓存状态是由header决定的，header的参数有四种。

#### Cache-Control

其优先级要比Expires高

##### max-age

单位为s，指定设置缓存最大的有效时间，定义时间的长短。当浏览器向服务器发送请求过后，在max-age这段时间里浏览器就不会再向服务器发送请求了。

example:

服务器设置缓存。

```js
const http = require('http');
const fs = require('fs');
const server = http.createServer();
server.on('request', (req, res) => {
    if(req.url == '/') {
        res.writeHead(200, {'Content-Type': 'text/html'});
        new fs.createReadStream('./index.html').pipe(res);
    } else if(req.url == '/request') {
        res.setHeader('Cache-Control', `max-age=${60*1}`);//1minutes
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end('Hello World');
    } else if(req.url == '/favicon.ico') {
        res.end();
    }
});
server.listen(7000, 'localhost');
```

客户端通过ajax请求数据，浏览器刷新的方式，cache缓存不没有效果的，所以要通过不刷新的ajax方式才能看到效果。

![](/img/cache-control.png)

##### s-maxage

单位为s，同max-age，只是用于共享缓存(比如CDN缓存)

比如，当s-maxage=60时，在这60秒中，即使更新了CDN的内容，浏览器也不会进行请求。也就是说max-age用于普通的缓存，而s-maxage用于代理缓存。
如果存在s-maxage，则会覆盖掉max-age和Expires header

##### public

指定响应会被缓存，并且在多用户间共享。也就是下图的意思。如果没有指定public还是private，则默认为public

##### private

响应只作为私有的缓存，不能在用户间共享。如果要求HTTP认证，响应会自动设置为private

##### no-cache

```js
res.setHeader('Cache-Control', 'no-cache');
```

设置了no-cache之后并不代表浏览器不缓存，而是在使用缓存前要向服务器确认资源是否被更改。

##### no-store

绝对禁止缓存，一看就知道如果用了这个命令当然就是不会进行缓存，每次请求资源都要从服务器重新获取

#### Expires

单位为毫秒ms，缓存过期时间，用来指定资源到期的时间，是服务器端的具体的时间点。也就是说，Expires=max-age + 请求时间，需要和Last-modified结合使用。Expires是Web服务器响应消息头字段，在响应http请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，而无需再次请求。

```js
let expires = new Date(Date.now() + 1000 * 60 * 1).toUTCString();//1minute
res.setHeader('Expires', expires);
```

![](/img/expires.png)

#### Last-modified

服务器端文件的最后修改时间，需要和cache-control共同使用，是检查服务器端资源是否更新的一种方式。当浏览器再次进行请求时，会向服务器传送If-Modified-Since报头，询问Last-Modified时间点之后资源是否被修改过。如果没有修改，则返回码为304，使用缓存；如果修改过，则再次去服务器请求资源，返回码和首次请求相同为200，资源为服务器最新资源。

handle函数

```js
const handle = (req, res, filename) => {
	fs.stat(filename, (err, stat) => {
		let lastModified = stat.mtime.toUTCString();
		if(lastModified == req.headers['if-modified-since']) {
			res.writeHead(304, 'Not Modified');
			res.end();
		} else {
			res.setHeader('Last-Modified', lastModified);
			res.writeHead(200, {'Content-Type': 'text/html'});
			new fs.createReadStream(filename).pipe(res);
		}
	});
};
```

#### ETag

根据实体内容生成一段hash字符串，标识资源的状态，由服务端产生。浏览器会将这串字符串传回服务器，验证资源是否已经修改。

```js
const getHash = (str) => {
	let shasum = crypto.createHash('sha1');
	return shasum.update(str).digest('base64');
};

const handle = (req, res, filename) => {
	fs.readFile(filename, (err, filedata) => {
		let hash = getHash(filedata);
		let noneMatch = req.headers['if-none-match'];
		if (hash == noneMatch) {
			res.writeHead(304, 'Not Modified');
			res.end();
		} else {
			res.setHeader('ETag', hash);
			res.writeHead(200, {'Content-Type': 'text/html'});
			res.end(filedata);
		}
	});
};
```