---
title: 搭建github博客
date: 2020-04-17 01:11:55
tags: github
---
<!-- toc -->

一直想把所有博客迁移到github上，今天正式开始，不定时更新~~


## 安装nodejs
windows上安装nodejs

## 安装Hexo
```bash
npm i hexo-cli -g # 安装Hexo
hexo -v #验证是否成功
hexo init #切换到对应的目录，然后初始化文件夹
npm install #安装必要组件
hexo g #生产静态网页
hexo s #打开本地服务器，然后浏览器打开localhost:4000预览博客
```

## 连接github(前提你已经创建了github pages)
首先登录自己的github，创建GitHub Pages
```bash
git config --global user.name "xxx" #配置github账户
git config --global user.email "xxx"
#复制公钥到github
```

## 修改hexo的配置文件
修改hexo目录下的配置文件：_config.yml
```yaml
deploy:
  type: git
  repository: https://github.com/xxx/xxx.github.io
  branch: master
```

## 写文章、发布文章
```bash
npm i hexo-deployer-git #安装扩展
hexo new post "name of you title" #新建文章并编辑，文件位于\source\_posts
hexo g #生产静态网页
hexo s #打开本地服务器并预览静态网页
hexo d #上传到github
```
至此简单的博客发布已经OK~~~ 挺easy!!

## 修改标题
修改_config.yml是hexo的全局配置
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: 博客的标题
subtitle: 博客的字标题
description: 文字描述
author: 
language: 
timezone:
```

## 博客域名配置
首先在购买的域名解析平台(我用的是阿里云云解析)添加自己的CNAME记录
![20200420020043.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200420020043.png)
打开你的github博客项目，点击settings，拉到下面Custom domain处，填上你自己的域名
![20200420020348.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200420020348.png)
域名HTTPS的配置可以参考下别人的文章:
[ssl配置](https://tzhou2018.github.io/2018/04/%E4%B8%BAGitHub-Pages%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9F%9F%E5%90%8D%E5%B9%B6%E6%B7%BB%E5%8A%A0SSL-%E5%BC%80%E5%90%AFHTTPS%E5%BC%BA%E5%88%B6/)

## 配置静态资源文件夹
- 修改hexo的配置文件_config.yml
```
post_asset_folder: true
```
- 配置markdown嵌入图片
[exo-renderer-marked](https://github.com/hexojs/hexo-renderer-marked) 3.1.0 引入了一个新选项，允许您在 Markdown 中嵌入图像，而无需使用asset_img 标签插件。

启用方式：
修改配置文件_config.yml
```yml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

- 创建一个新的测试文章
```
hexo new post "test static files"
```
可以看到一个跟文章同名的目录，可以将图片等文件放置在这里~~
![20200421224154.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200421224154.png)

- 可以使用以下这些标签引用自己的资源
```html
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

- 引用图片
把图片test.png放到目录下，新的博客中使用下面的方式引用图片
```html
{% asset_img test.png this is an example %}
```
查看效果:
![20200421224843.png](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200421224843.png)

## 修改首页文章显示数量

> 每次首页显示太多，加载会非常慢，可以通过修改hexo的全局配置设置每页显示的数量

修改hexo主目录配置文件./_config.yml,修改index_gererator的per_page参数

```
index_generator:
  path: ''
  per_page: 10
  order_by: -date
```

- 首页不要展开全文

在非首行处添加 <!-- more -->

## 更换主题
如果对于默认的首页不感冒，那么就可以更换博客的主题，以目前比较欢迎的主题[next](https://github.com/iissnan/hexo-theme-next)为例

- 切换到hexo的目录，安装next
```bash
git clone --branch v5.1.2 https://github.com/iissnan/hexo-theme-next themes/next
```
- 修改hexo的配置文件_config.yml
```
theme: next
```
- 重新generate，并查看效果


