---
title: 本地部署kubernetes集群
date: 2020-04-20 01:11:38
tags: Kubernetes
---
<!-- toc -->

> 通过在本机笔记本电脑上的virtualbox上使用kubeadm部署kubernetes集群，用于练习。


## 环境说明
操作系统：centos7.6
内核版本：3.10.0-957.el7.x86\_64 (可自行升级内核) <br>
![系统结构](https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200220165903.png)
Kubernetes版本：v1.17
---

<! -- more -->

## 虚拟机创建
- 虚拟机centos7-kub01（k8s-master）网卡配置:
   - 网卡1使用host-only网络，用于kubernetes节点之间的连接。还有用于笔记本电脑连接虚拟机。
<img src="https://github.com/michaelzhang02010479/saveimage/blob/master/img/20200419233613.png?raw=true" width=100%>

   - 网卡2使用网络地址转换（NAT），用于连接互联网，下载镜像
<img src="https://cdn.jsdelivr.net/gh/michaelzhang02010479/saveimage@master/img/20200419234213.png" width="100%">

- 虚拟机kube02的网卡配置保持一致

- 安装操作系统，本教程使用centos7.6，操作系统请自行安装

- 升级内核（为支持更高级特性，可选内核升级）
脚本参考：
```bash
#check the current version
uname -a  
#check current kernel verison
cat /etc/redhat-release 
#check the current kernel verison that has been installed
rpm -qa | grep kernel 
#安装内核源
yum install https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm 
#check infomation of kernel-lt and kernel-ml
yum info --enablerepo=elrepo-kernel kernel-lt kernel-ml 
#check the current verison of kernel 
rpm -qa | grep kernel 
#remove the specify verison of kernel-tools and kernel-tools-libs
yum remove $(rpm -qa | grep kernel | grep -v $(uname -r)) 
# LT Version（Long term support version）
# Install LT Version
yum install --enablerepo=elrepo-kernel -y kernel-lt kernel-lt-headers kernel-lt-devel kernel-lt-tools kernel-lt-tools-libs
#check the kernel you have installed
rpm -qa | grep kernel 
#rebuild configuration of grub2(Optional operation)
grub2-mkconfig -o /boot/grub2/grub.cfg 
#check the useful kernel that you hav installed
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg 
#选择需要启动的内核
grub2-set-default 0 
#重启虚拟机
reboot 
```

