---
layout: post
title: 解决 Windows 功能更新 0x80242016 报错失败
date: 2019-03-24 10:27:00
categories: 
- Windows
tags:
- Windows
- Update
---

昨天更新 Windows Insider 18362 的时候一直失败，反复安装后都还是原先的版本。从 Update 设置中查看到错误代码为 0x80242016，解决方法如下：

* 以管理员权限执行 `net stop wuauserv`；
* 将 C:\\Windows\\SoftwareDistribution 目录删除或者改名；
* 再以管理员权限执行 `net start wuauserv`；
* 重新走更新流程（重新下载）。

其中管理员权限执行可以在 Win + R 运行窗口中，按住 Ctrl + Shift 来方便地实现。