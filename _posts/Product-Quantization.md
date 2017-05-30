---
title: 近似最近邻之乘积量化（Product Quantization）<br>
date: 2017-03-13 11:39:49
tags: 
- 乘积量化 
- Product Quantization 
- PQ
description: product quantizer是由Herve Jegou等人2011年在IEEE上发表的论文《Product Quantization for Nearest Neighbor Search》中提出来的。
---
# 简介
 <p>product quantizer是由Herve Jegou等人2011年在IEEE上发表的论文《__Product Quantization for Nearest Neighbor Search__》中提出来的。它的提出是为了在内存和效率之间求得一个平衡，既保证图像索引结构需要的内存足够，又使得检索质量和速度比较好。对于任何基于固定维数特征的事物，它可以应用到其索引结构的建立及检索上。它属于**ANN（approximate nearest neighbor)**算法。与它相关的算法有E2LSH(Euclidean Locality-Sensitive Hashing),KD-trees,K-means。</p> 
# 主要思想
**Product Quantizer**翻译过来是乘积量化器，包括过程：<br>
- 将原始数据向量分解成低维的子向量段<br>
- 对子向量段分别量化<br>
- 原始的向量可用子向量段的索引的笛卡尔积进行表示<br>
- 向量间的距离，可以用表示的值进行估计<br><br>
***
现在的方法如KD-tree对于高维数据并不是很有效，甚至已经退化为穷尽搜索了。<br>
解决的办法：近似最近邻搜索<br>
**1.** 对原始向量直接处理，*randomized KD-tree*和_层次k均值_(hierarchical k-means)（这两个方法都被**FLANN**(fast library for approximate nearest neighbor search)采用）<br>
**2.** binary coding,如*SH*(spectral hashing)，*RBM*(restricted Boltzmann machine),*boosting*和*LSH*等方法，它们对原始的特征向量进行二值压缩。<br><br>
乘积量化（PQ）属于第一类。主要有两个优点：<br>
1. 提出搜索结果的距离选择多样，而Binary coding的方法结果选择的距离可选择性很少。<br>
2. 结果能对向量间的L2度量进行估计，这在eps近邻搜索中很有用。<br>

![ADC&SDC](/Product-Quantization/ADC&SDC.png)
Symmetric distance computation(SDC):
$\hat d(x,y) = d(q(x),q(y))	= \sqrt{\sum_jd(q(x),q(y))^2}$
Asymmetric distance computation(ADC):
$\hat d(x,y) = d(x,q(y)) = \sqrt{\sum_jd(u_j(x),q_j(u_j(y))^2}$

**距离估计**（distance approximation）
Asymmetric distance computation(ADC):
$\hat e(x,y) = \hat d(x,y)^2 + \sum_j\xi_j(x) + \sum_k\xi_k(y)$
**距离估计偏差**（distance approximate error）
![dae](/Product-Quantization/dae.png)
从上图中可见，整体距离估计偏小于真实距离，而其中SDC比ADC更偏小。所以在乘积量化中采取ADC进行距离估计。

解决**穷尽搜索**的问题：
* **倒排文件**（inverted file system），索引相同的点组成一个队列。首先学习一个字典（codebook），进行*粗分类*，通过*k-means*方法，进行初步量化，建立倒排文件。$\Longrightarrow$ "Video Google"
* 计算**残差向量** Product Quantization $\Longrightarrow$ $r(y) = y -q_c(y)$ 
$\ddot y = q_c(y) + q_p(y - q_c(y))$

**PQ实验结构**
![inverted_file](/Product-Quantization/inverted_file.png)
*索引算法：*
**Indexing** a vector $y$ proceeds as follows: 
1. quantize $y$ to $q_c(y)$ 
2. compute the residual $r(y) = y - q_c(y)$
3. quantize $r(y)$ to $q_p(r(y))$,which,for the product quantizer,amounts to assigning $u_j(y)$ to $q_j(u_j(y))$,for $j = 1 \cdots m$.
4. add a new entry to the inverted list corresponding to $q_c(y)$. It contains the vector (or image) identifier and the binary code(the product quantizer's indexes).




符号解释：
$q_c$ $\Longrightarrow$ 稀疏量化器(coarse quantizer)
$q_p$ $\Longrightarrow$ 乘积量化器(product quantization)
$L_i$ $\Longrightarrow$ 倒排表（inverted list)
$x$ $\Longrightarrow$ 查询向量
$y$ $\Longrightarrow$ 数据库向量，从中查找出最近
$\ddot y$ $\Longrightarrow$ 向量y的评估值