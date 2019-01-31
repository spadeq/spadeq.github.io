---
title: 虚拟机挂载本地 ISO 始终处于连接中的解决办法
date: 2015-07-27 12:54:15
categories:
- ITM
- Cloud
tags:
- VMware
---
在 vSphere Client 中，给某个虚拟机挂载光驱时选择本地 ISO，结果就始终处于 Connecting 状态，即便关闭虚拟机也没有用，导致无法挂载 ISO。

# 成因

导致这个问题需要两个条件：一是虚拟机光驱类型为 Client Device 并且模式为 Passthrough IDE，二是 vSphere Client 没有以管理员方式运行。

# 解决方案

关闭 vSphere Client，然后以管理员方式重新运行，这时 Connecting 状态就应该已经解除。编辑 CD/DVD 属性，令其选择 Emulate IDE。