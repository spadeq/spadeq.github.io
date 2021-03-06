---
layout: post
title: Machine Learning 笔记
date: 2019-02-19 10:36:00
categories: 
- ML
tags:
- Machine Learning
---

> It's not who has the best algorithm that wins. It's who has the most data.  --- Andrew Ng

机器学习（Machine Learning）的含义是，通过一定的算法，使用历史数据进行训练，训练后产生模型。利用模型，可以对未来新的数据进行预测。

# Machine Learning 关键概念

ML 的关键概念是特征（feature）和标签（label）。特征是数据所具有的性质，标签是数据的类别。可以认为 label 指的就是 classification。

从分类的角度来看，历史数据就是训练集，未来新的数据就是测试集。

整个过程可以分为训练阶段和预测阶段。训练阶段是通过训练集（训练样本），输入机器学习算法，经过评估函数的评估后形成模型。预测阶段是将新的数据输入模型，得到预测。

# ML 分类

从大方向上可以分为三个类别：

## 监督学习（Supervised learning）

训练集数据已经带有标签，即已经确定了分类，并且训练集已经分类好。主要使用的手段是回归分析（regression）。

## 无监督学习（Unsupervised learning）

训练集数据没有带标签，可能也不知道最终应该分几类，需要在学习过程中自动形成分类。主要涉及的概念有聚类（clustering）、降维（dimension reduction）、异常检测（anomaly detection）。

## 强化学习（Reinforcement learning）

AlphaGo 的高大上学习方式。。以后再看。

# 统计回归

回归分析是预测输入变量与输出变量之间的关系。其中输出变量是连续的。

$$y=\beta_0+\beta_1x_1+\beta_2x_2+\cdots+\beta_nx_n+\epsilon$$

根据变量数量，可分为一元回归和多元回归。

根据算法是否线性，可以分为线性回归（最小二乘法）和非线性回归（核方法、树类方法）。

常用的算法包括：线性回归、支持向量机（SVM）、树类算法和神经网络。

插一嘴：[Kaggle](https://www.kaggle.com/)，一个可以用来做数据分析研究的网站。

# 分类

分类问题的输出变量是离散的。根据输出变量的多少，可以分为二分类和多分类。

衡量分类算法要看它的精确率、召回率。

常用的算法包括：K-means、感知机、朴素贝叶斯、决策树、逻辑斯蒂回归、支持向量机、神经网络。

# 聚类

无监督学习的一种。把相似的对象通过静态分类的方法分成不同的组别或更多的子集。其中同一子集的成员都有相似的属性。

算法：K-means、高斯混合聚类、密度聚类、层次聚类。

用途：用户画像、动植物分类等。

# 机器学习的流程

1. 数据提取（ETL）
1. 数据清洗（Feature cleaning）。必要前提，干掉脏数据，可能会耗费很多精力。
1. 特征工程（Feature engineering）。关键因素，搞分类的都知道，核心就是玩特征。
1. 训练模型（Training model）
1. 验证与优化模型（Validation）。多种手段和方法，慢慢打磨。