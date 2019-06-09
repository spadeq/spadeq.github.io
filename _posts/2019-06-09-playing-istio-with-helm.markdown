---
layout: post
title: Istio 动手玩
date: 2019-06-09 10:51:00
categories: 
- Infrastructure
tags:
- Kubernetes
- Docker
- Helm
- Istio
- Service Mesh
---

## 安装

### 获取 Istio 安装包

从 [Istio Git](https://github.com/istio/istio/releases) 下载最新的二进制包（注意不一定是排在第一位的），上传到 k8s 的 master 节点，执行：

```bash
tar zxvf istio-*
```

### 修改镜像来源

大家应该看出套路了，凡是在国内或者内网倒腾 k8s 系列玩具，总免不了要在容器镜像上下点文章。不过 istio 的大部分镜像都是在 docker hub 上的，没有用到 gcr，会省事很多。首先获取镜像列表。

```bash
cd istio-*/install/kubernetes
grep -r image: istio-demo.yaml | egrep -o -e "image:.*" | sort | uniq
```

大概输出如下：

```text
image: "docker.io/istio/citadel:1.1.8"
image: "docker.io/istio/galley:1.1.8"
image: "docker.io/istio/kubectl:1.1.8"
image: "docker.io/istio/kubectl:1.1.8"
image: "docker.io/istio/mixer:1.1.8"
image: "docker.io/istio/pilot:1.1.8"
image: "docker.io/istio/proxy_init:1.1.8"
image: "docker.io/istio/proxyv2:1.1.8"
image: "docker.io/istio/sidecar_injector:1.1.8"
image: "docker.io/jaegertracing/all-in-one:1.9"
image: "docker.io/kiali/kiali:v0.16"
image: "docker.io/prom/prometheus:v2.3.1"
image: "grafana/grafana:6.0.2"
image: [[ annotation .ObjectMeta `sidecar.istio.io/proxyImage`  "docker.io/istio/proxyv2:1.1.8"  ]]
```

这些 docker.io 开头需要 login，所以也建议先抓下来。

如有必要，可以修改 `install/kubernetes/helm/istio-init/values.yaml` 中的镜像地址。

### 准备模板文件

执行：

```bash
helm template install/kubernetes/helm/istio --name istio --namespace istio-system > my-istio.yaml
```

### 部署集群

执行：

```bash
kubectl create ns istio-system
kubecty apply -f my-istio.yaml
```

### 查看 Istio 服务状态

执行：

```bash
kubectl get pod -n istio-system -w
```
