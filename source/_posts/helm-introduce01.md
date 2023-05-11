---
title: Helm介绍01-初识Helm
date: 2021-02-06 19:59:51
tags: helm
---


## Helm介绍

### Helm概念
> Helm 是 Deis 开发的一个用于 Kubernetes 应用的包管理工具，主要用来管理 Charts。有点类似于 Ubuntu 中的 APT 或 CentOS 中的 YUM。Helm Chart 是用来封装 Kubernetes 原生应用程序的一系列 YAML 文件。可以在你部署应用的时候自定义应用程序的一些 Metadata，以便于应用程序的分发。对于应用发布者而言，可以通过 Helm 打包应用、管理应用依赖关系、管理应用版本并发布应用到软件仓库。对于使用者而言，使用 Helm 后不用需要编写复杂的应用部署文件，可以以简单的方式在 Kubernetes 上查找、安装、升级、回滚、卸载应用程序。

### Helm架构

{% asset_img 1.png helm架构 %}

Helm由两部分组成：**Helm Client** 和 **Tiller Server**，Client负责管理 chart，而Server端负责管理 release。

- Helm Client 是最终用户的命令行客户端。客户端负责以下部分：

    - 本地 chart 开发
    - 管理存储库
    - 与 Tiller 服务交互
    - 发送要安装的 chart
    - 查询有关发布的信息
    - 请求升级或卸载现有 release

> Chart：一个 Helm 包，其中包含了运行一个应用所需要的工具、资源定义等，还可能包含 Kubernetes 集群中的服务定义，类似 APT 的 dpkg 或者 Yum 的 RPM 文件，

- Tiller Server 是一个集群内服务，与 Helm 客户端进行交互，并与 Kubernetes API 服务进行交互。服务负责以下内容：

  - 监听来自 Helm 客户端的传入请求
  - 结合 chart 和配置来构建发布
  - 将 chart 安装到 Kubernetes 中，然后跟踪后续 release
  - 通过与 Kubernetes 交互来升级和卸载 chart
  
> Release: 在 Kubernetes 集群上运行的 Chart 的一个实例。在同一个集群上，一个 Chart 可以安装很多次。每次安装都会创建一个新的 release。

*Helm 整个系统的主要任务就是，在仓库中查找需要的 Chart，然后把 Chart 以 Release 的形式安装到 Kubernetes 之中*

Helm 客户端使用 Go 编程语言编写，并使用 gRPC 协议套件与 Tiller 服务进行交互。

Tiller 服务也用 Go 编写。它提供了一个与客户端连接的 gRPC 服务，它使用 Kubernetes 客户端库与 Kubernetes 进行通信。目前，该库使用 REST + JSON。

Tiller 服务将信息存储在位于 Kubernetes 内的 ConfigMaps 中。它不需要自己的数据库。

## Helm安装

### 二进制版本安装

- 下载对应[需要的版本](https://github.com/helm/helm/releases)
- 解压对应的tar包
- 在解压目中找到helm程序，移动到需要的目录中

### 脚本安装

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### 通过包管理器安装

- macos安装

brew install helm

- Debian/Ubuntu

```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

- [其他安装方式](https://helm.sh/zh/docs/intro/install/)

## Helm使用

使用Helm的先决条件：
- 使用Helm，需要一个Kubernetes集群(最简单的是自己装一个minikube，或者自己通过虚拟机搭建一个单节点的kubernetes,可以使用[kubez-ansible](https://github.com/caoyingjunz/kubez-ansible)快速搭建)，也应该有一个本地的 kubectl
- 安装和配置Helm

### Helm初始化

安装Helm后，可以添加一个chart仓库,一个常见的选择是添加Helm的官方仓库:
```
helm repo add stable https://charts.helm.sh/stable
```

可以看到可以被您安装的charts列表：
{% asset_img 2.png 查看安装的chart %}

### 使用Helm安装一个release

```bash

# 在chart仓库中搜索mysql
helm search repo mysql

# 使用helm安装mysql
helm install stable/mysql --generate-name
```

{% asset_img 3.png 使用helm安装mysql %}

通过helm ls可以查看已经创建的release

{% asset_img 5.png 使用helm安装mysql %}

> 所以，通过Helm就可以快速部署一个自己所需要的组件应用，节省大部分时间。可以为自己的应用编写chart，并快速发布。



## 参考资料
[快速入门指南](https://helm.sh/zh/docs/intro/quickstart/)
[使用Helm](https://helm.sh/zh/docs/intro/using_helm/)
[Kubernetes Helm 架构](https://whmzsu.github.io/helm-doc-zh-cn/architecture-zh_cn.html)