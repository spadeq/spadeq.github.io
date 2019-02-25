---
layout: post
title: 设置 microk8s 环境
date: 2019-02-25 14:56:00
categories: 
- Container
tags:
- Kubernetes
- microk8s
- docker
- Container
---

真实的生产 Kubernetes 集群需要多节点，而且部署较为复杂。在学习和开发测试环节中，我们更加倾向于部署一个单机版 Kubernetes。这就是本文使用的 microk8s。

# 基本软件包安装

推荐使用 Ubuntu，原生支持 snap。安装命令：

```shell
snap info microk8s
sudo snap install microk8s --classic
microk8s.status
microk8s.enable dns dashboard
sudo snap alias microk8s.kubectl kubectl
sudo snap alias microk8s.docker docker
```

注意不需要另行安装 docker，microk8s 已经自带包含了。`snap info` 命令可以查看当前 microk8s 所对应的版本，默认安装最新的 stable 版。最后两条命令是将 microk8s 开头的命令映射到通常使用的命令，简化操作。

# 镜像源优化

由于众所周知的原因，国内如果要好好的玩容器和 k8s，是必须对网络进行优化的。。

## Docker 镜像

修改 `~/snap/microk8s/current/etc/docker/daemon.json` 文件，加入以下内容。由于 snap 的特殊文件系统机制，不可以直接修改 /snap 目录下的文件，也不使用系统自身的 /etc/docker 目录作为配置。

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

然后重启 microk8s：

```shell
sudo snap restart microk8s
```

可以随便 pull 一个镜像，测试是不是切换到国内源。

## k8s 容器

如果这时候你直接进行 `kubectl create` 等操作，会发现容器创建不了，始终处于 ContainerCreating 状态。用 `kubectl describe pod <podname>` 查看报错信息可以发现，是因为无法从 k8s.gcr.io 上拉取镜像。解决方法有多种，我个人是采用 docker 中央源 + 改 tag 的办法。

首先，将所需要的容器从中央源拉取下来（注意将版本号和镜像名替换为 describe 报错中对应的内容）：

```shell
docker pull mirrorgooglecontainers/pause:3.1
docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
```

当然全套 kubernetes 需要的镜像有很多，需要一个一个手工操作。（蛋疼）