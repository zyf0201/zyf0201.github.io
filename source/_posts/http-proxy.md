---
title: 配置HTTP正向代理服务器
date: 2021-05-27 10:17:55
tags: 
    - openrestry
    - nginx
    - http proxy
---


> 在政府金融行业，或者一些特殊的网络隔离情况下，内网主机是无法访问互联网的，这个时候就需要特定的正向代理服务器才能实现连接互联网的需求。Nginx是一个异步框架的网页服务器，也可以用作反向代理、负载平衡器和HTTP缓存，同时，Nginx也可以作为一个正向代理服务器。


# HTTP / HTTPS 正向代理的分类

- 按客户端有无感知的分类
  - 普通代理：在客户端需要在浏览器中或者系统环境变量手动设置代理的地址和端口
  - 透明代理：客户端不需要做任何代理设置，“代理”这个角色对于客户端是透明的。如企业网络链路中的 Web Gateway 设备。
  
- 按代理是否解密 HTTPS 的分类
  - 隧道代理：也就是透传代理代理服务器只是在 TCP 协议上透传 HTTPS 流量，对于其代理的流量的具体内容不解密不感知客户端和其访问的目的服务器做直接 TLS / SSL 交互本文中讨论的 NGINX 代理方式属于这种模式。
  - 中间人（MITM，Man-in-the-Middle）代理：代理服务器解密 HTTPS 流量，对客户端利用自签名证书完成 TLS / SSL 握手，对目的服务器端完成正常 TLS 交互。(注：这种情况客户端在 TLS 握手阶段实际上是拿到的代理服务器自己的自签名证书，证书链的验证默认不成功，需要在客户端信任代理自签证书的根 CA 证书。所以过程中是客户端有感的。如果要做成无感的透明代理，需要向客户端推送自建的根 CA 证书，在企业内部环境下是可实现的。)


# Nginx直接代理HTTP

首先编译按安装nginx，我用tengine代替，以macos为例

> Tengine是一个由淘宝从Nginx复刻出来的HTTP服务器，现时版本为2.2.2。Tengine对Nginx的修改版本是于2011年12月开始释放出来成为开源项目，两者配置兼容。

- tengine安装
```bash
wget https://tengine.taobao.org/download/tengine-2.3.3.tar.gz
tar -xvf tengine-2.3.3.tar.gz 
cd tengine-2.3.3
./configure --with-cc-opt="-I/usr/local/opt/openssl/include/ -I/usr/local/opt/pcre/include/" \
--with-ld-opt="-L/usr/local/opt/openssl/lib/ -L/usr/local/opt/pcre/lib/"
make && make install
```

- nginx.conf配置
http代理配置：
```
server {
        listen 80;                #监听端口
        resolver 8.8.8.8;   #dns解析地址
        location / {
            proxy_pass http://$http_host$request_uri;
        }
    }
```

- 测试http代理连接
```
# -i参数打印出服务器回应的 HTTP 标头,-v参数输出通信的整个过程，用于调试
curl http://www.sina.com -v -i --proxy 127.0.0.1:80
```
- 查看对应的返回
```
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET http://www.sina.com/ HTTP/1.1
> Host: www.sina.com
> User-Agent: curl/7.76.1
> Accept: */*
> Proxy-Connection: Keep-Alive
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
< Server: Tengine/2.3.3
Server: Tengine/2.3.3
< Date: Mon, 31 May 2021 11:53:02 GMT
Date: Mon, 31 May 2021 11:53:02 GMT
< Content-Type: text/html
Content-Type: text/html
< Content-Length: 24000
Content-Length: 24000
< Connection: keep-alive
Connection: keep-alive
< Vary: Accept-Encoding
Vary: Accept-Encoding
< ETag: W/"607e94fd-c2c"V=5965C31
ETag: W/"607e94fd-c2c"V=5965C31
< X-Powered-By: shci_v1.13
X-Powered-By: shci_v1.13
< Expires: Mon, 31 May 2021 11:53:10 GMT
Expires: Mon, 31 May 2021 11:53:10 GMT
< Cache-Control: max-age=120
Cache-Control: max-age=120
< X-Via-SSL: ssl.25.sinag1.shx.lb.sinanode.com
X-Via-SSL: ssl.25.sinag1.shx.lb.sinanode.com
< Edge-Copy-Time: 1622461870071
Edge-Copy-Time: 1622461870071
```

- 也可以通过修改本地hosts解析进行测试

- http代理的原理

{% asset_img web_proxy.png http-proxy %}

> 代理服务器类似于一个中间人的角色，假如我通过代理访问 A 网站（以www.sina.com为例），对于 A 来说，它会把代理当做客户端，完全察觉不到真正客户端的存在，这实现了隐藏客户端 IP 的目的。当然代理也可以修改 HTTP 请求头部，通过 X-Forwarded-IP 这样的自定义头部告诉服务端真正的客户端 IP。


# Nginx代理HTTPS请求

