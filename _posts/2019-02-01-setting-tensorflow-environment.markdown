---
layout: post
title: 设置 Tensorflow 基本环境
date: 2019-02-01 16:07:00
updated: 2019-03-04 21:23:00
categories:
  - ML
  - Tensorflow
tags:
  - Machine Learning
  - Tensorflow
  - Anaconda
  - Python
---

本文介绍使用 Anaconda + Visual Studio Code 搭建 Tensorflow 的基本开发环境。

<!-- more -->

## 安装基本软件

### Anaconda

从 [Anaconda 官网](https://www.anaconda.com/download/) 下载 Python 3.7 版的 Anaconda 图形界面安装包并安装。

非官方源现在基本上都停了，老老实实用官方的慢源吧。。

~~安装完成后切换清华源。打开 conda 命令提示符，然后执行：~~

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
```

### Visual Studio Code

从 [VSC 官网](https://code.visualstudio.com/Download) 下载对应操作系统的版本，Windows 推荐使用 64 位 User Installer。完毕后，安装以下扩展插件：

- Python（就在欢迎页上）
- Anaconda Extension Pack

## 配置 Anaconda 的 Tensorflow 包

~~打开 Anaconda Navigator，左边选 Environment，然后点击中间列下方的 Create。~~

~~在弹出对话框中，为环境起名（比如 tensor），勾选 **Python 3.6**，点击创建。~~

现在 tensorflow 包已经支持 python 3.7，所以不需要额外创建 Python 3.6 环境了。当然，如果你希望拥有一个独立的 tensorflow 环境，也可以按照上面的操作来新建一个环境，只是 Python 版本直接勾选最新的 3.7 即可。

右边栏选择 All，然后搜索 tensorflow。如果你有一块 nVidia 独立选卡，勾选 tensorflow-gpu，Apply，Apply。如果没有 nVidia 显卡，就只能勾选 tensorflow，不能使用 GPU 加速。

## 配置 VSC 的编译环境

在 Anaconda Navigator 选到 Home，确定 Applicaions on 选的是刚建的环境，然后点 VS Code 下面的 Launch 按钮。这一步的作用是让 Anaconda 协助配置 VSC。

~~首先将 shell 改成 cmd。选择 File -> Preferences -> Settings，输入 powershell，然后把 Shell 改成 `C:\WINDOWS\System32\cmd.exe`。~~

首先打开 Terminal，如果用的是 PowerShell，请执行下面的命令，启用本地 PS 脚本执行权限：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

打开 VSC，打开一个目录（可以新建一个空目录），然后新建一个文件，保存为 .py 扩展名（名称无所谓）。

提示 Linter pylint 未安装，点 Install，上面选 Install with conda。

~~着会提示 PowerShell 不支持，建议改为 Command Prompt，接受。~~

再次 Install，可能还需要回答 y。这时候最好关闭重启 VSC。另外，VSC 的 Python 运行时也确定了（左下角可以看到）。

菜单选择 File - Preference，找到 jedi，将其改为 False，Reload 之后就可以自动安装微软的 Language Server，大大加强 IntelliSense 的功能。

键入以下的代码：

```python
import tensorflow as tf
print(tf.__version__)
```

然后按下 Ctrl + F5 运行，如果能正确检测到显卡，并输出正确的 tensorflow 版本，那么环境就配置成功。例如我的显示如下：

```txt
    2019-03-04 21:25:11.154549: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1115] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 19376 MB memory) -> physical GPU (device: 0, name: TITAN RTX, pci bus id: 0000:01:00.0, compute capability: 7.5)
```
