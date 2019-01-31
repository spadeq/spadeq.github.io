---
title: 允许 RAID 总线类型下强制启用存储空间直通
date: 2017-08-25 14:13:15
updated: 2017-08-25 14:13:15
categories:
- ITM
- Cloud
tags:
- Windows
- Hyperconverged
- ServerSAN
---
存储空间直通（Storage Spaces Direct，S2D）是 Windows Server 2016 引入的 Server SAN 技术，针对的就是 VMware vSAN，其目的是让 Windows Server 原生具备作为超融合架构底层的能力。

这本来是个非常好的东西，但有一个非常令人不爽的限制，那就是加入 S2D 资源池的本地硬盘，其总线类型必须为 SAS、NVMe 等类型，而不能是 RAID。可惜的是，目前市面上的多数阵列卡，虽然提供了 JBOD 的磁盘模式，但从 Windows 中看到的总线类型却依然是 RAID。检查方式如下：

``` powershell
Get-Disk | select Number, FriendlyName, OperationalStatus, Size, PartitionStyle, BusType, MediaType | sort Number | ft -AutoSize
```

在 BusType 那一栏中显示的就是磁盘的总线类型。如果有 RAID，那么在创建故障转移群集时，就会验证失败。

解决的办法也很简单，首先，以不启用 S2D 的选项，先创建集群。待集群创建完毕后，在其中一台机器上执行：

``` powershell
(Get-Cluster).S2DType = 256
```

这里可以赋值的选项有三种，分别为：

* 256，即 `0x100`，仅 RAID
* 4294967295，即 `0xFFFFFFFF`，全类型
* 134144，即 `0x20C00`，微软推荐的默认类型

修改完毕后，再启用 S2D：

``` powershell
Enable-ClusterS2D -AutoConfig:0 -SkipEligibilityCheck
```

在故障转移群集管理器或者 SCVMM 中，就可以管理资源池了。