---
title: ESXi 虚拟机 vmdk 压缩瘦身
date: 2015-10-13 15:12:14
updated: 2017-08-16 16:27:00
categories: 
- ITM
- Cloud
tags:
- VMware
---
为了节约虚拟机的磁盘占用，VMware 提供了精简置备（Thin Provision）方式，使得磁盘占用按需分配。但是，虚拟机内删除文件虽然释放了 Guest OS 的可用空间，但却不会使得 vmdk 文件相应地缩小。随着虚拟机的使用，vmdk 势必不断增大。对于桌面级 VMware Workstaion，可以通过 Disk Cleanup 功能或者 vmware-tools 的命令来释放未用空间，而 ESXi 并没有直接提供这个功能。本文介绍如何通过手动操作，对精简置备的 vmdk 进行瘦身。
<!-- more -->

# Guest OS 准备

首先，我们需要在 Guest OS 上进行准备工作，对未使用的磁盘进行置零操作。

对于Linux虚拟机，可以使用 zerofree 工具，可参见 [此文](/2017/08/11/zerofree-unused-spaces-in-ubuntu/)。

对于 Windows 虚拟机，我们需要用到另一个工具——SDelete，这是微软 Sysinternals 工具中的一款，可以 [点击这里下载](https://technet.microsoft.com/en-us/sysinternals/bb897443)。

将 SDelete.exe 复制到虚拟机中，对每个盘符执行下面的语句（最后的 C: 依次替换为每个盘符）

    SDelete.exe -z C:

等待进度走完（有时候会走到 102% -。-），即宣告准备完毕。

要注意的是，清零过程中精简置备的磁盘会膨胀到设置的大小，因此请确保存储空间足够。

# 压缩 VMDK

在 vSphere Client 中，将虚拟机关机。找到该虚拟机对应的主机，开启主机的 SSH 登录。随后，登入主机，在 /vmfs/volumes 下找到虚拟机目录，查看其对应的 vmdk 文件。注意，这里面可能会有以 -flat 和 -ctk 等后缀的 vmdk，这不是我们要操作的，只有不带后缀的 vmdk 需要进行操作。对它们依次执行命令：

    vmkfstools -K <vm-name>.vmdk

耐心等待进度走完。完成之后记得把 SSH 服务关掉。

再次开启虚拟机，你会看到存储使用那里已经是压缩后的大小了。Good luck。