---
layout: post
title: Kubernetes 集群全攻略
date: 2019-06-06 19:56:00
categories: 
- Infrastructure
tags:
- Kubernetes
- Docker
---

## 基本环境

操作系统使用 Ubuntu 19.04，标准安装。

K8s 集群起步建议至少一个管理节点（master），至少两个计算节点。

### 安装 Docker

执行 `sudo apt install docker.io` 安装 docker。

然后赋予当前用户执行 docker 命令的权限：

```bash
sudo usermod -aG docker `whoami`
sudo systemctl enable docker
sudo systemctl start docker
sudo chmod a+rw /var/run/docker.sock
docker info
```

重启一下。

### 从 apt 源安装控制平面

在国内，用不了 Google 源，建议使用阿里云或者科大的源。但是要注意，科大源目前没有 gpg 秘钥，这个秘钥只能用阿里云的。下面的命令混用了这两个源。

```bash
sudo su -
apt update && apt install -y apt-transport-https curl
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://mirrors.ustc.edu.cn/kubernetes/apt/ kubernetes-xenial main
EOF
apt update && apt install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

### 修改参数

```bash
sudo su -
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
systemctl restart docker
```

### 关闭交换空间

```bash
sudo swapoff -a
```

修改 `/etc/fstab`，将其中 swap 一行注释掉。

创建 `/etc/sysctl.d/k8s.conf`，然后新增：

```conf
vm.swappiness = 0
```

可以通过 `sudo sysctl -w vm.swappiness=0` 命令即时生效。

## 利用 kubeadm 初始化集群

在主节点上执行：

```bash
sudo kubeadm init
```

这时可以查看提示信息，常见的问题包括：

* 没有关闭交换空间。参照上一节内容操作。
* 没有使能 docker 服务。执行 `sudo systemctl enable docker`。
* cgroup 驱动不正确。参照上一节内容操作。
* CPU 只有单核。这个往往是虚拟机里常见的，增加 vCPU 核数；
* kernel 5.0 版本不支持。说明你可能用的源太老，kubeadm 不是最新。

除此之外，就是关键的一个问题，拉取镜像失败。

### 拉取镜像

执行 `sudo kubeadm config images list` ，可以得到所需的镜像列表，例如：

```txt
k8s.gcr.io/kube-apiserver:v1.14.2
k8s.gcr.io/kube-controller-manager:v1.14.2
k8s.gcr.io/kube-scheduler:v1.14.2
k8s.gcr.io/kube-proxy:v1.14.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
```

应对方法：

1. 在外网的 vps 上抓取这些镜像，然后 save 出来，下载到本地，再 load。这样 tag 都不用改，比较方便；
1. 使用国内的镜像源，比如阿里云，抓取镜像，但是需要 retag。

无论如何，最终保证每个节点上都有这些对应 tag 的镜像即可。

### 主节点初始化

排除以上故障之后，执行下面命令正式初始化集群：

```bash
sudo kubeadm init --apiserver-advertise-address 10.0.0.220 --pod-network-cidr 10.244.0.0/16
```

注意，第一个地址是根据自己实际情况填写，第二个地址用于 flannel 网络，必须是这个网段，不可以更改。可以得到下面的输出：

```txt
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.220:6443 --token swec3l.velq2geig5p126fi \
    --discovery-token-ca-cert-hash sha256:63b8a602e06e210e2e3918171e1a325a892a5d46d873191a60d4c5f6370bcadb
```

那么首先按照提示执行下列命令，获取普通用户执行权限。

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

特别地，如果发现上面哪一步搞坏了，想重新开始，就执行：

```bash
sudo kubeadm reset
```

### 创建 flannel 网络

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

当然也可以事先下载好这个文件，然后在本地执行。

### 添加 node

在所有的 node 节点上，都要执行第一章中的内容，以及准备好 docker 镜像。

之后，再执行前面获得的加入命令：

```bash
sudo kubeadm join 10.0.0.220:6443 --token swec3l.velq2geig5p126fi \
    --discovery-token-ca-cert-hash sha256:63b8a602e06e210e2e3918171e1a325a892a5d46d873191a60d4c5f6370bcadb
