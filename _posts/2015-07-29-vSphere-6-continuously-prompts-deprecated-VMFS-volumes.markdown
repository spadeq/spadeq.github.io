---
title: vSphere 6 总提示已弃用 VMFS 卷
date: 2015-07-29 12:50:29
categories:
- ITM
- Cloud
tags:
- VMware
---
我昨天给几台 ESXi 6 主机挂载了新的存储，结果主机全部变叹号，点进去看，提示下面的错误：

> Deprecated VMFS volume(s) found on the host. Please consider upgrading volume(s) to the latest version.

即「已弃用 VMFS 卷」。这让我很奇怪，仔细检查所有已挂载存储，全都是 VMFS 5，并没有低版本，更没办法升级。原来这是 vSphere 6 的一个已知 bug，解决方法是重启管理代理，方法有两种。

一是去 ESXi 主机控制台上按 F2，以 root 登陆，然后选择 Restart Management Agents。

二是通过 SSH 登陆 ESXi 主机。具体步骤如下：

1. 用 Client 登陆 vCenter，找到这台主机，在其 Configuration 中选择 Security Profile，点右边的 Properties
1. 在列表中选择 SSH，点击 Options，再点 Start
1. 用 SSH 工具，以 root 方式登陆该主机
1. 运行 `/etc/init.d/hostd restart`
1. 运行 `/etc/init.d/vpxa restart`
1. 如果此时在 vCenter 中看到主机被断开，可以右键点击主机，点 Connect
1. 参照第 1 步，关闭 SSH 服务。