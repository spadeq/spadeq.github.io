---
layout: post
title: 在 Docker Swarm 上部署 Minio 集群
date: 2019-05-13 17:23:00
categories: 
- Documents
tags:
- LaTeX
- Visual Studio Code
---

## 基本环境

操作系统使用 Ubuntu 18.04，标准安装。/ 挂载点不需要太大，默认虚拟机的 16G 已够用。另外挂载数据盘，建议 1T 以上，使用 LVM。

Minio 集群至少需要四个节点，因此至少安装 4 台虚拟机。主机名随意，最好以数字后缀区分。

### 创建文件系统

假设新挂载的硬盘为 /dev/sdb，执行下列操作：

```shell
sudo apt dist-upgrade
sudo vgcreate vg_docker /dev/sdb
sudo lvcreate -n lv_docker -l 100%FREE vg_docker
sudo mkfs.ext4 /dev/mapper/vg_docker-lv_docker
sudo mkdir /docker-data
sudo mount /dev/mapper/vg_docker-lv_docker /docker-data
df -h
```

为了自动启动后仍可自动挂载，将下面一行加入 /etc/fstab 中：

```conf
/dev/mapper/vg_docker-lv_docker /docker-data ext4 defaults 0 0
```

### 安装 Docker

执行 `sudo apt install docker.io` 安装 docker。

修改 /etc/docker/daemon.js （不存在就新建），内容如下：

```json
{
  "data-root": "/docker-data"
}
```

然后赋予当前用户执行 docker 命令的权限：

```shell
sudo usermod -aG docker `whoami`
sudo chmod a+rw /var/run/docker.sock
```

最后启动 Docker：

```shell
sudo systemctl enable docker
sudo systemctl start docker

docker info
```

在信息中，确定 Docker Root Dir 的内容是 /docker-data。

## Swarm 集群初始化

假定现在四台服务器分别为 minio-1、minio-2、minio-3、minio-4，选择 minio-1 为主节点，另外三个为工作节点。

### 主节点

执行以下命令（将 `<ip>` 替换为节点实际地址：

```shell
docker swarm init --advertise-addr <ip>
```

然后会有一串提示：

    To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1m6rb67e2i6l1hingm0m9640432rhpkwimw0fdoli405x9tz4o-14pklnbph8h0tw61y78p0x18o <ip>:2377

这就是工作节点加入所使用的命令。

### 工作节点

将上一步提示的那条命令在所有工作节点上挨个执行一遍。

回到主节点上执行：

```shell
docker node ls
```

如果出现类似下面的输出，就说明集群创建成功了。

    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
    x10titjzvr1cmwmm7d2pujyto *   minio-1             Ready               Active              Leader              18.09.2
    6ec4ehqm33fmclea6bj4j6u5g     minio-2             Ready               Active                                  18.09.2
    o0s2ip8juud0bmfzipooljm42     minio-3             Ready               Active                                  18.09.2
    hvmesf3vqojf2n0v2mlcodla0     minio-4             Ready               Active                                  18.09.2

## 部署 minio 服务

提示：如果是在内网部署，需要将 minio/minio:latest 镜像进行人肉搬运，用 `docker image load < minio.tar` 加载到**每个**节点上，然后用 `docker image ls` 检查一下。

### 创建密钥

手动生成 S3 接口使用的 access key 和 secret key。然后在主节点执行（替换成自己的 key）：

```shell
echo "ABCDEFGHIJKLMN123456" | docker secret create access_key -
echo "AaBbCcDdEeFfGgHhIiJjKkLlMmNn1234567890Zz" | docker secret create secret_key -
```

### 标记节点

为了绑定容器与节点之间一一对应的关系，给节点打上标签：

```shell
docker node update --label-add minio-node1=true minio-1
docker node update --label-add minio-node2=true minio-2
docker node update --label-add minio-node3=true minio-3
docker node update --label-add minio-node4=true minio-4
```

### Stack compose 文件

我们使用 compose 文件来配置 stack。Minio 官方给了一个示例文件 docker-compose-secrets.yaml，我们稍加改动（主要是主机名、标签名）即可：

