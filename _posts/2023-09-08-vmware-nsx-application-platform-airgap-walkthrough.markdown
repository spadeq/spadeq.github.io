---
layout: post
title: VMware NSX Application AirGap 通关攻略
date: 2023-09-08 16:00:00
categories:
  - IT
tags:
  - VMware
  - NSX
  - Kubernetes
---

本文介绍如何在离线环境下，搞定 NSX Application Platform。难度指数极高，请耐心、细心。

## 证书

离线环境最重要最麻烦的，就是证书，需要自行搞定从 CA 到服务器证书的一整套操作。建议首先学习一下 PKI 的基本理论。

### 根 CA

为了简单起见，建议用虚拟机先装个 Windows Server 2022，开启证书管理功能，生成一个根 CA。因为目前这种方式生成出来的根 CA 兼容性最好。

将 Windows 生成的根 CA 保存下来，这是个 p12 的二进制格式。用证书工具将其中的证书和 key 以文本形式拆出来。

（以上内容后续将提供具体命令）