```

回到主节点上，通过 `kubectl get nodes` 可以查看是否加入成功：

```txt
NAME         STATUS   ROLES    AGE    VERSION
k8s-master   Ready    master   22m    v1.14.2
k8s-node1    Ready    <none>   12m    v1.14.2
k8s-node2    Ready    <none>   6m7s   v1.14.2
k8s-node3    Ready    <none>   3m8s   v1.14.2
```

如果有节点是 NotReady 的状态，那肯定是上面某一步出错了，reset 重来吧。

## dashboard

安装 dashboard 的命令为：

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

然后可以用 `kubectl get pod -A` 命令查看是否启动成功。当然了，如果在国内，肯定是不会成功的，你会看到提示拉取镜像失败。还需要把 `k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1` 镜像给弄到节点上来。

正常应该是这样的：

```txt
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-fb8b8dccf-9v7nr                 1/1     Running   0          76m
kube-system   coredns-fb8b8dccf-cs64z                 1/1     Running   0          76m
kube-system   etcd-k8s-master                         1/1     Running   0          75m
kube-system   kube-apiserver-k8s-master               1/1     Running   0          75m
kube-system   kube-controller-manager-k8s-master      1/1     Running   0          75m
kube-system   kube-flannel-ds-amd64-hnrwm             1/1     Running   0          60m
kube-system   kube-flannel-ds-amd64-s9rx7             1/1     Running   1          57m
kube-system   kube-flannel-ds-amd64-x66n8             1/1     Running   0          75m
kube-system   kube-flannel-ds-amd64-zj8vx             1/1     Running   0          67m
kube-system   kube-proxy-94gns                        1/1     Running   0          60m
kube-system   kube-proxy-gmrmg                        1/1     Running   1          57m
kube-system   kube-proxy-p7wdr                        1/1     Running   0          67m
kube-system   kube-proxy-sqxqv                        1/1     Running   0          76m
kube-system   kube-scheduler-k8s-master               1/1     Running   0          75m
kube-system   kubernetes-dashboard-5f7b999d65-wkhtm   1/1     Running   0          44m
```

### 登录 dashboard

正常情况下你是没办法直接访问到 k8s 集群中的容器服务的，目前我们也没有部署 ingress，所以只能用 workaround。以下两种方法，官方建议 k8s proxy，但这样做的问题在于 ssl 访问会失败。还有一种办法就是修改 NodePort。

#### k8s proxy

在管理节点执行：

```bash
kubectl proxy --accept-hosts='^*$' --address='0.0.0.0'
```

然后访问 <http://[MASTER-IP]:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy>。

#### NodePort

首先执行：

```bash
kubectl describe svc -n kube-system kubernetes-dashboard
```

可以看到 Type 值是 ClusterIP。我们下面将其改成 NodePort：

```bash
kubectl patch svc -n kube-system kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}'
```

再执行之前的命令，可以看到 Type 已经被改为了 NodePort。下面我们找一下容器暴露的端口。执行：

```bash
kubectl get svc -n kube-system
```

结果可能如下：

```txt
NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP   16h
kubernetes-dashboard   NodePort    10.100.19.3   <none>        443:30134/TCP            15h
```

说明它在主机上对应的端口是 30134。再继续寻找它所在的节点主机：

```bash
kubectl get pods -A -o wide
```

对应输出可能如下：

```txt
kube-system   kubernetes-dashboard-5f7b999d65-wkhtm   1/1     Running   0          15h   10.244.1.2   k8s-node1    <none>           <none>
```

这就说明位于 node1 上。访问 <https://[node1-ip]:30134> 可打开登录页面。登录时会提示从两种方式中选择一种。我们采取令牌的方法。

### 创建登录 token

这里用到的是 k8s 的 RBAC 机制。

首先创建服务用户：

```bash
kubectl create serviceaccount admin-user -n kube-system
```

然后创建集群角色绑定：

```bash
kubectl create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=kube-system:admin-user
```

最后获取 tocken：

```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

显示类似于：

```txt
Name:         admin-user-token-vwdvn
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: d1c72619-88e1-11e9-8920-000c29dc0e56

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXZ3ZHZuIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJkMWM3MjYxOS04OGUxLTExZTktODkyMC0wMDBjMjlkYzBlNTYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.DeT18f_NSEpXqqCeWP0SB_D8pomLJrDT9uDm3v3re5AEBvFO8f3gDrFDVdFcFBB23izBhRvISjTUGKPj-l19oWoDCVP3-vQ6WBnTCu81bFoBvqsS4KHaCyOWBexWe5trYrtV1hK8Rl_z3EY2_WORg95glJVa2I92-kUdhPGTqwsW6WznjSJbQkPC-REPjbqRrnSJ0kA8dl7l57drKQhkhHIVEQjcwuhxvegxocplEFmdcfsJBISIcj0acUiHl9zPYTMSUmoRLwDuRfKOCXEbLDkPTYg0ic6enEQ67OyVPNdHbZ3QUlteIv-OiCqaD_u97X2bStkgqTgmKWW9SQRMxA
```

后面这一长串就是登录 token，在登录页面填入即可登录。

## 集群升级

本文只针对 v1.14.x 小版本升级。

### 升级控制平面

首先执行 `sudo apt update && apt-cache policy kubeadm` 结果可能如下：

```txt
kubeadm:
  Installed: 1.14.2-00
  Candidate: 1.14.3-00
```

说明当前安装的是 1.14.2 版，但是有新的 1.14.3 版。另外，我们之前安装 kubeadm 的时候做了 `apt-mark hold`，即防止 `apt upgrade` 命令平时升级这些包。所以现在升级时要临时取消 hold。注意替换命令中的版本号为最新版本。

