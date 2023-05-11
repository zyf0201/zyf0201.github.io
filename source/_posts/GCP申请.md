---
title: Google Cloud免费实例申请
date: 2020-04-24 01:07:04
tags: GCP
---

> 之前因为更换工作导致原来的EC2主机无法使用，因原来的的博客都搭在公司的EC2实例上。这次不得不迁移到新的虚机上，偶然听说google cloud新用户可以免费送一年，就打算把博客什么的迁移到新的google cloud的VM实例。

## GCP账号申请
- 申请google账号
登陆Google cloud的console地址（[https://console.cloud.google.com](https://console.cloud.google.com/)）
**登陆后填写相关信息后可以免费获得300美刀赠金：**
（以下图片来自于网络）
同意条款后，填写个人资料，如实填写即可。
![20200424011350.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424011350.png)
![20200424011518.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424011518.png)

<!-- more -->

- 填写信用卡信息
填写信用卡卡号，需要注意，不支持银联，需要提前准备好一张多币卡。填写OK后会有一笔扣款交易失败，所以不用担心乱扣费。
![20200424011633.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424011633.png)

>特别说明：正常使用300美金可以用一年，但是我个人去申请的时候发现多了新的条款，流量如果是流出到非北美地区可能是需要额外扣费，具体以实际使用为准，但是不影响，如果赠金用完google cloud也不会随意扣费。

## 登录GCP，新建vm虚机
- 登陆console,点击compute engine的vm实例：
![20200424011824.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424011824.png)
- 使用vm实例之前针对默认的新项目需要首先启动compute engine，需要等一会
![20200424011859.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424011859.png)
- 启用完毕后，开始创建vm虚机：
![20200424011934.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424011934.png)
- 购买虚拟机
通过对比发现台湾区域的vm实例比较便宜（28.5美刀每个月），镜像根据个人习惯使用了centos7，开放http和https端口。
![20200424012008.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424012008.png)
- 连接虚拟机实例
等待新建完成后，就可以用console通过ssh连接到vm实例，把自己的公钥导入到虚机接下来直接用xshell登录即可。
![20200424012129.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200424012129.png)

总体来说google cloud的console界面非常友好，有了自己新的虚机，接下来把博客迁移到新的虚机就可以啦！！！

- 20200203更新：
前几天google的虚拟机已经到期了，果然没到一年就余额就用完了。应该是流量额外扣费，建议配合CDN使用，现在这情况如果继续续费就太贵了，所以还是乖乖用vps，5美元一个月。
