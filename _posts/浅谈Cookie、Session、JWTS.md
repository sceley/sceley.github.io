---
title: 浅谈Cookie、Session、JWTS
date: 2017-11-03 22:35:10
tags: NODEJS
---

## 序言

HTTP是一个无状态的协议，现实中的业务却是需要一定的状态的，否则无法区分用户之间的身份。

<!--more-->

## Cookie

### 初识Cookie

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
    if(opt.maxAge) pairs.push(`Max-Age=${opt.maxAge}`);
    if(opt.domain) pairs.push(`Domain=${opt.domain}`);
    if(opt.path) pairs.push(`Path=${opt.path}`);
    if(opt.expires) pairs.push(`Expires=${opt.expires.toUTCString()}`);
    if(opt.httpOnly) pairs.push(`HttpOnly`);
    if(opt.secure) pairs.push(`Secure`);
    return pairs.join(`; `);
};
```

判断用户的状态，如下所示:

```js
let handle = function (req, res) {
    if(!req.cookies.isVisit) {
        req.setHeader(`Set-Cookie, ${serialize('isVisit', 1)}`);
        res.writeHead(200);
        res.end('欢迎第一次来动物园');
    } else {
        res.writeHad(200);
        res.end(`动物园再次欢迎你`);
    }
};
```

客户端收到这个带Set-Cookie的响应后，在之后的请求时会在Cookie字段中带上这个值。

Set-Cookie是较少的，在报头中可能存在多字段。为此res.setHeader的第二个参数可以是一个数组，如下所示:

```js
Set-Cookie: foo=bar; Path=/; Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=.domain.com
Set-Cookie: foo=val; Path=/; Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=.domain.com
```

### Cookie的性能影响

由于Cookie的实现机制，一旦服务器端向客户端发送了设置Cookie的意图，除非Cookie过期，否则客户端每次请求都会发送这些Cookie到服务器端，一旦设置的Cookie过多，将会导致报头较大。大多数的Cookie并不需要每次都用上，因为这会造成宽带的部分浪费。在YSlow的性能优化规则中有这么一条:

- 减小Cookie的大小
- 为静态组件使用不同的域名
- 减少DNS查询

> 如果在域名的根节点设置Cookie，几乎所有的子路径下的请求都会带上这些Cookie

### Session

Cookie并非完美，前文提及的体积过大就是一个显著的问题，最为严重的问题是Cookie可以在前后端进行修改，因此数据就极容易被篡改和伪造。

Session的数据只保留在服务器端，客户端无法修改，这样数据的安全性得到一定的保障，数据也无须在协议中每次都被传递。

将每个客户端和服务器中的数据一一对应起来的两种实现方式。

#### 基于Cookie来实现用户和数据的映射。(口令sessionid)

```js
let sessions = {};
let key = 'session_id';
let EXPIRES = 20 * 60 * 1000;

let generate = function () {
    let session = {};
    session.id = (new Date()).getTime() + Math.random();
    session.cookie = {
        expires: (new Date()).getTime() + EXPIRES
    };
    sessions[session.id] = session;
    return session;
} ;
```

每一次请求到来时，检查Cookie中的口令与服务器端的数据，如果过期，就重新生成，如下所示:

```js
function (req, res) {
    let id = req.cookies[key];
    if(!id) {
        req.session = generate();
    } else {
        let session = sessions[id];
        if(session) {
            if(session.cookie.expires > (new Date()).getTime() ) {
                //更新超时时间
                session.cookie.expires = Date.now() + EXPIRES;
                req.session = session;
            } else {
                //如果超时了，删除旧数据，并重新生成。
                delete sessions[id];
                req.session = gennerate();
            }
        } else {
            //如果session过期或口令不对，重新生成session
            res.session = generate();
        }
    }
    handle(req, res);
};
```

把重新生成的Session响应给客户。

```js
let writeHead = res.writeHead;
res.writeHead = function () {
    let cookies = res.getHeader('Set-Cookie');
    let session = serialize(key, req.session.id);
    cookies = Array.isArray(cookies) ? cookies.concat(session) : [cookies, session];
    res.setHeader('Set-Cookie', cookies);
    return writeHead.apply(this, arguments);
};
```

Sesion依赖于Cookie实现，是目前大多数Web应用的方案。

#### 通过查询字符串来实现浏览器和服务器端数据对应

略...

### Session问题

#### Session与内存

#### Session与安全

口令保存在客户端，口令可能会被盗取，口令可能会被伪造。

对口令通过私钥加密进行签名。

```js
let sign = function (val, secret) {
    return val + '.' + crypto
        .createHmac('sha256', secret)
        .update(val)
        .digest('base64')
        .replace(/\=+$/, '');
};
```

在响应时，设置session值到Cookie中，如下所示:

```js
let val = sign(req.sessionID, secret);
res.setHeader('Set-Cookie', cookie.serialize(key, val));
```

接收请求时，检查签名

```js
let unsign = function (val, secret) {
    let str = val.slice(0, val.lastIndex('.'));
    return sign(str, secret) == val ? str : false;
};
```

### JSON Web Token (JWT)

[jwt](https://jwt.io)官方介绍

JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA.

#### jwt的组成

一个jwt实际上就是一个字符串，它由三部分组成，头部、载荷与签名。

#### 头部(Header)

The header typically consists of two parts: the type of the token, which is JWT, and the hashing algorithm being used, such as HMAC SHA256 or RSA.

用于描述关于该JWT的最基本信息，类型与签名所用的算法。

for example

```js
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Then, this JSON is Base64Url encoded to form the first part of the JWT.

#### 载荷(Payload)

The second part of the token is the payload, which contains the claims. Claims are statements about an entity (typically, the user) and additional metadata. There are three types of claims: reserved, public, and private claims.

- Reserved claims: These are a set of predefined claims which are not mandatory but recommended, to provide a set of useful, interoperable claims. Some of them are: iss (issuer), exp (expiration time), sub (subject), aud (audience), and others.
- Public claims: These can be defined at will by those using JWTs. But to avoid collisions they should be defined in the IANA JSON Web Token Registry or be defined as a URI that contains a collision resistant namespace.
- These are the custom claims created to share information between parties that agree on using them.

An example of payload could be:

```js
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

The payload is then Base64Url encoded to form the second part of the JSON Web Token

#### 签名(signature)

To create the signature part you have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that.

For example if you want to use the HMAC SHA256 algorithm, the signature will be created in the following way:

```js
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

The signature is used to verify that the sender of the JWT is who it says it is and to ensure that the message wasn't changed along the way.

#### 放它们到一起

The output is three Base64 strings separated by dots that can be easily passed in HTML and HTTP environments.

The following shows a JWT that has the previous header and payload encoded, and it is signed with a secret. 

![](https://cdn.auth0.com/content/jwt/encoded-jwt3.png)

#### JWT是怎么样工作的

![](https://cdn.auth0.com/content/jwt/jwt-diagram.png)

#### JWT与Session的区别

![](/img/session.png)

![](/img/token.png)