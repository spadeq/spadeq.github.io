---
layout: post
title: Helm 与 Kubernetes 集成部署
date: 2019-06-08 21:12:00
categories: 
- Infrastructure
tags:
- Kubernetes
- Docker
- Helm
---

## Helm 二进制安装

从 [Helm Git](https://github.com/helm/helm/releases) 下载最新的二进制包，上传到 k8s 的 master 节点，执行：

```bash
tar zxvf helm-v*
sudo cp linux-amd64/helm /usr/local/bin/

helm version
```

## Helm 集群初始化

Helm 的服务端组件叫做 Tiller，我们采用集群安装的方式来进行初始化。

按理说命令是很简单的，只要 `helm init`。但由于我们伟大的内网不能访问到某些重要的站，所以还得 workaround 一下。首先是加入一个不从互联网更新 charts 库的参数：

```bash
helm init --skip-refresh
```

这时虽然提示成功，但你用 `helm version` 命令还是会看到没有 tiller。接着排查：

```bash
kubectl get pod -A -o wide
```

会看到亲切的 ImagPullBackOff 错误：

```text
kube-system   tiller-deploy-66b7dd976-6njbz           0/1     ImagePullBackOff   0          4m24s   10.244.1.4   k8s-node1    <none>           <none>
```

那么继续，看看到底需要的是哪个镜像：

```bash
kubectl describe pod -n kube-system tiller-deploy-66b7dd976-6njbz
```

找到对应的 warning：

```text
  Warning  Failed     4m9s (x2 over 5m10s)   kubelet, k8s-node1  Failed to pull image "gcr.io/kubernetes-helm/tiller:v2.14.1": rpc error: code = Unknown desc = Error response from daemon: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

好，用你喜欢的办法拿到 `gcr.io/kubernetes-helm/tiller:v2.14.1` 这个镜像吧，导入到各个 node 里面，然后再看 pod 状态：

```bash
kubectl get pod -A
```

还有一个办法就是在 init 的时候指定某个镜像，类似于：

```bash
helm init --skip-refresh --tiller-image=xxxxx/tiller:xxxx
```

可以看到已经 running 了，再用 `helm version` 查看，就能看到 server 端的版本了。
