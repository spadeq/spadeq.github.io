---
title: NPIV 的那点事儿
date: 2015-02-02 17:07:57
categories:
- ITM
- Devices
tags:
- NPIV
- Storage
---
NPIV 做的是 HBA 卡的虚拟化，说白了就是让一个HBA卡拥有多个 WWN，分给物理机和虚拟机，让它们在 SAN 网络中能够玩的更灵活。要实现 NPIV，必须HBA 卡、SAN 交换机同时支持才行，不过好在市面上近五年来的产品应该都支持这项技术了。

我这次应用 NPIV 的场景，是在虚拟机上部署 Symantec Netbackup，既有常规的磁带库备份，也有带 dedup 的虚机备份。为此，我需要两个虚拟服务器，分别在 SAN 网络上连接磁带库和存储，算是比较综合的一个应用了。其过程还是比较坎坷的，因为网上靠谱的资料实在太少。这里写成一个 manual 的形式，以作备忘。

# 虚拟环境配置

虽然我是 VMware 的拥趸，但如果你想用 ESXi 实现上面提到的目标，现在（5.5 update 2）是不可能的。因为 ESXi 的 NPIV 很奇葩，只是给 RDM 裸机映射的虚拟机分配一个 WWN 号，你这不是逗我玩么，我都已经通过 RDM 连接到 LUN 了，我还要这个 WWN 干啥？你丫还不支持带库。。因此，这里的 hypervisor 我们只能选择 Hyper-V。本文使用的是无需花钱的 Windows Hyper-V Server 2012 R2。

抱歉，太监了。。