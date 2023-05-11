---
title: 推荐一个一键安装Kubernetes集群的工具
date: 2021-01-22 00:40:45
tags: 
    - Kubernetes 
    - Ansible
---

推荐一个工具，可以用于一键安装Kubernetes及相关生态的工具，可以参考项目[源码地址](https://github.com/caoyingjunz/kubez-ansible)。


如果要在单机安装Kubernetes集群，可以直接运行all-in-one脚本

```bash
curl https://raw.githubusercontent.com/yingjuncao/kubez-ansible/master/tools/all_in_one.sh | bash

```

亲测单机安装有效
{% asset_img 1.png P1 %}

多节点安装可以支持，可以参考[kubez-ansibel安装多节点 的使用方法](https://github.com/caoyingjunz/kubez-ansible/blob/master/doc/source/install/multinode.md)。


后续将会更新新的工具！！请持续关注～～
