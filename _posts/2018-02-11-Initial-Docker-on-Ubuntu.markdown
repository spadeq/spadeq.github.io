---
layout: post
title: Ubuntu 上运行 Docker 的基础配置
date: 2018-02-11 15:51:21
updated:
categories: 
- Linux
- Container
tags:
- Docker
---
此文思路也可以用于其他发行版安装 Docker。

# 安装

从 Docker 官网的 repo 中下载 docker-ce 与 Ubuntu 对应版本的 deb 包，上传到服务器，然后执行：

```shell
sudo dpkg -i docker-ce*
sudo apt install -f
```

# 权限组配置

这时候如果你执行 `docker ps` 之类的命令就会提示权限错误，必须使用 `sudo`。为了避免以后频繁使用 `sudo`，可以做以下的权限配置。

首先将当前用户加入到 docker 组中（替换自己的用户名）：

```shell
sudo usermod -aG docker <login name>
```

然后重启服务器。如果不想服务器，希望立刻生效，则手动赋予权限：

```shell
sudo chmod a+rw /var/run/docker.sock
```

# 国内镜像源加速

在国内直接使用 Docker 官方源来下载镜像是很不明智的行为，我们可以通过简单地修改一个配置文件，实现镜像加速。

```shell
sudo vi /etc/docker/daemon.json
```

内容如下：

```json
{
    "registry-mirrors":["https://registry.docker-cn.com"]
}
```

然后重启 Docker 服务：

```shell
sudo systemctl restart docker.service
```