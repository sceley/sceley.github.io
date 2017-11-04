---
title: 浅谈Cookie、Session、JWTS
date: 2017-11-03 22:35:10
tags: 
categories: NODEJS
---

## 序言

HTTP是一个无状态的协议，现实中的业务却是需要一定的状态的，否则无法区分用户之间的身份。

<!--more-->

### Cookie

#### 初识Cookie

HTTP是无状态的，如何标识和认证一个用户，最早的方案就是Cookie（曲奇饼）。它能记录服务器与客户端之间的状态，最早的用处就是用来判断用户是否第一次访问网站。在1997年形成规范RFC 2109，目前最新的规范为RFC 6065，它是一个由浏览器和服务器共同协作实现的规范。

Cookie的处理分为如下几步。

- 服务器向客户端发送Cookie。
- 浏览器将Cookie保存。
- 之后每次浏览器都将会将Cookie发向服务器端。

客户端发送的Cookie在请求报文的Cookie字段中。

HTTP_Parser会将所有的报文字段解析到req.headers，那么Cookie就是req.headers.cookie。根据规范中的定义，Cookie值的格式是key-value; key2=value2形式，如果我们需要Cookie，解析它也十分容易。

```js
const parseCookie = function (cookie) {
    let cookies = {};
    if(!cookie) {
        return cookies;
    }
    let list = cookie.split(';');
    for(let i = 0; i < list.length; i++) {
        let pair = list[i].split('=');
        cookies[pair[0].trim()] = pair[1];
    }
    return cookies;
};
```

在业务逻辑代码执行之前，我们将其挂载在req对象上，让业务代码可以直接访问，如下所示:

```js
function (req, res) {
    req.cookies = parseCookie(req.headers.cookie);
    handle(req, res);
}
```

这样我们的业务代码就可以进行判断处理了，如下所示:

```js
let handle = function (req, res) {
    res.writeHead(200);
    if(req.cookies.isVisit) {
        res.end('欢迎第一次来到动物园');
    } else {
        //TODO
    }
};
```

告知客户端的方式是通过响应报文实现的，响应的Cookie值在Set-Cookie字段中。它的格式与请求中的格式不太相同，规范中对它的定义如下所示:

```js
Set-Cookie: name=value; Path='/';Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=.domain.com;
```

其中name=value是必须包含的部分，其余部分皆是可选参数。这些可选参数将会影响浏览器在后续将Cookie发送给服务器端的行为。以下为主要的几个选项。

- path表示这个Cookie影响到的路径，当前访问的路径不满足该匹配时，浏览器则不发送这个Cookie。
- Expires和Max-Age是用来告知浏览器这个Cookie何时过期，如果不设置该选项，在关闭浏览器时将会丢失掉这个Cookie。如果设置了过期时间，浏览器将会把Cookie内容写入到磁盘中并保存，下次打开浏览器依旧有效。Expires的值是一个UTC格式的时间字符串，告知浏览器此Cookie何时将过期，Max-Age则告知浏览器此Cookie多久过期。前者一般而言不存在问题，但是如果服务器端的时间和客户端的时间不能匹配，这种时间设置就会存在偏差。为此，Max-Age告知浏览器这条Cookie多久之后过期，而不是一个具体的时间点。
- HttpOnly告知浏览器不允许通过脚本document.cookie去更改这个Cookie值，事实上，设置HttpOnly之后，这个值在document.cookie中不可见。但是在HTTP请求过程中，依然会发送这个Cookie到服务器端。
- Secure。当Secure值为true时，在HTTP中是无效的，在HTTPS中才有效，表示创建的Cookie只能在HTTPS连接中被浏览器传递到服务器端进行通话验证，如果是HTTP连接则不会传递该消息，所以很难被窃听到。

知道Cookie在报文头中的具体格式后，下面我们将Cookie序列化成符合规范的字符串，相关代码如下。

```js
let serialize = function (name, val, opt) {
    let pairs = [name + '=' + encode(val)];
    opt = opt || {};
    if(opt.maxAge)
}
```