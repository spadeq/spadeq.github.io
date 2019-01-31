---
title: vCenter Server Appliance 6.0 打补丁
date: 2015-05-14 12:59:31
categories:
- ITM
- Cloud
tags:
- VMware
- Patch
---
vCenter和vCSA 6.0.0a 的补丁已经放出来了，具体的升级内容可以查看 VMware 官方的 release note。vCSA 6.0.0a 的补丁是两个 iso 文件：

* [VMware-vCenter-Server-Appliance-6.0.0.5110-2656759-patch-TP.iso](http://kb.vmware.com/kb/2111641)
* [VMware-vCenter-Server-Appliance-6.0.0.5110-2656759-patch-FP.iso](http://kb.vmware.com/kb/2111640)

如果要打完整补丁，实际上只需要那个 FP iso 就可以了。由于 vCSA 6 取消了 5480 的管理页面，现在打补丁需要用全新的方式。

1. 在 vSphere Client 里打开 vCSA 虚拟机的控制台，登录，进【Troubleshooting Mode Options】，将 SSH 登录打开（会显示 Disable SSH 项）。
1. 挂载 patch-FP.iso 到 vCSA 的虚拟机。
1. ssh 登录 vCSA，在 vmware shell 下，输入
        software-packages install --iso
1. 等待完成
1. 重启 vCSA。

OK，这样就完了。