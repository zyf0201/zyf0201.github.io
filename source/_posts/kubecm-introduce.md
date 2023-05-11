---
title: 使用kubecm管理多个Kubernetes集群
date: 2021-03-16 19:23:07
tags: kubecm
---


> 最近遇到需要切换多个Kubernetes集群的场景，发现了kubecm这个工具，可以轻松管理你的Kubeconfig文件，github地址：https://github.com/sunny0826/kubecm


## Kubecm安装
- Macos安装
```bash
brew install kubecm
```

- 二进制下载安装
```bash
# linux x86_64
curl -Lo kubecm.tar.gz https://github.com/sunny0826/kubecm/releases/download/v${VERSION}/kubecm_${VERSION}_Linux_x86_64.tar.gz
# macos
curl -Lo kubecm.tar.gz https://github.com/sunny0826/kubecm/releases/download/v${VERSION}/kubecm_${VERSION}_Darwin_x86_64.tar.gz
# windows
curl -Lo kubecm.tar.gz https://github.com/sunny0826/kubecm/releases/download/v${VERSION}/kubecm_${VERSION}_Windows_x86_64.tar.gz

# linux & macos
tar -zxvf kubecm.tar.gz kubecm
cd kubecm
sudo mv kubecm /usr/local/bin/

# windows
# Unzip kubecm.tar.gz
# Add the binary in to your $PATH
```

## Kubecm 使用

- Kubecm用法
```text
Usage:
  kubecm [flags]
  kubecm [command]

Available Commands:
    add         Merge configuration file with $HOME/.kube/config
    alias       Generate alias for all contexts
    clear       Clear lapsed context, cluster and user
    completion  Generates bash/zsh completion scripts
    create      Create new KubeConfig(experiment)
    delete      Delete the specified context from the kubeconfig
    help        Help about any command
    list        List kubeconfig
    merge       Merge the kubeconfig files in the specified directory
    namespace   Switch or change namespace interactively
    rename      Rename the contexts of kubeconfig
    switch      Switch Kube Context interactively
    version     Print version info


Flags:
      --config string   path of kubeconfig (default "$HOME/.kube/config")
  -h, --help   help for kubecm

Use "kubecm [command] --help" for more information about a command.
```
  
- 添加集群连接信息
示例：
```bash
kubecm add -f test.yaml  ## test.yaml为Kubernets集群的连接信息
```
输入上述命令后会提示是否覆盖$HOME/.kube/config文件，如果选择flase，则会在本地生成./config.yaml

- 罗列所有集群信息

```bash
kubecm list
```
{% asset_img 1.png kubecm-list %}

- 切换集群
```
kubecm switch
```
根据箭头切换集群信息

- 删除集群
```
kubecm delete
```
根据箭头删除对应的集群