之前我们搭建了http代理服务器，但是我们会发现上述的代理是无法代理https流量的，这是为什么呢？因为作为反向代理时，代理服务器通常终结（终止）HTTPS 加密流量，再转发给后端实例.HTTPS 流量的加解密和认证过程发生在客户端和反向代理服务器之间。
而作为正向代理在处理客户端发过来的流量时，HTTP 加密封装在了 TLS / SSL 中，代理服务器无法看到客户端请求的 URL 中想要访问的域名，如下图。所以代理 HTTPS 流量，相比于 HTTP，需要做一些特殊处理。

{% asset_img https-proxy.png https-proxy %}

这种代理的本质是中间人，而 HTTPS 网站的证书认证机制是中间人劫持的克星。普通的 HTTPS 服务中，服务端不验证客户端的证书，中间人可以作为客户端与服务端成功完成 TLS 握手；但是中间人没有证书私钥，无论如何也无法伪造成服务端跟客户端建立 TLS 连接。当然如果你拥有证书私钥，代理证书对应的 HTTPS 网站当然就没问题了。

## HTTP CONNECT 隧道
NGINX 解决 HTTPS 代理的方式都属于透传（隧道）模式，即不解密不感知上层流量。具体的方式有七层和四层的两类解决方案。

> HTTP 客户端通过 CONNECT 方法请求隧道代理创建一条到达任意目的服务器和端口的 TCP 连接，并对客户端和服务器之间的后继数据进行盲转发。

- 整个过程可以参考 HTTP 权威指南中的图：
{% asset_img web_tunnel.png web_tunnel %}
  - 客户端给代理服务器发送 HTTP CONNECT 请求。
  - 代理服务器利用 HTTP CONNECT 请求中的主机和端口与目的服务器建立 TCP 连接。
  - 代理服务器给客户端返回 HTTP 200 响应。
  - 客户端和代理服务器建立起 HTTP CONNECT 隧道，HTTPS 流量到达代理服务器后，直接通过 TCP 透传远端目的服务器。代理服务器的角色是透传 HTTPS 流量，并不需要解密 HTTPS。

> NGINX 作为反向代理服务器，官方一直没有支持 HTTP CONNECT 方法。但是基于 NGINX 的模块化，可扩展性好的特性，阿里的 @chobits 提供了ngx_http_proxy_connect_module模块，来支持 HTTP CONNECT 方法，从而让 NGINX 可以扩展为正向代理。

- tengine安装并启用ngx_http_proxy_connect_module

