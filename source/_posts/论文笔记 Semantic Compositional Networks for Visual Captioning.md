---
title: 论文笔记 Semantic Compositional Networks for Visual Captioning
date: 2018-08-18 20:58:18
tags:
mathjax: true
---

### 简介

​	该论文提出了语义组合网络(Semantic Compositional Network, SCN)，其有效利用语义概念（标签）来达到效果比较好的图文生成。

![](https://github.com/McGinn7/mcginn7.github.io/blob/master/img/temp/1534597565116.png?raw=true)





### Semantic compositional networks

![](https://github.com/McGinn7/mcginn7.github.io/blob/master/img/temp/1534597459535.png?raw=true)

- **模型基础**

  使用CNN提取图像特征，RNN作文字生成。

  文字生成的概率公式：
  $$
  p(\bold X | \bold I) = \prod _{t=1}^Tp(x_t|x_0, \dots , x_{t-1}, v)
  $$
   $ \bold X = (x_1, \dots , x_T)$ 表示文字序列，$v$ 为提取的图像特征。

	LSTM的转换函数：
$$
h_t = \sigma(Wx_{t-1}+Uh_{t-1}+\mathbb{1}(t=1)\cdot Cv)
$$
​	图像特征仅在开始输入**一次**。



- **语义概念检测 **

  作者将语义标签检测作为多标签分类问题。

  首先先从训练集的文字说明中提取常见的 $K \approx 1000$个单词作为分类标签 $y_i = [y_{i1}，\dots，y_{iK}] \in \{0, 1\}^K$。

  标签$s_i$使用MLP来预测(Ps：这里可能是在CNN的基础上加入MLP)，
  $$
  s_i = \sigma(MLP(v_i))
  $$
  $s_i$表示每个标签的概率，也可以理解为权重。

  优化目标函数：
  $$
  \frac 1N\sum_{i=1}^N \sum_{k=1}^K {(y_{ik}\log s_{ik}+(1-y_{ik}\log(1-s_{ik})))}
  $$

- **SCN-RNN**

  这一步就是将语义标签嵌入到RNN中。 

  ![](https://github.com/McGinn7/mcginn7.github.io/blob/master/img/temp/1534596069120.png?raw=true)

  嵌入相关公式：
  $$
  \hat x_{t-1}=W_bs\bigodot W_cx_{t-1} \\
  \hat h_{t-1} = U_bs\bigodot U_ch_{t-1} \\
  z=\mathbb{1}(t=1)\cdot Cv \\
  h_t = \sigma(W_a\hat x_{t-1} + U_a\hat h_{t-1} + z)
  $$

- **视频文字生成(video caption)**

  视频的图像特征包括两部分：均值池化2D CNN提取的图像特征和3D CNN提取的特征，两个特征连接起来作为视频的图像特征。


### 结果

- 在数据集COCO和Youtube2Text的各个评估指标全面提升。

![](https://github.com/McGinn7/mcginn7.github.io/blob/master/img/temp/1534596916958.png?raw=true)

![](https://github.com/McGinn7/mcginn7.github.io/blob/master/img/temp/1534596858609.png?raw=true)

