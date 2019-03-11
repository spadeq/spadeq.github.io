---
layout: post
title: Tensorflow 项目实战之 MNIST
date: 2019-03-11 18:56:00
categories: 
- Tensorflow
tags:
- Machine Learning
- Tensorflow
- Python
- MNIST
---

本文为学习何之源先生《21 个项目玩转深度学习》之笔记，这是一本非常好的 Tensowflow 学习书籍，如果您对本文的内容感兴趣，请多多支持原作者何先生！[原作链接](http://www.broadview.com.cn/book/5490)

# MNIST 数据

MNIST 是一套手写阿拉伯数字的标准识别数据，图片的类别就是阿拉伯数字的 0 ~ 9，共 10 类。

# Python 环境准备

Coding 环境可以按照[这篇博文](https://spadeq.github.io/2019/02/01/setting-tensorflow-environment.html)的内容来配置。然后为了顺利编译本文的代码，还需要在 Anaconda 中安装下列包：

* pillow