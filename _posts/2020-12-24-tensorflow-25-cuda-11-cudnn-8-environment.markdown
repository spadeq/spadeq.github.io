---
layout: post
title: TensowFlow 2.5 与 CUDA 11 和 cuDNN 8 环境配置
date: 2020-12-24 17:54:00
categories:
  - DL
tags:
  - Python
  - nVidia
  - CUDA
---

本文介绍如何在 nVidia 显卡上配置 Tensowflow 2.5 的 GPU 计算环境。目前网上很多帖子中多多少少都有弯路或者错误，实际的最短流程只要看本文即可。

## 安装最新显卡驱动

首先将 nVidia 显卡驱动更新到最新。推荐使用 GeForce Experience 软件自动更新。也可从  [nVidia 驱动网站](https://www.nvidia.com/Download/index.aspx) 上选择下载。驱动的类型，比如 GRD 或 SD，没有什么影响，可按自己喜好选择。

显卡驱动安装完毕后建议重启系统，然后在命令行中通过以下命令检查驱动版本和 CUDA 支持版本。

```powershell
PS C:\> nvidia-smi

Thu Dec 24 17:57:19 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.89       Driver Version: 460.89       CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name            TCC/WDDM | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro RTX 4000    WDDM  | 00000000:65:00.0  On |                  N/A |
| 30%   37C    P8    17W / 125W |   1614MiB /  8192MiB |     14%      Default |
+-------------------------------+----------------------+----------------------+
```

这里显示出的 CUDA 版本是该显卡支持的最高版本，而非实际本机安装的 CUDA Toolkit 版本。

## 安装 CUDA 开发包

TF 2.5 建议采用 11.0 版本的 CUDA，在 [这里](https://developer.nvidia.com/cuda-11.0-update1-download-archive?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exelocal) 下载。

双击运行安装，选择自定义方式，安装过程中不需要选第二大项（PhysX 等组件）和第三大项（驱动），只选第一大项（CUDA）。

安装完 CUDA 后，还要安装 cuDNN 插件。TF 2.5 建议采用 8.0.x 版本的 cuDNN，而且要对应 CUDA 11.0 的版本。可以在 [这里](https://developer.nvidia.com/rdp/cudnn-archive) 下载。解压 zip 包，将里面的内容对应复制到 `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.0` 目录下。

**重启系统**，否则任凭你怎么修改 PATH 都是没用的，会遇到 `cudnn64_8.dll` 相关错误。而且，如果按照上面的方式安装 cuDNN，也不需要修改任何环境变量。

重启回来在命令下执行 `nvcc -V`，有类似输出就说明安装成功了：

```text
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2020 NVIDIA Corporation
Built on Wed_Jul_22_19:09:35_Pacific_Daylight_Time_2020
Cuda compilation tools, release 11.0, V11.0.221
Build cuda_11.0_bu.relgpu_drvr445TC445_37.28845127_0
```

## 配置 Anaconda 环境

Windows 下建议使用 Anaconda 管理 python 发行版。在 [Anaconda 官网](https://www.anaconda.com/products/individual) 下载 individual 最新版（目前是 Python 3.8 64 位），以默认选项进行安装。

安装完毕后，打开 Anaconda-Navigator，创建一个新的环境，命名如 tf25，选择 Python 3.8。

在 Home 界面，切换 Application 到刚生成的 tf25，安装「CMD.exe Prompt」然后点击进入到带环境的命令行界面。

依次执行下列命令安装 tf 2.5：

* `conda install numpy matplotlib`，安装基本依赖包；
* `conda install cudatoolkit=11.0` 安装 CUDA 支持包；
* `pip install tf-nightly-gpu -i https://pypi.douban.com/simple`，安装 tf 2.5。特别注意不要像其他帖子里说的那样加上 `==2.5.0`，否则会报下面的错误。如果 tf 2.5 已经正式发布，则直接安装 `tensowflow-gpu` 即可，不需要走 nightly 通道。

```text
ERROR: Could not find a version that satisfies the requirement tf-nightly-gpu==2.5.0
ERROR: No matching distribution found for tf-nightly-gpu==2.5.0
```

## 验证

在 Anaconda 命令行里进入 python 环境，依次执行 `python`、`import tensorflow as tf` 和 `tf.config.list_physical_devices('GPU')` 命令。

```text
(tf25) C:\Users\Jacky Zhou>python
Python 3.8.5 (default, Sep  3 2020, 21:29:08) [MSC v.1916 64 bit (AMD64)] :: Anaconda, Inc. on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
2020-12-24 19:41:29.754843: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library cudart64_110.dll
>>> tf.config.list_physical_devices('GPU')
2020-12-24 19:43:58.924393: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library nvcuda.dll
2020-12-24 19:43:58.985607: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1760] Found device 0 with properties:
pciBusID: 0000:65:00.0 name: Quadro RTX 4000 computeCapability: 7.5
coreClock: 1.545GHz coreCount: 36 deviceMemorySize: 8.00GiB deviceMemoryBandwidth: 387.49GiB/s
2020-12-24 19:43:58.986009: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library cudart64_110.dll
2020-12-24 19:43:59.080881: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library cublas64_11.dll
2020-12-24 19:43:59.081357: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library cublasLt64_11.dll
2020-12-24 19:43:59.142691: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library cufft64_10.dll
2020-12-24 19:43:59.154208: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library curand64_10.dll
2020-12-24 19:43:59.261853: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library cusolver64_10.dll
2020-12-24 19:43:59.302162: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library cusparse64_11.dll
2020-12-24 19:43:59.307207: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library cudnn64_8.dll
2020-12-24 19:43:59.307760: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1902] Adding visible gpu devices: 0
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

关键几点：

* 能够正确 import tensorflow 的 package
* 能正确加载 cuda 运行时
* 正确认出 GPU 型号
* 没有报 cuDNN 的 DLL 错误
