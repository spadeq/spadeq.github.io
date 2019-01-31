---
title: 通过 Docker 部署 Grafana 和 Prometheus
date: 2018-02-11 17:07:18
updated:
categories:
- ITM
- Monitoring
tags:
- Grafana
- Prometheus
- Docker
---
目前的系统监控领域，Grafana + Prometheus 的方案得到了很多人的追捧。不得不承认第一眼看到 Grafana 的 dashboard，很难让人把它跟开源项目联系起来，真的是太！漂！亮！了！

这两个项目都提供了官方的 docker 部署方案，我们就以 docker 容器的方式进行部署。

<!-- more -->

# 基础环境

无论是把两个 app 部署在同一台 docker 主机还是分布在两台甚至多台主机，都是 OK 的，操作也基本上没有区别。由于我的部署环境为内网，所以采取了先在互联网下载镜像，然后在容器服务器上离线安装的方式。

## 获取镜像

考虑到版本管理和将来升级，不建议直接抓取镜像的 latest 版本。可以在 [Docker Store](https://store.docker.com) 上查看镜像当前的最新版本 tag 号。比如此时两个镜像的最新稳定版本分别为：

* Prometheus：prom/prometheus:4.6.3
* Grafana：grafana/grafana:v2.1.0

实际操作中下面的命令应根据查询到的版本进行修订。当然用旧版本也是可以的。

```shell
docker pull prom/prometheus:4.6.3
docker pull grafana/grafana:v2.1.0
docker pull busybox:latest

docker save prom/prometheus > prom463.tar.gz
docker save grafana/grafana > grafana210.tar.gz
docker save busybox > busybox.tar.gz
```

## 导入镜像

上面导出的这三个包，prom463 是用作 Prometheus 服务的，grafana210 和 busybox 是用作 Grafana 服务的。对于 Prometheus 容器主机：

```shell
docker image load < prom463.tar.gz
```

对于 Grafana 容器主机：

```shell
docker image load < grafana210.tar.gz
docker image load < busybox.tar.gz
```

# 容器部署

## Prometheus 服务

在 Prometheus 容器主机上，我们采取本地持久存储卷来保存 Prometheus 的配置与数据。这样做的好处是数据与配置可以在容器主机上比较方便地进行修改、备份，容器实例本身并不保存任何数据，可以做到随用随弃，便于升级。

首先创建一个本地目录用作持久化卷：

```shell
sudo mkdir /prometheus-data
```

然后在该目录下创建 Prometheus 配置文件 prometheus.yml，例如官方提供的示例内容如下：

```yaml /prometheus-data/prometheus.yml
global:
  scrape_interval: 15s

  external_labels:
    monitor: 'codelab-monitor'

scrape_configs:
  - job_name: 'prometheus'

    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']
```

接着就可以启动 Prometheus 容器了（官网的命令有误）：

```shell
docker run \
  -d \
  -p 9090:9090 \
  --name=prometheus \
  -v /prometheus-data:/prometheus-data \
  prom/prometheus:4.6.3 \
  --config.file=/prometheus-data/prometheus.yml
```

用 `docker ps` 命令可以确定容器启动成功。容器外部暴露端口不一定非得是 9090，可以自由变更。

## Grafana 服务

类似的，我们也采取本地持久卷的方式存储 Grafana 的数据。官网推荐使用 busybox 提供卷服务。假设持久化卷位于 /grafana：

```shell
sudo mkdir /grafana
docker run \
  -d \
  -v /grafana \
  --name grafana-storage
  busybox:latest
```

之后启动 Grafana 服务。由于访问 Grafana 网页服务比较频繁，我在这里直接对外暴露容器主机的 80 端口，可以免去每次在地址后输入 `:3000` 的麻烦。

```shell
docker run \
  -d \
  -p 80:3000 \
  --name=grafana \
  --volumes-from grafana-storage \
  grafana/grafana:v2.1.0
```

在浏览器访问本机 IP 地址，不用任何冒号后缀，如果能显示 Grafana 的登录界面，就表示部署成功啦。

# Prometheus 与 Grafana 的集成

To Be Continu'd.