```bash
sudo su -
apt-mark unhold kubeadm && \
apt update && apt install -y kubeadm=1.14.3-00 && \
apt-mark hold kubeadm
exit
```

确定 kubeadm 版本已升级：

```bash
kubeadm version
```

### 升级主节点

执行 `sudo kubeadm upgrade plan`，结果如下：

```txt
COMPONENT   CURRENT       AVAILABLE
Kubelet     4 x v1.14.2   v1.14.3

Upgrade to the latest version in the v1.14 series:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.14.2   v1.14.3
Controller Manager   v1.14.2   v1.14.3
Scheduler            v1.14.2   v1.14.3
Kube Proxy           v1.14.2   v1.14.3
CoreDNS              1.3.1     1.3.1
Etcd                 3.3.10    3.3.10

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.14.3
```

这里可以看到几个关键的镜像也有更新。在国内或者内网，还是要进行一次搬运，将新版本镜像导入到每一个节点上。

确定每个节点上都有这些镜像的对应版本后，执行：

```bash
sudo kubeadm upgrade apply v1.14.3
```

回答 y，然后耐心等待，在中间某些过程会停留较长时间，只要确定镜像都准备好，肯定能过的。（实在出错，就重启再来）

看到下面的结果，就是成功了：

```txt
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.14.3". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

然后执行下面的命令，升级 kubectl 和 kubelet：

```bash
sudo su -
apt-mark unhold kubelet kubectl && \
apt update && apt install -y kubelet=1.14.3-00 kubectl=1.14.3-00 && \
apt-mark hold kubelet kubectl
exit

sudo systemctl restart kubelet
```

### 升级其它控制平面节点

在单 master 下无需执行，参照上一节内容，把 `sudo kubeadm upgrade apply` 命令换成 `sudo kubeadm upgrade node experimental-control-plane` 即可。另外，`sudo kubeadm upgrade plan` 也不需要执行。

### 升级工作节点

就是那几个非 master 的节点。注意要逐个升级。

首先将 kubeadm 进行升级。

```bash
sudo su -
apt-mark unhold kubeadm && \
apt update && apt install -y kubeadm=1.14.3-00 && \
apt-mark hold kubeadm
exit
```

在**主节点**上，执行下面命令，将工作节点 cordon：

```bash
kubectl drain k8s-node1 --ignore-daemonsets --delete-local-data
```

注意，这个 delete-local-data 的参数是会删除所有本地卷的，这是因为之前部署了 dashboard。正式生产环境要慎重。然后回到工作节点升级 kubelet 配置：

```bash
sudo kubeadm upgrade node config --kubelet-version v1.14.3
```

最后升级 kubelet 和 kubectl：

```bash
sudo su -
apt-mark unhold kubelet kubectl && \
apt update && apt install -y kubelet=1.14.3-00 kubectl=1.14.3-00 && \
apt-mark hold kubelet kubectl
exit

sudo systemctl restart kubelet
```

再到**主节点**上 uncordor 工作节点，并查看节点状态：

```bash
kubectl uncordon k8s-node1
kubectl get nodes
```

如果工作节点版本已更新，则表明成功。

```txt
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   25h   v1.14.3
k8s-node1    Ready    <none>   25h   v1.14.3
k8s-node2    Ready    <none>   25h   v1.14.2
k8s-node3    Ready    <none>   25h   v1.14.2
```

以此类推，去升级其它的工作节点吧！

## 常用命令

* `kubectl get componentstatuses` 查看node节点组件状态
* `kubectl get svc -n kube-system` 查看系统应用
* `kubectl cluster-info` 查看集群信息
* `kubectl describe --namespace kube-system service kubernetes-dashboard` dashboard 详细服务信息
* `kubectl apply -f kube-apiserver.yaml` 更新 kube-apiserver 容器
* `kubectl delete -f kubernetes-dashboard.yaml` 删除应用
* `kubectl delete service example-server` 删除服务
* `systemctl start kube-apiserver` 启动服务。
* `kubectl get deployment -A` 所有启动的应用
* `kubectl get pod -A -o wide` 查看 pod 上的服务
* `kubectl get pod -o wide -n kube-system` 查看应用在哪个 node 上
* `kubectl describe pod --namespace=kube-system` 查看 pod 上活动信息
* `kubectl describe depoly kubernetes-dashboard -n kube-system` 查看应用详细信息
* `kubectl get depoly kubernetes-dashboard -n kube-system -o yaml` 应用信息输出到文件中
* `kubectl get service kubernetes-dashboard -n kube-system` 查看应用
* `kubectl get events` 查看事件
* `kubectl get rc`
* `kubectl get svc`
* `kubectl get namespace` 获取namespace信息
* `kubectl delete node` 删除节点
