---
layout: post
title: 使用 NFS 作为 Docker Swarm 持久卷
date: 2019-06-09 20:39:00
categories: 
- Infrastructure
tags:
- Kubernetes
- Docker
- Storage
---

折腾了两天，最后还是决定先拿 NFS 和 Swarm 组合，做一个最简单的集群，防止第一个应用上线就陷入 k8s 的汪洋大海。。

## Swarm 集群

注意替换 IP 和 token 为实际。先是主节点：

```bash
docker swarm init --advertise-addr 10.0.0.220
```

然后是工作节点：

```bash
docker swarm join --token SWMTKN-1-1761ucm1qtwp5zudzrpej70bsg3oh91s58m96nl1aw8outyx4e-7qho23q4nr65byy9u61amrzgm 10.0.0.220:2377
```

最后在主节点确认都已经加入，所有节点应该都是 Active。

```bash
docker node ls
```

## NFS 服务

首先，你得有一个 NFS 服务器，假定是 `10.0.0.200`，共享的目录是 `/volumes1/swarm`。

在每个 swarm 节点安装 NFS 客户端，并检查可挂载。

```bash
sudo apt install -y nfs-common
sudo showmount -e 10.0.0.200
```

能显示出 NFS 服务器上的卷，表明是可挂载的。

```text
Export list for 10.0.0.200:
/volume1/swarm *
```

## Stack 配置文件引用

既然是集群，我们就不会希望跑到每个节点上面去配 NFS 挂载和映射。做集群共享卷的关键，要在 compose 文件中创建 volume。语法如下：

```yaml
volumes:
  my-vol:
    driver_opts:
      type: "nfs"
      o: "addr=10.0.0.200,nolock,soft,rw"
      device: ":/volume1/swarm"
```

注意替换 o 属性中的 NFS 服务器地址，以及 device 属性中的挂载点。为了数据安全，现在我们就只能为每个 volume 建立一个 nfs 挂载点。

## 与应用结合

下面我们来一个稍微复杂一点的 stack 配置，Prometheus 与 Grafana 两个容器，其中 Grafana 挂载 NFS 卷作为持久化存储，两个容器通过 overlay 网络互访。

```yaml
version: '3.7'

services:
  grafana:
    image: grafana/grafana:latest
    hostname: grafana
    deploy:
      restart_policy:
        delay: 10s
        max_attempts: 10
        window: 60s
    networks:
      - monitor_distributed
    ports:
      - 3000:3000
    volumes:
      - grafana-data:/var/lib/grafana

  prometheus:  
    image: prom/prometheus:latest
    hostname: prometheus
    deploy:
      restart_policy:
        delay: 10s
        max_attempts: 10
        window: 60s
    networks:
      - monitor_distributed

networks:
  monitor_distributed:
    driver: overlay

volumes:
  grafana-data:
    driver_opts:
      type: "nfs"
      o: "addr=10.0.0.200,nolock,soft,rw"
      device: ":/volume1/grafana"
```

执行命令启动 stack：

```bash
docker stack deploy --compose-file=docker-compose.yaml monitor_stack

docker stack ls
docker stack service monitor_stack
```

直到两个容器都运行起来。

### 存储分析

执行命令查看卷：

```bash
docker volume ls

docker volume inspect monitor_stack_grafana-data
```

结果如下：

```json
[
    {
        "CreatedAt": "2019-06-09T13:34:03Z",
        "Driver": "local",
        "Labels": {
            "com.docker.stack.namespace": "monitor_stack"
        },
        "Mountpoint": "/var/lib/docker/volumes/monitor_stack_grafana-data/_data",
        "Name": "monitor_stack_grafana-data",
        "Options": {
            "device": ":/volume1/grafana",
            "o": "addr=10.0.0.200,nolock,soft,rw",
            "type": "nfs"
        },
        "Scope": "local"
    }
]
```

可以看到，实际上 NFS 目录还是被挂载到节点宿主机上的，只不过这一过程不需要我们人工干预，swarm 会自动根据容器位置进行动态挂载。

在这几个节点上分别执行 `docker ps`，找到 Grafana 所在的主机，然后执行（注意替换四位 id 为自己的实际）：

```bash
docker exec -it d313 /bin/bash
mount
```

其中一行应该是

```text
:/volume1/grafana on /var/lib/grafana type nfs (rw,relatime,vers=3,rsize=131072,wsize=131072,namlen=255,soft,nolock,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.0.0.200,mountvers=3,mountproto=tcp,local_lock=all,addr=10.0.0.200)
```

为了确认，还可以去 NFS 服务器上打开目录看看里面是不是有 Grafana 生成的文件。

### 多实例共享分析

我们把 Grafana 的副本数增加一个：

```bash
docker service scale monitor_stack_grafana=2
```

稍等一下，用 `docker service ls` 就会发现 Grafana 容器的副本数增加为 `2/2`。找到另一个容器所在的目录，然后同样执行上一节的 `mount` 命令。结果应该是一样的。

要注意的是，这种共享持久化卷怎么说都还是 NFS，文件级的共享是可以的。数据库这种还是要依赖应用自己的高可用。
