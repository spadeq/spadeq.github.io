---
layout: post
title: 在 nVidia 显卡设备上安装 Ubuntu 20.04
date: 2020-06-07 16:42:00
categories:
  - OS
tags:
  - Ubuntu
  - nVidia
---

本文介绍如何在具有 nVidia 显卡的设备上安装 Ubuntu 20.04。

## 安装程序

引导安装界面，可以选择 Safe Graphics Mode，也可以在普通引导项上按 e，然后在 quiet splash 后加上 `nouveau.modeset=0`，按 F10 进入安装界面。

安装过程中，选择安装第三方驱动，如果启用了 Secure Boot，还需要提供一个安全密码。

在创建用户的地方，一定注意**不要选择自动登录**，否则会陷入登录无限循环。

## 进入系统

如果可以正常进入系统，恭喜，后面的内容与你无关了。

但是经常会出现一种情况，就是在登录界面循环，表现为输入登录密码后又回到登录界面，那多半就是显卡驱动有问题，必须手动解决。

### 禁用 nouveau 驱动

按 Ctrl + Alt + F2，进入命令行，登录。操作过程中可能会浮现出桌面，不用管，继续输入命令。

创建 `/etc/modprobe.d/blacklist-nouveau.conf`：

```conf
blacklist nouveau
options nouveau modeset=0
```

然后执行 `sudo update-initramfs -u`。

### 卸载 nVidia 驱动

```bash
sudo apt-get remove --purge nvidia*
sudo shutdown -r now
```

### 重新安装驱动

这个时候重新启动应该就可以进入桌面了。先配置好网络，然后执行以下命令更新系统到最新：

```bash
sudo apt update
sudo apt upgrade
sudo shutdown -r now
```

重启进入系统后，打开「Software & Updates」，选择「Additional Drivers」，在其中的 NVIDIA 显卡条目中，选择带有「proprietary,tested」的项目（通常是第一项），点击「Apply Changes」，等待安装完毕后重新启动系统。

如果重新启动后还是进入了登录死循环，则只能安装官方驱动。

### 安装官方驱动

在 [nVidia 驱动网站](https://www.nvidia.com/Download/index.aspx?lang=en-us) 上找到自己显卡对应的驱动并下载，文件名类似于「NVIDIA-Linux-x86_64-440.82.run」。

按 Ctrl + Alt + F2，进入命令行，执行：

```bash
sudo apt-get remove --purge nvidia*
sudo shutdown -r now
```

重启完成后安装驱动：

```bash
chmod +x NVIDIA*.run
sudo ./NVIDIA*.run
```

* 提示有更好的安装方式，选择「Continue installation」；
* DKMS 选择 Yes；
* 32 位兼容库，选择 Yes；
* libglvnd，选择 Install and overwrite；
* X 配置，选择 Yes；
* 最后 OK，重启。

## 关闭自动登录

如果以上做法还是不行，说明在安装过程中可能开启了自动登录。

