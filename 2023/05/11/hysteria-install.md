---
title: Hysteria2介绍及安装
date: 2023-10-17 02:06:12
tags: vpn
---

## Hysteria介绍

[Hysteria](https://v2.hysteria.network/zh/) 是一个强大、快速、抗审查的代理工具。

今天测试了一下，跟传统的协议相比，hysteria协议的速度很快，打开youtube 4K毫无压力，各位童靴可以试试。在这里简单的说一下centos7系统的安装步骤，建议直接查看[文档](https://v2.hysteria.network/zh/docs/getting-started/Installation/)操作。

## 安装Hysteria

一键安装脚本：
```bash
bash <(curl -fsSL https://get.hy2.sh/)
```

前提条件：
- 一个带公网IP的VPS，your.IP
- 一个指向该公网IP的域名，your.domain.net

服务端配置：
- 首先编辑配置文件,以下配置可以使用 ACME 自动获取域名的 TLS 证书，详细配置信息可以参考[这里](https://v2.hysteria.network/zh/docs/advanced/Full-Server-Config/)：
```yaml
# listen: :443 
acme:
  domains:
    - your.domain.net #改为你自己的域名
  email: your@email.com 

auth:
  type: password
  password: Se7RAuFZ8Lzg # 密码可以自定义

masquerade: 
  type: proxy
  proxy:
    url: https://news.ycombinator.com/ 
    rewriteHost: true
```

- 编辑好配置文件放置到这个目录：/etc/hysteria/config.yaml

启动服务端的Hysteria服务(启动前建议确认防火墙配置):

```bash
systemctl start hysteria-server.service 
```

如果启动失败，可以用下面的命令行启动并查看日志：
```bash
/usr/local/bin/hysteria server --config /etc/hysteria/config.yaml
```

## 客户端配置
这边介绍v2rayN和shadowRotcket的客户端配置

- v2rayN(windows)
新增配置文件，并保存为config.yaml文件：
```yaml
server: xx.xx.xx.xx:443 #改为你VM的IP

auth: xxx #改成你对应的password

bandwidth: 
  up: 20 mbps
  down: 100 mbps

socks5:
  listen: 127.0.0.1:10808 #对应的代理端口

http:
  listen: 127.0.0.1:10809 
tls:
  insecure: true
  sni: xxx.xxx.com #对应的域名地址

quic:
  initStreamReceiveWindow: 16777216
  maxStreamReceiveWindow: 16777216
  initConnReceiveWindow: 33554432
  maxConnReceiveWindow: 33554432

fastOpen: true
```

更新hysteria核心文件：
根据你的操作系统下载[hesteria核心文件](https://github.com/apernet/hysteria/releases),并保存到v2rayN-Home目录\bin\hysteria\
{% asset_img core-file.png core file %}


新增配置：
路径：服务器-添加自定义配置服务器
{% asset_img v2rayN-hesteria-config.png v2rayN-hesteria-config %}

- ShadowSocket配置
需要更新为支持hesteria2协议的版本
{% asset_img v2rayN-hesteria-config.png v2rayN-hesteria-config %}

- 如需完整的客户端配置文件，请参考：
[完整客户端配置](https://v2.hysteria.network/zh/docs/advanced/Full-Client-Config/)