- 设置yum update时不升级内核
```bash
vi /etc/yum.conf
#最下面加个*
exclude=kernel*
```
---
## 安装docker
脚本参考：
```bash
#uninstall docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
 
#install docker tools
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
 
#Tsinghua University yum repository
sudo yum-config-manager \
--add-repo \
https://mydream.ink/utils/container/docker-ce.repo
 
#install docker-ce
sudo yum install docker-ce docker-ce-cli containerd.io
 
#image speed up
mkdir -p /etc/docker
cat <<EOF >>/etc/docker/daemon.json
{
    "registry-mirrors": [
        "https://dockerhub.azk8s.cn",
        "https://reg-mirror.qiniu.com"
    ]
}
EOF
 
#start docker
sudo systemctl daemon-reload
sudo systemctl start docker#stop swapswapoff -ased -i -e '/swap/d' /etc/fstabecho 'vm.swappiness=0' > /etc/sysctl.d/k8s.confsysctl -p /etc/sysctl.d/k8s.conf
```
---
## 安装kubeadm
```bash
# add kubernetes repo
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```
kubeadm安装可以参考：[Kubernetes官网](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

---
## 使用kubeadm安装kubernetes
- 首先确认已经安装的kubeadm的版本

```
[root@kub01 ~]# yum list installed|grep kube
Failed to set locale, defaulting to C
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
cri-tools.x86\_64                     1.13.0-0                       @kubernetes
kubeadm.x86\_64                       **1.17.0-0**                       @kubernetes
kubectl.x86\_64                       1.17.0-0                       @kubernetes
kubelet.x86\_64                       1.17.0-0                       @kubernetes
kubernetes-cni.x86\_64                0.7.5-0                        @kubernetes
```
- **提前下载需要使用到镜像，这个非常重要**
因为国内网络会提示k8s.gcr.io连接超时，Kubernetes v1.17版本需要用到的镜像如下(master节点)：
```yaml
k8s.gcr.io/kube-apiserver:v1.17.0
k8s.gcr.io/kube-controller-manager:v1.17.0
k8s.gcr.io/kube-scheduler:v1.17.0
k8s.gcr.io/kube-proxy:v1.17.0
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.5
```
通过docker pull命令先在国内源将镜像下载完毕，然后通过docker tag命令重新设置tag（master节点）
```bash
docker pull gcr.azk8s.cn/google\_containers/kube-proxy:v1.17.0  
docker pull gcr.azk8s.cn/google\_containers/kube-apiserver:v1.17.0  
docker pull gcr.azk8s.cn/google\_containers/kube-controller-manager:v1.17.0  
docker pull gcr.azk8s.cn/google\_containers/kube-scheduler:v1.17.0  
docker pull gcr.azk8s.cn/google\_containers/etcd:3.4.3-0  
docker pull gcr.azk8s.cn/google\_containers/coredns:1.6.5  
docker pull gcr.azk8s.cn/google\_containers/pause:3.1
```

- docker tag重新设置镜像名称(master节点)
```bash
docker tag gcr.azk8s.cn/google\_containers/kube-proxy:v1.17.0 k8s.gcr.io/kube-proxy:v1.17.0 
docker tag gcr.azk8s.cn/google\_containers/kube-apiserver:v1.17.0 k8s.gcr.io/kube-apiserver:v1.17.0 
docker tag gcr.azk8s.cn/google\_containers/kube-controller-manager:v1.17.0 k8s.gcr.io/kube-controller-manager:v1.17.0 
docker tag gcr.azk8s.cn/google\_containers/kube-scheduler:v1.17.0 k8s.gcr.io/kube-scheduler:v1.17.0 
docker tag gcr.azk8s.cn/google\_containers/etcd:3.4.3\-0 k8s.gcr.io/etcd:3.4.3\-0 docker tag gcr.azk8s.cn/google\_containers/coredns:1.6.5 k8s.gcr.io/coredns:1.6.5 docker tag gcr.azk8s.cn/google\_containers/pause:3.1 k8s.gcr.io/pause:3.1
```

- **子节点**只需要下载其中两个镜像即可
```
docker pull gcr.azk8s.cn/google\_containers/kube-proxy:v1.17.0  
docker pull gcr.azk8s.cn/google\_containers/pause:3.1
docker tag gcr.azk8s.cn/google\_containers/kube-proxy:v1.17.0 k8s.gcr.io/kube-proxy:v1.17.0 docker tag gcr.azk8s.cn/google\_containers/pause:3.1 k8s.gcr.io/pause:3.1
```
注意： 如果没有对应版本的镜像，就只能自己去搜寻了

- Kubernetes安装
镜像搞定后，接下来安装kubernetes，通过kubeadm init安装,kubeapi server地址设置为master的IP地址，pod网络设置为10.244.0.0/16
```
kubeadm init --apiserver-advertise-address 192.168.56.107 --pod-network-cidr=10.244.0.0/16
```
安装日志信息（部分信息省略）：
```
[root@kub01 ~]# kubeadm init --apiserver-advertise-address 192.168.56.107 --pod-network-cidr=10.244.0.0/16
W0119 01:08:20.120396   20293 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
W0119 01:08:20.120526   20293 version.go:102] falling back to the local client version: v1.17.0
...
kubeadm join 192.168.56.107:6443 --token 40yu1v.jb0rfmijbjwwp1mk \
    --discovery-token-ca-cert-hash sha256:be3edcdd89b0a742a216a2a041633e57cbd4223e100edacc504785249499530c
```

- master节点配置kubectl
```bash
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
如果是root用户，可以运行一下命令配置kubectl
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

- 添加子节点到Kubernetes集群
子节点可以通过以下命令加入到kubernetes集群（安装日志信息最后会打印）：
```
kubeadm join 192.168.56.107:6443 --token 40yu1v.jb0rfmijbjwwp1mk \\ \--discovery-token-ca-cert-hash sha256:be3edcdd89b0a742a216a2a041633e57cbd4223e100edacc504785249499530c
```
  
- 安装网络插件
查看所有的pods，可以发现coredns这个pod一直处于pending
```
[root@kub01 ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-k467p        0/1     Pending   0          6m55s
kube-system   coredns-6955765f44-qrpv4        0/1     Pending   0          6m55s
kube-system   etcd-kub01                      1/1     Running   0          7m9s
kube-system   kube-apiserver-kub01            1/1     Running   0          7m9s
kube-system   kube-controller-manager-kub01   1/1     Running   0          7m9s
kube-system   kube-proxy-mmv9x                1/1     Running   0          6m54s
kube-system   kube-scheduler-kub01            1/1     Running   0          7m9s
```

查看所有的节点，都处于notreayd状态，这是因为没有安装网络插件，可选calico、flannel或者其他插件，
```
[root@kub01 ~]# kubectl get nodes
NAME    STATUS     ROLES    AGE   VERSION
kub01   NotReady   master   12m   v1.17.0
kub02   NotReady   <none>   41s   v1.17.0
```

此处我们在这里使用flannel网络插件（不同的网络插件此处不详细解释），但是又因为国内网络网络下载镜像可能超时(太苦逼了)，我们还是通过docker tag命令重新打tag的方式处理（master和node节点都需要执行）
```
docker pull quay-mirror.qiniu.com/coreos/flannel:v0.11.0-amd64
docker tag quay-mirror.qiniu.com/coreos/flannel:v0.11.0-amd64 quay.io/coreos/flannel:v0.11.0-amd64
```

在master节点通过kubctl安装flannel，首先下载flannel的安装配置文件
```
wget https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```
修改kube-flannel.yml，添加启动参数，**因为我们是双网卡的虚拟机，需要指定对应的网卡**，这一步非常重要，否则可能导致后续的dns解析异常问题（都是血和泪的教训），首先确认host-only对应的网卡名称（enp0s3），网卡名称可以通过ifconfig命令查看
```
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.107  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::3270:547f:5509:7c24  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:c6:e6:f1  txqueuelen 1000  (Ethernet)
        RX packets 41219  bytes 3758617 (3.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 42864  bytes 21514707 (20.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.3.15  netmask 255.255.255.0  broadcast 10.0.3.255
        inet6 fe80::7653:3467:95b2:bf4b  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:3e:a2:7a  txqueuelen 1000  (Ethernet)
        RX packets 17180  bytes 20648597 (19.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3664  bytes 279458 (272.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

kube-flannel.yaml文件修改对应的启动参数位置：
```
 containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - \--iface=enp0s3
```

安装flannel：
```
kubectl apply -f kube-flannel.yml
```

运行完毕后查看所有的pods和nodes状态
```
[root@kub01 ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-k467p        1/1     Running   2          57m
kube-system   coredns-6955765f44-qrpv4        1/1     Running   2          57m
kube-system   etcd-kub01                      1/1     Running   2          57m
kube-system   kube-apiserver-kub01            1/1     Running   3          57m
kube-system   kube-controller-manager-kub01   1/1     Running   2          57m
kube-system   kube-flannel-ds-amd64-chb5z     1/1     Running   3          44m
kube-system   kube-flannel-ds-amd64-dzwpr     1/1     Running   0          44m
kube-system   kube-proxy-46jzs                1/1     Running   0          45m
kube-system   kube-proxy-mmv9x                1/1     Running   3          57m
kube-system   kube-scheduler-kub01            1/1     Running   2          57m
```
```
[root@kub01 ~]# kubectl get nodes
NAME    STATUS   ROLES    AGE    VERSION
kub01   Ready    master   3h2m   v1.17.0
kub02   Ready    <none>   170m   v1.17.0
```

终于安装完毕,使用kubeadm安装kubernetes可以参考：[Kubernetes官方文档](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/)

 ## 部署ingress
Ingress主要实现集群内所有服务的入口，通过一系列规则集合来允许外部的访问。
```bash
git clone https://github.com/nginxinc/kubernetes-ingress/
cd kubernetes-ingress/deployments
git checkout v1.6.3
# Create a namespace and a service account for the Ingress controller
kubectl apply -f common/ns-and-sa.yaml
# Create a cluster role and cluster role binding for the service account
kubectl apply -f rbac/rbac.yaml
# Create a config map for customizing NGINX configuration
kubectl apply -f common/default-server-secret.yaml
# Create custom resource definitions for VirtualServer and VirtualServerRoute resources
kubectl apply -f common/custom-resource-definitions.yaml
# Use a DaemonSet Run the Ingress Controller
kubectl apply -f daemon-set/nginx-ingress.yaml
# Check that the Ingress Controller is Running
kubectl get pods --namespace=nginx-ingress
```
参考：[nginx-ingress-controller](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/)

## 问题排错
- 安装过程中coredns 一直处于ContainerCreating状态，因为我之前重复添加了节点并修改了主机名，需要reset，并删除对应的cni配置文件，以下是相关的命令：
```bash
kubeadm reset
rm -rf /var/lib/cni/ /var/lib/kubelet/* /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig docker0 down
ip link delete cni0
ip link delete flannel.1
reboot
```
然后再重新join
```
kubeadm join xxx
```
