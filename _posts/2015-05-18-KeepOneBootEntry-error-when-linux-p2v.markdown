---
title: P2V Linux 遇到 KeepOneBootEntry 错误的解决方案
date: 2015-05-18 12:56:44
categories:
- ITM
- Cloud
tags:
- P2V
- Linux
---
在对一台 RHEL 5.x 物理机进行 p2v 的过程中，进度 97% 时提示一个错误导致转换失败，具体错误提示为：
> FAILD: An error occurred during the conversion: 'KeepOneBootEntry: There is no matching kernel modules for kernel /xen.gz-2.6.18-194.el5'

导致这个错误的原因，是该物理机 RHEL5 在安装时选择了虚拟化组件（Xen），这一点可以通过 `uname -a` 进行确认，当前操作系统确实是运行在 xen 核心下的。因此，这个时候的 Linux 本身就是一个 hypervisor，而 VMware 无法对 hypervisor 进行虚拟化。

解决方案就很简单了，删除虚拟化组件：

    yum remove libvirt

然后修改 /boot/grub/menu.lst，将带有 xen 的启动项注释掉（记得先备份），使操作系统从常规的 kernel 启动。之后再启动 p2v job，就 ok 啦。