```yaml docker-compose-secrets.yaml
version: '3.1'

services:
  minio1:
    image: minio/minio:latest
    hostname: minio1
    volumes:
      - minio1-data:/export
    ports:
      - "9001:9000"
    networks:
      - minio_distributed
    deploy:
      restart_policy:
        delay: 10s
        max_attempts: 10
        window: 60s
      placement:
        constraints:
          - node.labels.minio-node1==true
    command: server http://minio1/export http://minio2/export http://minio3/export http://minio4/export
    secrets:
      - secret_key
      - access_key

  minio2:
    image: minio/minio:latest
    hostname: minio2
    volumes:
      - minio2-data:/export
    ports:
      - "9002:9000"
    networks:
      - minio_distributed
    deploy:
      restart_policy:
        delay: 10s
        max_attempts: 10
        window: 60s
      placement:
        constraints:
          - node.labels.minio-node2==true
    command: server http://minio1/export http://minio2/export http://minio3/export http://minio4/export
    secrets:
      - secret_key
      - access_key

  minio3:
    image: minio/minio:latest
    hostname: minio3
    volumes:
      - minio3-data:/export
    ports:
      - "9003:9000"
    networks:
      - minio_distributed
    deploy:
      restart_policy:
        delay: 10s
        max_attempts: 10
        window: 60s
      placement:
        constraints:
          - node.labels.minio-node3==true
    command: server http://minio1/export http://minio2/export http://minio3/export http://minio4/export
    secrets:
      - secret_key
      - access_key

  minio4:
    image: minio/minio:latest
    hostname: minio4
    volumes:
      - minio4-data:/export
    ports:
      - "9004:9000"
    networks:
      - minio_distributed
    deploy:
      restart_policy:
        delay: 10s
        max_attempts: 10
        window: 60s
      placement:
        constraints:
          - node.labels.minio-node4==true
    command: server http://minio1/export http://minio2/export http://minio3/export http://minio4/export
    secrets:
      - secret_key
      - access_key

volumes:
  minio1-data:

  minio2-data:

  minio3-data:
  
  minio4-data:

networks:
  minio_distributed:
    driver: overlay

secrets:
  secret_key:
    external: true
  access_key:
    external: true
```

### 启动集群

启动集群命令：

```shell
docker stack deploy --compose-file=docker-compose-secrets.yaml minio_stack
```

如果不出意外，在四个节点上都会分别运行一个 docker 容器。通过 `docker logs -f <container-id>` 可以查看日志，如果有类似下面的输出，表示启动成功：

    Waiting for the first server to format the disks.
    Status:         4 Online, 0 Offline.
    Endpoint:  http://10.255.0.47:9000  http://10.0.2.6:9000  http://172.18.0.3:9000  http://127.0.0.1:9000

    Browser Access:
       http://10.255.0.47:9000  http://10.0.2.6:9000  http://172.18.0.3:9000  http://127.0.0.1:9000

### 删除集群

如果出现错误，想推导重来，那么可以通过下面这条命令删掉这个 stack：

```shell
docker stack rm minio_stack
```

集群会在每个节点逐个删除容器。如果删不掉，就只能手动 kill 了。创建的 volume 不会自动删除，需要在**每个**节点上手动执行下面的命令进行清除：

```shell
docker volume prune
```

## 使用 Minio

### 单节点使用

在之前的集群配置中，我们对每个工作节点上的容器采用从 9001 到 9004 等不同的端口号来区分服务，而这几个端口在所有的节点 IP 上都是可以对外访问的（docker swarm mesh route 的功能），即 <http://minio-1:9001>、<http://minio-2:9001> 都可以访问容器 1。而无论访问哪个容器，看到的数据也是一样的。

### 负载均衡

既然是集群，那么单节点的使用方式就显得太 low 了。下面我们在集群之前通过 HAProxy 作为负载均衡器，实现一个地址和一个端口对外服务（HAProxy 可进一步通过 keepalived 实现冗余，在这里不做介绍了）。

首先新安装一台 Ubuntu 服务器，然后安装 haproxy：

```shell
sudo apt install haproxy vim-haproxy haproxy-doc
```

修改 haproxy 配置 /etc/haproxy/haproxy.cfg，在后面增加：

```conf
frontend minio_front
        bind *:9000
        stats uri /haproxy?stats
        default_backend minio_back

backend minio_back
        balance roundrobin
        server minio1 11.64.19.231:9001 check
        server minio2 11.64.19.232:9002 check
        server minio3 11.64.19.233:9003 check
        server minio4 11.64.19.234:9004 check
```

启动服务：

```shell
sudo systemctl enable haproxy
sudo systemctl start haproxy
```

最好能重启一下服务器。通过 <http://[ha-ip]:9000> 即可以负载均衡的方式访问服务。同时可以从 <http://[ha-ip]:9000/haproxy?stats> 取得监控数据。