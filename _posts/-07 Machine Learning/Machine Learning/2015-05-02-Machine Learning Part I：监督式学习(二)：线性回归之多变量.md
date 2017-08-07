---
layout: post
title: Machine Learning(一)：监督式学习 Part II：线性回归之多变量
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: 线性回归之多变量是对线性回归之单变量的范化，都属于监督式学习的回归问题。本文我们对线性回归之多变量做一个简单的介绍。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 线性回归之多变量介绍 ##

上篇笔记中我们介绍了**线性回归之单变量**，即输入变量为一个$x_1$。这篇笔记我们输入变量的个数延伸为任意多个，即$x_1,x_2,...,x_n$，该模型更能准确描述现实生活中的问题，且**线性回归之单变量**是**线性回归之多变量**的退化情况。

## 2 线性回归之多变量详解 ##

### 2.1 预测函数 ###

**线性回归之多变量**的预测函数如下：

$$
h_\theta(x) = \theta_0 + \theta_1x_1 +\theta_2x_2 + ... + \theta_nx_n
$$

将其**向量化(Vectorization)**用矩阵表示为：

$$
h_\theta(x) = [\theta_0 + \theta_1 + \theta_2 + \cdots + \theta_n]

\begin{bmatrix}
x_0 \\
x_1 \\
\vdots\\
x_n \\
\end{bmatrix}

= \mathbf{\theta^Tx}

$$

where $x_0^i = 1 \;for \; (i\in1,\cdots,m)$




### 2.2 成本函数 ###

**线性回归之多变量**的成本函数和**线性回归之单变量**一样。

$$
J(\theta) = \frac 1 {2m} \sum_{i=1}^m (h_\theta(x^{(i)}-y^{(i)})^2
$$

### 2.3 寻找最优值 ###

#### 2.3.1 梯度下降 ####

{: .img_middle_hg}
![梯度下降](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-02/data.png)



$for \; (j\in1,\cdots,n+1)$repreat until convergence:

$$
\theta_j := \theta_j - \alpha \frac 1 m \sum_{i=1}^m \lbrace[h_\theta(x^{(i)})-y^{(i)}]x_j^{(i)}\rbrace
$$

展开即$for \; (j\in1,\cdots,n+1)$(其中n是多变量的维度+1，即上图中的col feature维度+1；m是数据的维度，即上图中的row维度)

$$
\theta_0 := \theta_0 - \alpha \frac 1 m \sum_{i=1}^m \lbrace[h_\theta(x^{(i)})-y^{(i)}]x_0^{(i)}\rbrace
\\
\theta_1 := \theta_1 - \alpha \frac 1 m \sum_{i=1}^m \lbrace[h_\theta(x^{(i)})-y^{(i)}]x_1^{(i)}\rbrace
\\
\vdots
\\
\theta_n := \theta_n - \alpha \frac 1 m \sum_{i=1}^m \lbrace[h_\theta(x^{(i)})-y^{(i)}]x_n^{(i)}\rbrace
$$



#### 2.3.2 代数解 ####

### 2.4 实际情况 ###

#### 2.4.1 Feature Scaling ####

所有的变量在一个scale上可以确保梯度下降可以快速找到最优值。

将所有输入变量x转换成如下形式：

$$
x_i := \frac {x_i-\mu_i} {S_i}
$$

where $\mu_i$是所有$x_i$的平均值，${S_i}$是$x_i$的最大最小值之差。

{: .img_middle_hg}
![feature scaling](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-02/feature scaling.png)


#### 2.4.2 Learning Rate ####

如何确定梯度下降程序运行正确。主要看$J_\theta$和No. of iteration的关系。

{: .img_middle_hg}
![feature scaling](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-02/debug.png)

### 2.5 多项式回归 ###

在**线性回归之单变量**和**线性回归之多变量**之间有一种转换情况，即**线性回归之单变量**不能很好拟合数据，这个时候就需要用单变量的多项式来通过**线性回归之伪多变量**来拟合，例如下图，单独size一个变量的线性拟合不能准确描述价格的变化，因此用size的**多项式**通过
**线性回归之多变量**来拟合。由于$size$，$size^2$，$size^3$的scale不在同一个数量级上，因此需要通过[Feature Scaling](#feature-scaling)来优化。

{: .img_middle_hg}
![feature scaling](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-02/polynomial regression.png)

### 2.6 Normal Equation ###

除了用梯度下降来接近$minJ_(\theta)$，还可以通过代数方法求导获得。

$$
J(\theta_0,\theta_1,\cdots,\theta_j) = \frac 1 {2m} \sum_{i=1}^m (h_\theta(x^{(i)})-y^{(i)})^2
$$

$$
\frac \partial {\partial \theta_j} J(\theta) = ... = 0
$$

for every $j$, solve for $\theta_0,\theta_1,\cdots,\theta_j$

{: .img_middle_hg}
![feature scaling](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-02/standard equation.png)

梯度下降和标准方程都是两个求解$\theta$来获取最低$J(\theta)$的方法，但是两者各有优缺点，见下图。

{: .img_middle_hg}
![feature scaling](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-02/梯度下降vs标准方程.png)

## 3 Matlab and Octave 入门 ##

见[这里]({{ site.baseurl}}/machine%20learning/2015/05/15/Machine-Learning(A1)-MatLab和Octave基础.html)。

## 4 作业 ##

见[Linear Regression](https://github.com/shunmian/-11-Machine-Learning)。

附上一张跑分图。

{: .img_middle_lg}
![assignment1](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-02/assignment1.png)

## 5 总结 ##

本文将**线性回归之单变量**泛化成**线性回归之多变量**的情况，并且对其预测函数，成本函数，以及如何获取最低成本值的方法(梯度下降，标准方程)进行讲解。在实际过程中，可以通过Feature Scaling来加速学习速率；并且通过$J_\theta$和No. of iteration的关系来debug程序是否正确(正确的程序每多一步，都会降低$J_\theta$)。其中单变量如果线性回归不准确，可以将其转换成多项式再通过**线性回归之多变量**和Feature Scaling来求解。



## 6 参考资料 ##
- [《Deep Learning》](http://deeplearning.net/);
- [《Microsoft Azure Machine Learning》](https://azure.microsoft.com/en-us/services/machine-learning/);
- [《A Visual Introduction to Machine Learning》](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/);




