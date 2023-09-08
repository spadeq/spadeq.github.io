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

### VMCA

VMCA 是 vCenter 使用的中间 CA。SSH 以 root 用户登录 vCenter，执行 `shell` 切换到命令行，然后执行：

```bash
/usr/lib/vmware-vmca/bin/certificate-manager
```

* 选择 2，然后依次输入信息。注意 IP 和主机名必须输入 vCenter 的，之后路径可以写 /tmp，此工具会在目录下生成一个 key 文件和一个 csr 文件。之后保持选单不动。
* 浏览器访问 Windows Server 的地址，<http://[IP]/certsrv>。
* 点击申请证书、高级证书申请，把上一步生成的 csr 文件内容复制到框里，点击提交。
* 登录到 Windows Server 内，运行 `certsrv.msc`，选择挂起的申请，找到刚才的申请条目，右键点击颁发。
* 回到浏览器，退回到最初的页面，点击查看挂起的证书申请的状态，然后点击链接，选择 Base64，下载。
* 下载的文件内容打开，将其中内容复制到 vCenter 的 `/tmp/vmca_issued_cer.cer` 中，然后再把根 CA 的证书内容续在后面，保存。
* 回到 vCenter 之前的 certificate-manager 选单，这次选择 1，导入。
* 证书输入 `/tmp/vmca_issued_cer.cer`，key 输入 `vmca_issued_key.key`，等待进度走完。
* 到 vCenter 页面上确定证书更换成功。

  
