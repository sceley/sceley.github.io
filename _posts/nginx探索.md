---
title: nginx探索
date: 2019-05-15 16:06:01
tags:
---

## 序言

nginx是一个高性能的http服务器。

nginx常使用于:

- 正/反向代理
- 跨越解决方案
- 静态资源服务器
- 负载均衡

<!-- more -->

## 基本配置

```
events {
}
http {
	server {
		location path {
		}
	}
}
```

### Main

nginx的全局配置，对全局生效。

```
#user  nobody;	//定义运行的用户
worker_processes  1; //定义进程数

//定义全局错误日志
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid; //定义进程pid文件
```

### Event

```
event {
	use   epoll; //[epoll | select | poll] 定义IO模型方式
	worker_connections  1024; // 定义单个进程最大连接数
}

```

### Http

```
http {
	include       mime.types;
	default_type  application/octet-stream;
	
	//定义日志格式
	#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	#                  '$status $body_bytes_sent "$http_referer" '
	#                  '"$http_user_agent" "$http_x_forwarded_for"';

	//定义访问日志
	#access_log  logs/access.log  main;
	
	//sendfile指令指定nginx是否调用sendfile函数来输出文件
	sendfile        on;
	#tcp_nopush     on;
	
	//定义连接超时时间
	#keepalive_timeout  0;
	keepalive_timeout  65;
	
	//定义响应的数据是否压缩
	#gzip  on;
	
	//定义虚拟主机
	server {
        listen       8080; //监听端口
        server_name  localhost; //设置监听域名

        #charset koi8-r;
        
		 //定以本虚拟主机的访问日志
        #access_log  logs/host.access.log  main;

        location / {
            root   html; //定义服务器的默认网站根目录位置
            index  index.html index.htm; //定义首页索引文件的名称
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        // 定义错误提示页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
	
}
```

### 内置变量

- $host	请求信息中的Host，如果请求中没有Host行，则等于设置的服务器名
- $request_method	客户端请求方法，如 GET、POST
- $remote_addr 客户端IP地址
- $remote_port 客户端端口
- $scheme	HTTP方法(如http，https)
- server_addr 服务器地址
- $server_name	服务器名称
- server_port		请求到达服务器的端口号
- $http_user_agent	客户端信息
- $http_cookie	客户端cookie信息
- $request_uri 包含请求参数的原始URI
- $uri	不带请求参数的当前URI
- $document_uri	与$uri相同
- $http_referer 客户端所在页面

### location

location 命令匹配

- ～	表示执行正则匹配，区分大小写
- ~*	表示执行正则匹配，不区分大小写
- ^~	表示普通字符开头匹配，开头匹配，一般用来匹配目录
- =   	表示普通字符精确匹配
- /		表示通用匹配，任何请求都会匹配到

匹配优先级: 首先匹配 =，其次匹配^~, 再其次按文件中顺序的正则匹配，最后是交给/通用匹配。当有匹配成功时候，停止匹配。

### rewrite

nginx通过rewrite来支持url重写

语法rewrite regex replacement [flag]

rewrite指令执行顺序：

1. 执行server块的rewrite指令(这里的块指的是server关键字后{}包围的区域)
2. 执行location匹配
3. 执行选定的location中的rewrite指令

如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件

如果循环超过10次，则返回500 Internal Server Error错误

flag标志位

- last:	继续执行后续rewrite指令集(继续重写后的url匹配，即重新第一步走)
- break:	停止执行当前虚拟主机的后续rewrite指令集(终止重写后的匹配)，break一般用在location中
- redirect: 返回302临时重定向，地址栏会显示重写后的url地址
- permanent: 返回301永久重定向，地址栏会显示重写后的url地址

## 代理

代理是在服务器和客户端之间假设的一层服务器，代理将接收客户端的请求并将它转发给服务器，然后将服务端的响应转发给客户端。

![](/img/proxy.jpg)

正向代理的 proxy_pass 代理地址是不固定的由客户端请求访问的页面决定的，而反向代理的 proxy_pass 代理地址是固定的服务器地址。

### 正向代理

正向代理是为我们服务的，即为客户端服务的，客户端可以根据正向代理访问到它本身无法访问到的服务器资源。

正向代理对我们是透明的，对服务端是非透明的，即服务端并不知道自己收到的是来自代理的访问还是来自真实客户端的访问。

```
server
{
	listen	80;
	resolver        8.8.8.8; //谷歌提供的DNS解析服务器
	location / {
		proxy_pass	$scheme://$http_host$request_uri;
	}
   access_log off;
}
```

nginx重新加载配置文件

```
$ nginx -s reload
```

配置本机的代理地址

![mac-proxy-config](/img/mac-proxy-config.png)

然后你就可以代理上网了

最后通过开发者工具查看请求，发现请求头部Remote Address字段变成了代理服务器的ip地址了，这说明代理成功

![request-header](/img/request-header.png)

> 翻墙使用的原理就是正向代理，我自己使用Shadowsocks也是

### 反向代理

> 以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

反向代理是为服务端服务的，反向代理可以帮助服务器接收来自客户端的请求，帮助服务器做请求转发，负载均衡等。

反向代理对服务端是透明的，对我们是非透明的，即我们并不知道自己访问的是代理服务器，而服务器知道反向代理在为他服务。

```
server
{
	upstream proxyserver  {
		server ip:80;
	}
	
	location / {
		proxy_pass  http://proxyserver;
	}
}

```

## 跨越解决方案

eg:

- 前端server的域名为: frontend.server.com
- 后端server的域名为: backend.server.com

在浏览器frontend.server.com向backend.server.com发起请求出现跨域。

nginx进行拦截然后由nginx代理请求:

```
server {
	listen 80;
	server_name frontend.server.com;
	location / {
		root /usr/local/src/app;
		index index.htm;
	}
	
	location ~ ^/api {
		proxy_pass backend.server.com;
	}
}
```

客户端frontend.server.com/api请求会转发到backend.server.com/api，浏览器的请求就是同源请求了。

## 负载均衡

负载均衡是将众多的客户端请求合理的分配到各个服务器，以达到服务端资源的充分利用和更少的请求时间。

```
upstream balanceServer {
	server 10.1.22.33:12345;
	server 10.1.22.34:12345;
	server 10.1.22.35:12345;
}

server {
	server_name domain.cn;
	listen 80;
	location ~ ^/api {
		proxy_pass http://balanceServer;
	}
}
```

负载均衡策略:

- 轮询（默认）每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除.
- weight 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
- ip_hash 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
- fair（第三方）按后端服务器的响应时间来分配请求，响应时间短的优先分配。

## 静态资源服务器

```
server {
	location ~* \.(png|jpg|gif|jpeg)$ {
		root	/root/static;
		expires	1d; 缓存设置
	}
}
```

本示例作为图片服务器，同时设置了缓存时间。











