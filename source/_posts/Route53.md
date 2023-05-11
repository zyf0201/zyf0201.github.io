---
title: Route53私有托管域——轻松管理你私有ip和域名
date: 2020-04-24 00:28:44
tags: AWS
---

> 作为一名运维，平常的运维工作总是有很多虚机需要管理，一大堆私有IP、ELB域名、ALB域名、RDS域名、及其他endpoint总是搞得人心烦意乱.

## 杂乱的IP地址和域名
![烦人无序的私有ip](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424003112.png)

无序的IP和杂乱的域名总是让人头大，这时候就想如果像通用域名那样直接通过自定义的域名就是找到我的虚机就完美了。比如通过www.baidu.com可以直接打开百度的网页，而不是通过http://183.232.231.174/记忆这样ip地址去打开网页。


![20200424003356.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424003356.png)

这个时候就可以用aws route53自带的私有托管区域来完美解决这个问题。首先看下官方文档中关于route53私有托管区域的说明：


<! -- more -->

> 私有托管区域私有托管区域 就是一个容器，其中包含的信息说明您希望 Amazon Route 53 如何响应您使用 Amazon VPC 服务创建的一个或多个 VPC 中的某个域及其子域的 DNS 查询。
下面是私有托管区域的工作原理：您创建一个私有托管区域（如 example.com），并指定要与该托管区域关联的 VPC。您在托管区域中创建记录，用于确定 Route 53 如何响应您的 VPC 中的域及子域的 DNS 查询。例如，假设您有一个在与私有托管区域关联的 VPC 之一中的 EC2 实例上运行的数据库服务器。您创建 A 或 AAAA 记录（如 db.example.com），并指定数据库服务器的 IP 地址。有关 记录的更多信息，请参阅 使用记录。有关使用私有托管区域的 Amazon VPC 要求的信息，请参阅 Amazon VPC 用户指南 中的使用私有托管区域。当应用程序提交 db.example.com 的 DNS 查询时，Route 53 会返回相应的 IP 地址。应用程序还必须运行在与 example.com 私有托管区域关联的 VPC 之一中的 EC2 实例上。应用程序使用从 Route 53 获得的 IP 地址与数据库服务器建立连接。

接下来让我们实践一下。

## 建立私有托管域
建立私有托管域的过程非常简单：
- 首先打开Route53界面，点击**create hosted zone**：
  - step1.输入域名：test.com
  - step2.选择private hosted zone from amazon VPC
  - step3.选择需要绑定的vpc，等新建完毕后可以绑定更多的vpc，甚至可以绑定其他aws账号下的vpc
)
  - setp4.在新建的托管域下新建一条新的A记录，并指向对应vpc下某个虚机ip（10.20.119.170）
![建立私有域名](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424003636.png)

## 域名测试
接下来测试域名是否生效，在其他相同vpc下的虚机上ping www.test.com，解析出来的地址正是10.20.119.170，测试OK，可以正常ping通，其他的私有elb地址、rds域名、MQ域名等其他endpoint也可以通过私有托管域进行解析。

![域名测试](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424004500.png)
