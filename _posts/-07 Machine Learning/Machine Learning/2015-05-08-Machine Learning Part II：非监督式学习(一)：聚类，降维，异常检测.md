---
layout: post
title: Machine Learning(二)：非监督式学习 Part I：聚类，降维，异常检测
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: 非监督式学习是没有label的数据，但是我们还是可以从中找出某些规律，比如给数据分类(聚类)等。本文我们介绍非监督式学习的三个子问题，聚类的K-means算法，降维的PCA算法和异常检测的高斯分布算法。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 非监督式学习 ##

> **Unsupervised Learning**：is the machine learning task of inferring a function to describe hidden structure from unlabeled data。换句话说，**监督式学习**是$\lbrace (x^{(1)},y^{(1)}),  (x^{(2)},y^{(2)}), \cdots,(x^{(m)},y^{(m)}) \rbrace$中拟合；**非监督式学习**是$\lbrace x^{(1)},  x^{(2)}, \cdots, x^{(m)} \rbrace$中找分成几类以及如何分类。<br/>非监督式学习方法有：聚类(Clustering)，降维 (Dimensionality Reduction)，异常检测(Anomaly Detection)，神经网络(Neural Networks)。

本文我们着重介绍**聚类(Clustering)**和**降维 (Dimensionality Reduction)**。

### 1.1 聚类(Clustering)： K-Means算法 ###

> 聚类(Clustering)：聚类是把相似的对象通过静态分类的方法分成不同的组别或者更多的子集（subset），这样让在同一个子集中的成员对象都有相似的一些属性，常见的包括在坐标系中更加短的空间距离等。

聚类的一个最为重要的算法是**K-Means算法**。


{: .img_middle_hg}
![K-Means Algorithm](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-08/K-Means Algorithm.png)


### 1.2 降维(Dimensionality Reduction)：PCA算法 ###

> 聚类(Clustering)：在机器学习和统计学领域，降维是指在某些限定条件下，降低随机变量个数，得到一组“不相关”主变量的过程。 降维可进一步细分为特征选择和特征检测两大方法。


降维的一个最为重要的算法是**主成分分析(Principal Component Analysis,PCA)**算法。

{: .img_middle_hg}
![PCA](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-08/PCA.png)

### 1.3 异常检测 (Anormaly Detection)：高斯分布 ###

{: .img_middle_hg}
![异常检测](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-08/异常检测.png)

## 2 作业 ##

见[这里](https://github.com/shunmian/-11-Machine-Learning)。附上一张跑分图。

{: .img_middle_mid}
![assignment7](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-08/assignment7.png)


## 3 总结 ##

本文介绍了**聚类的K-means算法**，**降维的PCA算法**和**异常检测的高斯分布算法**。

1. **聚类的K-means算法**用于将unlabel的数据归类；

2. **降维的PCA算法**用于数据压缩和可视化；

3. **异常检测**用于有大量正常数据和少量非正常数据，通过高斯分布，排除非正常数据。它不同于监督式学习的分类(通过学习**大量正常**和**大量不正常**数据)。

三者都属于非监督式学习。

## 4 参考资料 ##
- [《Unsupervised Learning》](https://en.wikipedia.org/wiki/Unsupervised_learning);