> 使用nginx也是一样的，但是暂时不要使用openresty，因为我用openresty+ngx_http_proxy_connect_module发现代理是有问题的，详细原因请参考此[链接](https://github.com/chobits/ngx_http_proxy_connect_module/issues/26)

macos为例，首先需要通过brew安装openssl和pcre
```bash
# 重新编译安装
./configure --with-cc-opt="-I/usr/local/opt/openssl/include/ -I/usr/local/opt/pcre/include/" \
--with-ld-opt="-L/usr/local/opt/openssl/lib/ -L/usr/local/opt/pcre/lib/"
--add-module=./modules/ngx_http_proxy_connect_module

make && make install
```

- nginx.conf配置

修改配置文件
```conf
worker_processes  1;        #nginx worker 数量
error_log logs/error.log;   #指定错误日志文件路径
events {
    worker_connections 1024;
}

http {

    server {
        resolver 8.8.8.8;   #dns解析地址
        listen 443;          #代理监听端口
        proxy_connect;
        proxy_connect_allow            all;
        location / {
            proxy_pass https://$host$request_uri;     #设定https代理服务器的协议和地址
            proxy_set_header HOST $host;
            proxy_buffers 256 4k;
            proxy_max_temp_file_size 0k;
            proxy_connect_timeout 30;
            proxy_send_timeout 60;
            proxy_read_timeout 60;
            proxy_next_upstream error timeout invalid_header http_502;
                }
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
            root   html;
        }
    }
}
```

- 测试连接
```bash
curl https://www.baidu.com -v -i --proxy 127.0.0.1:443
```
{% asset_img curl_https.png https-tunnel %}
通过-v参数调试可以看到对应的连接过程，客户端首先跟代理服务器（127.0.0.1）建立HTTP CONNECT隧道，代理回复 HTTP / 1.1 200 连接建立后就开始交互 TLS / SSL 握手和流量了。


## NGINX流（四层解决方案）

> NGINX 官方从 1.9.0 版本开始支持ngx_stream_core_module模块，模块默认不建立，需要配置时加上–with 流选项来开启。

- [ngx_stream_ssl_preread_module](http://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html?spm=a2c4e.10696291.0.0.e01e19a41l7ou4) 模块

要在不解密的情况下拿到 HTTPS 流量访问的域名，只有利用 TLS / SSL 握手的第一个客户端 Hello 报文中的扩展地址 SNI（服务器名称指示）来获取.NGINX 官方从 1.11.5 版本开始支持利用ngx_stream_ssl_preread_module模块来获得这个能力，模块主要用于获取客户端 Hello 报文中的 SNI 和 ALPN 信息。对于 4 层正向代理来说，从客户端 Hello 报文中提取 SNI 的能力是至关重要的，否则 NGINX stream 的解决方案无法成立。同时这也带来了一个限制，要求所有客户端都需要在 TLS / SSL 握手中带上 SNI 字段，否则 NGINX stream 代理完全没办法知道客户端需要访问的目的域名。

- 重新编译安装tengine并启用ngx_stream_ssl_preread_module

```bash
# 重新编译安装，并增加--with-stream,--with-stream_ssl_preread_module,--with-stream_ssl_module参数
./configure --with-cc-opt="-I/usr/local/opt/openssl/include/ -I/usr/local/opt/pcre/include/" \
--with-ld-opt="-L/usr/local/opt/openssl/lib/ -L/usr/local/opt/pcre/lib/" \
--add-module=./modules/ngx_http_proxy_connect_module \
--with-stream \
--with-stream_ssl_preread_module \
--with-stream_ssl_module
```

- nginx.conf配置
```conf
#user  nobody;
worker_processes  1;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;
events {
    worker_connections  1024;
}

stream {
    resolver 8.8.8.8;
    server {
        listen 443;
        ssl_preread on;
        proxy_connect_timeout 5s;
        proxy_pass $ssl_preread_server_name:$server_port;
    }
}
```

- 测试连接

四层正向代理是透传上层 HTTPS 流量，不需要 HTTP 连接来建立隧道，也就是说不需要客户端设置 HTTP（S）代理。如果我们在客户端手动设置 HTTP（S）代理是否能访问成功呢？我们可以用 curl --proxy 来设置代理为这个正向服务器访问测试，看看结果：
{% asset_img curl_proxy_stream.png curl_proxy_stream %}
可以看到客户端试图于正向 NGINX 前建立 HTTP CONNECT 隧道，但是由于 NGINX 是透传，所以把 CONNECT 请求直接转发给了目的服务器。目的服务器不接受 CONNECT 方法，所以最终出现“curl: (56) Proxy CONNECT aborted”，导致访问不成功。


对于四层正向代理，NGINX对上层流量基本上是透传，也不需要HTTP CONNECT来建立隧道。适合于透明代理的模式，比如将访问的域名利用DNS解定向到代理服务器。我们可以通过在客户端绑定/etc/hosts中来模拟。

增加本地解析
```bash
# /etc/hosts
127.0.0.1 www.baidu.com
```

直接curl测试
```bash
*   Trying 127.0.0.1:443...
* Connected to www.baidu.com (127.0.0.1) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=CN; ST=beijing; L=beijing; OU=service operation department; O=Beijing Baidu Netcom Science Technology Co., Ltd; CN=baidu.com
*  start date: Apr  2 07:04:58 2020 GMT
*  expire date: Jul 26 05:31:02 2021 GMT
*  subjectAltName: host "www.baidu.com" matched cert's "*.baidu.com"
*  issuer: C=BE; O=GlobalSign nv-sa; CN=GlobalSign Organization Validation CA - SHA256 - G2
*  SSL certificate verify ok.
> GET / HTTP/1.1
> Host: www.baidu.com
> User-Agent: curl/7.76.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
< Accept-Ranges: bytes
Accept-Ranges: bytes
< Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
< Connection: keep-alive
Connection: keep-alive
< Content-Length: 2443
Content-Length: 2443
< Content-Type: text/html
Content-Type: text/html
< Date: Tue, 01 Jun 2021 06:36:24 GMT
Date: Tue, 01 Jun 2021 06:36:24 GMT
< Etag: "588603eb-98b"
Etag: "588603eb-98b"
< Last-Modified: Mon, 23 Jan 2017 13:23:55 GMT
Last-Modified: Mon, 23 Jan 2017 13:23:55 GMT
< Pragma: no-cache
Pragma: no-cache
< Server: bfe/1.0.8.18
Server: bfe/1.0.8.18
< Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/
Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/

<
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=https://ss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus=autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn" autofocus></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=https://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');
                </script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
* Connection #0 to host www.baidu.com left intact
```

# 参考链接
[使用NGINX作为HTTPS正向代理服务器](https://developer.aliyun.com/article/706196)
[HTTP 代理原理及实现（一）](https://imququ.com/post/web-proxy.html)
[搭建Nginx正向代理服务](https://www.lagou.com/lgeduarticle/72755.html)
[使用 Nginx 搭建 HTTPS 正向代理服务](https://bigzuo.github.io/2018/12/15/nginx-https-forward-proxy/)
[服务器名称指示 (Server Name Indication)](https://zh.wikipedia.org/zh-hans/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA)
[应用层协议协商 (Application-Layer Protocol Negotiation)](https://zh.wikipedia.org/zh-hans/%E5%BA%94%E7%94%A8%E5%B1%82%E5%8D%8F%E8%AE%AE%E5%8D%8F%E5%95%86)
[proxy_connect](https://tengine.taobao.org/document_cn/proxy_connect_cn.html)
[Module ngx_stream_ssl_preread_module](http://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html?spm=a2c4e.10696291.0.0.e01e19a41l7ou4)
