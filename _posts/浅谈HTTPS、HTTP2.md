---
title: 浅谈HTTPS、HTTP2
date: 2017-11-11 16:38:58
tags: HTTP
---

## 序言

随着科技的发展，网络通讯对安全的要求越来越高。在如今，很多企业都在逐渐的把自己的网站升级为HTTPS、HTTP2，提高网站的安全可靠性。本编文章将总结自己在HTTPS、HTTP2上的学习。

<!--more-->

## HTTPS协议的介绍

- HTTPS协议需要到CA申请证书。
- HTTPS协议运行在SSL/TLS之上，SSL/TLS运行在TCP之上。
- HTTPS使用443端口。
- HTTPS可以方式有效防止运营商劫持。
- HTTPS降低用户访问速度。SSL握手，HTTPS对速度会有一定程度的影响。

## SPDY的介绍

- 降低延迟，针对HTTP高延迟的问题，SPDY采用多路复用。(多路复用: 多个请求stream共享一个tcp连接)
- 请求优先级。多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。
- header压缩。
- 基于HTTPS的加密协议传输。
- 服务器推送。(例如:网页有一个style.css请求，在客户端收到style.css数据的同时，服务器会将style.js的文件推送给客户端，当客户端再次尝试获取style.js时就可以直接从缓存中获取到)。
- SPDY在SSL之上在HTTP之下。

## HTTP2协议的介绍

HTTP2.0可以说是SPDY的升级版。

- 二进制协议，HTTP1.x的解析是基于文本，HTTP2.0的协议解析采用二进制格式。HTTP1.x的头信息肯定是文本，数据体可以是文本也可以是二进制；HTTP2.0是彻底
- 多路复用。
- header压缩，HTTP2.0使用encoder来减少传输的header大小，通讯双方各自cache一份header fileds表，即避免了重复header的传输，又减小了需要传输的大小。
- 服务器推送。
- 支持明文HTTP传输，而SPDY强制使用HTTPS。

## 签名证书

自签名证书，就是自己扮演CA机构，给自己的服务器端颁发签名证书。以下为生成私钥、生成CSR文件、通过私钥自签名生成证书的过程。

```
$ openssl genrsa -out ca.key 1024
$ openssl req -new -key ca.key -out ca.csr
$ openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

流程图如下

![](/img/yourselfsign.png)

接下来到服务器端，服务器端需要向CA机构申请签名证书。在申请签名证书之前依然是要创建自己的CSR文件。值得注意的是，这个过程中的Common Name要匹配服务器域名，否则在后续认证过程中会出错。如下是生成命令:

```
$ openssl genrsa -out server.key 1024
$ openssl req -new -key server.key -out server.csr
```

![](/img/servercsr.png)

得到CSR文件后，向我们自己的CA机构申请签名。签名需要CA的证书和私钥参与，最终颁发一个带有CA签名的证书，如下所示: 

```
$ openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
```

## HTTPS服务器

```js
const https = require('https');//https模块
const fs = require('fs');
const opts = {
    key: fs.readFileSync('./server.key'),//私钥
    cert: fs.readFileSync('./server.crt'),//数字证书(签名证书)
};
const server = https.createServer(opts, function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World');
});
server.listen(8443);
```

客户端访问结果

![](/img/https.png)

## HTTP2服务器

首先先下载第三方HTTP2

```
$ cnpm install http2 --save-dev
```

然后编写代码

```js
const http2 = require('./node_modules/http2');
const fs = require('fs');

const options = {
	key: fs.readFileSync('./server.key'),
	cert: fs.readFileSync('./server.crt'),
};

const server = http2.createServer(options);
server.on('request', function (req, res) {
	res.writeHead(200, {'Content-Type': 'text/plain'});
	res.end('Hello World');
});
server.listen(8443);
```

查看网络: 

![](/img/http2firefox.png)

![](/img/http2chrome.png)

## 通过Let'Encrypt获取第三方CA

自己充当CA自颁发数字证书虽然方便，但是客户端并不信任该证书，因为客户端并不包含用于签名的CA证书。

所以咱们使用Let'Encrypt机构给我们的服务器颁发证书。

使用certbot获取证书，certbot的官方介绍:

Certbot, previously the Let's Encrypt Client, is EFF's tool to obtain certs from Let's Encrypt, and (optionally) auto-enable HTTPS on your server.

安装certbot

```
$ wget https://dl.eff.org/certbot-auto
$ chmod a+x certbot-auto
$ ./certbot-auto
$ sudo certbot certonly
```