---
title: 修复 S2D 的 Storage Pool
date: 2017-08-25 16:09:19
updated:
categories:
- ITM
- Cloud
tags:
- Windows
- Storage Spaces
- Hyperconverged
---
如果在折腾 S2D 的时候，不小心把一整个 pool 的状态都搞得不对，磁盘什么的乱七八糟，可以用下面的方法进行整体的修复。

# 重置可写状态

通过下面的命令，先查看一下 Storage Pool 的状态。

``` powershell
Get-StoragePool
```

关键看最后一栏「IsReadOnly」。如果是 True，则执行下面的命令：

``` powershell
Get-StoragePool -FriendlyName <PoolName> | Set-StoragePool -IsReadOnly $False
```

# 删除错误的存储池

继续利用 powershell 的通道指令，将其删除：

``` powershell
Get-StoragePool -FriendlyName <PoolName> | Remove-StoragePool
```

# 重置磁盘

执行 `Get-PhysicalDisk` 命令，查看磁盘状态。如果其 OperationalStatus 不是 OK，则需要重置。

``` powershell
Get-PhysicalDisk | ? OperationalStatus -eq "Unrecognized Metadata" | Reset-PhysicalDisk
```

如果 CanPool 不是 True，则需要重新初始化磁盘。根据实际情况执行下面的语句：

``` powershell
Get-PhysicalDisk -Canpool $False | Get-Disk | Set-Disk -isReadonly $False
Get-PhysicalDisk -Canpool $False | Get-Disk | Set-Disk -isOffline $False
Get-PhysicalDisk -Canpool $False | Get-Disk | Clear-Disk -RemoveData -Confirm:$False
```