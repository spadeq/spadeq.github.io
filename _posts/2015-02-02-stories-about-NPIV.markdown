---
title: NPIV的那点事儿
date: 2015-02-02 17:07:57
categories:
- ITM
- Devices
tags:
- NPIV
- Storage
---
NPIV做的是HBA卡的虚拟化，说白了就是让一个HBA卡拥有多个WWN，分给物理机和虚拟机，让它们在SAN网络中能够玩的更灵活。要实现NPIV，必须HBA卡、SAN交换机同时支持才行，不过好在市面上近五年来的产品应该都支持这项技术了。

我这次应用NPIV的场景，是在虚拟机上部署Symantec Netbackup，既有常规的磁带库备份，也有带dedup的虚机备份。为此，我需要两个虚拟服务器，分别在SAN网络上连接磁带库和存储，算是比较综合的一个应用了。其过程还是比较坎坷的，因为网上靠谱的资料实在太少。这里写成一个manual的形式，以作备忘。

# 虚拟环境配置

虽然我是VMware的拥趸，但如果你想用ESXi实现上面提到的目标，现在（5.5 update2）是不可能的。因为ESXi的NPIV很奇葩，只是给RDM裸机映射的虚拟机分配一个WWN号，你这不是逗我玩么，我都已经通过RDM连接到LUN了，我还要这个WWN干啥？你丫还不支持带库。。因此，这里的hypervisor我们只能选择Hyper-V。本文使用的是无需花钱的Windows Hyper-V Server 2012 R2。

[抱歉，太监了。。]