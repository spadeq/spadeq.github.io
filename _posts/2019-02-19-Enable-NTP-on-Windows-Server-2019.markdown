---
layout: post
title: 启用 Windows Server 2019 的 NTP 服务
date: 2019-02-19 12:01:00
categories: 
- Windows
tags:
- Windows Server
- NTP
---

网上流传的改注册表大法可能不好用，这里提供一个使用组策略的方法。

1. 运行 `gpedit.msc` 打开组策略管理器，展开「Computer Configuration」->「Administrative Templates」->「System」->「Windows Time Service」；
2. 进入「Time Providers」，点开「Enable Windows NTP Server」，选中 Enable，OK；
3. 回到上一层，点开「Global Configuration Settings」，选中 Enable，然后把下面「AnnounceFlags」改成 5，OK；
4. 运行 `services.msc` 打开服务管理器，找到「Windows Time Properties」，将启动类型设为自动，并且重启服务。
5. 找一台 Linux 主机用 `ntpdate` 命令验证看是否成功。