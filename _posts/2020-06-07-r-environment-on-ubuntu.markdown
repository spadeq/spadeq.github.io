---
layout: post
title: Ubuntu 上的 R 环境配置
date: 2020-06-07 16:42:00
categories:
  - Development
tags:
  - R
  - Linux
---

本文介绍如何在 Ubuntu 上配置基础的 R 开发环境，以及 Visual Studio Code 插件配置。

## 安装 R 基础软件

首先在创建新的源列表 `/etc/apt/sources.list.d/cran.list`，内容如下：

```list
deb https://mirrors.ustc.edu.cn/CRAN/bin/linux/ubuntu focal-cran40/
```

然后执行：

```bash
sudo apt remove -y gpg
sudo apt install -y gnupg1
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
sudo apt install -y r-base
```

注意首先必须删除系统自带的 gpg，否则添加 apt key 的时候会报「IPC connect call failed」错误。

## 配置 Language Server

首先安装依赖：

```bash
sudo apt install -y libcurl4-openssl-dev libssl-dev libxml2-dev
```

然后执行 `R` 进入 R 环境，安装包：

```r
install.packages("languageserver", INSTALL_opts = c('--no-lock'))
```

注意最后的 `--no-lock` 参数，如果不提供的话，可能会出现「Permission denied ERROR: moving to final location failed」错误。