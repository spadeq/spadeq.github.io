---
title: 通过 zerofree 回收 Ubuntu 虚拟机中的未使用空间
date: 2017-08-11 08:30:30
categories:
- ITM
- Linux
tags: 
- VMware
- Linux
---
本方法适合于回收 VMware Workstation 中的 Ubuntu 虚拟机未使用空间。硬盘格式必须为精简置备，如果是提前分配全部空间的磁盘，本方法无效。ESXi 下的虚拟机，只有 zerofree 步骤适用，在最新的 VMFS6 格式分区上，未使用空间会被自动回收。另外，文件系统必须是 ext（4、3、2 都可以），其它文件系统无效。

<!-- more -->

# 前置条件

首先你必须有 root 用户的口令，如果没有，可以 `sudo su -` 切换到 root 用户下，然后执行 `passwd`。

其次，必须安装了 vmware-tools，当然，可以用 open-vm-tools 来代替，执行：

    sudo apt install open-vm-tools zerofree

# 回收 OS 的空间

重启 Ubuntu 服务器，按住 shift，在选单中选择 Advanced，然后选择最新内核对应的 Recovery，进入恢复模式。

在恢复菜单中，选择 root 项，输入 root 口令。

重新以只读方式挂载根分区：

    mount -o remount,ro /

执行置零：

    zerofree -v /dev/mapper/ubuntusource--vg-root

注意，如果没有使用 LVM，则可以直接使用 /dev/sda1 这样的路径名称。如果还有 home 等其它的分区，那么也分别进行只读挂载和 zerofree 操作。
等进度走完，就说明置零操作成功。

# 缩减 vmdk 大小

重新以正常方式启动虚拟机，执行：

    sudo vmware-toolbox-cmd disk shrinkonly

等待进度完成。如果虚拟机磁盘是多文件的话，速度会稍微快一点，单文件会非常慢，耐心等待吧。