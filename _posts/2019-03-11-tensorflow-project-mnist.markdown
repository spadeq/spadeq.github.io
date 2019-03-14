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

MNIST 是一套手写阿拉伯数字的标准识别数据，图片的类别就是阿拉伯数字的 0 ~ 9，共 10 类。图片为 28 x 28 像素的灰度图像，取值为 0 ~ 1 之间的浮点数。每幅图片就是手写 0 ~ 9 的各种形态。原始的 MNIST 数据由 60000 幅构成训练集（train），10000 幅构成测试集（test）。TensorFlow 又将训练集中抽取了 5000 幅作为验证集（validation），实际训练集为 55000 幅。

## MNIST 数据的使用

目前一般使用 `tensorflow.examples.tutorials.mnist` 包来处理 MNIST 数据，但是这种方法已经被标记为 deprecated，将来会废弃。官方推荐的方式可以参照其 [github](https://github.com/tensorflow/models/tree/master/official/mnist)。但是本文仍然保留原先的用法。

```python
from tensorflow.examples.tutorials.mnist import input_data

mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)

print(mnist.train.images.shape)
# (55000, 784)
print(mnist.train.labels.shape)
# (55000, 10)

print(mnist.validation.images.shape)
# (5000, 784)
print(mnist.validation.labels.shape)
# (5000, 10)

print(mnist.test.images.shape)
# (10000, 784)
print(mnist.test.labels.shape)
# (10000, 10)

print(mnist.train.images[0, :])
```

mnist 对象包括下列属性：

| 属性 | 内容 | 形状 |
| - | - | - |
| mnist.train.images | 训练集图像 | 55000 x 784 |
| mnist.train.labels | 训练集标签 | 55000 x 10 |
| mnist.validation.images | 验证集图像 | 5000 x 784 |
| mnist.validation.labels | 验证集标签 | 5000 x 10 |
| mnist.test.images | 测试集图像 | 10000 x 784 |
| mnist.test.labels | 测试集标签 | 10000 x 10 |

由于图像不换行保存，所以从 28 x 28 的矩阵变成了 784 维向量。

# Python 环境准备

Coding 环境可以按照[这篇博文](https://spadeq.github.io/2019/02/01/setting-tensorflow-environment.html)的内容来配置。然后为了顺利编译本文的代码，还需要在 Anaconda 中安装下列包：

* pillow