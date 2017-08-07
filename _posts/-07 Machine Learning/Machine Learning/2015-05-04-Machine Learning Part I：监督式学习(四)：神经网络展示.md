---
layout: post
title: Machine Learning(一)：监督式学习 Part IV：神经网络展示
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: 本文介绍神经网络。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 神经网络介绍##

## 2 神经网络详解## 

### 2.1 非线性假设 ###

如果是一个非线性的分类问题，我们需要多项式来拟合，比如选择2次项或者3次项，这个时候我们的Final Feature的个数复杂度是:

1. 2次项的复杂度是$O(\frac {n^2} 2)$，
2. 3次项的复杂度是$O({n^3})$。

也就是说不是Scalable的。

{: .img_middle_lg}
![non-linear hypothesis](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-04/non-linear hypothesis.png)


### 2.2 神经元与大脑 ###

> **神经网络**：在机器学习和认知科学领域，人工神经网络模仿人类大脑用**只有一种学习算法**通过人工神经元联结进行计算，一种非线性统计性数据建模工具。典型的神经网络具有：

1. 结构(Architecture，神经元连接的权重（weights）和神经元的激励值（activities of the neurons））

2. 激励函数（Activity Rule）

3.学习规则（Learning Rule）

科学家发现对于大脑中的某个区域，如果改变相应的输入信号，该区域可以学习处理任何信号，比如从听觉改成视觉。这改变了人们思考如何通过计算机来模仿人类的大脑，即通过单一算法而不是成百上千种算法。运行算法的单元就是神经元。

{: .img_middle_lg}
![brain one learning algorithm](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-04/brain one learning algorithm.png)

### 2.3 模型展示 ###


{: .img_middle_hg}
![what is netural network](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-04/what is netural network.png)

### 2.4 例子和直觉 ###

下面我们来看看**Netural Network**如何构建3个基本的逻辑电路，即**AND**，**OR**，**Negation** gate;以及在此基础上搭建出来的**XNOR** gate。

{: .img_middle_hg}
![XNOR Gate](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-04/XNOR Gate.png)


[Handwritten Digit Classification - Yann Lecun](https://www.youtube.com/watch?v=yxuRnBEczUU)是利用**Netural Network**识别手写数字的一个视频，是**Yann Lecun(机器学习的著名科学家)**发明的一个OCR程序。


### 2.5 多类的分类 ###

{: .img_middle_lg}
![what is netural network](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-04/Neutral Network Multiclass Classification.png)

## 3 作业 ##

见[Classification](https://github.com/shunmian/-11-Machine-Learning)。附上一张跑分图。


{: .img_middle_lg}
![assignment3](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-04/assignment3.png)

## 4 总结 ##

本文根据人类大脑神经元作为基本单元，“单一学习算法”为算法，用神经网络模拟人脑。

通常神经网络有$l$层(第1层是输入层，第n层是输出层，其他为隐藏层)，每一层有$S_j$个激发单元，激励函数是$sigmoid(A\Theta^T)$ 函数，$\Theta^T$是权重。它的universal的学习函数算法为：

$$
\begin{align}
a_1 &= x
\\
a_1 &= [ones(size(a_1,1),1) \;a_1]; z_2 = a_1\Theta_1^T; a_2 = sigmoid(z_2); 
\\
a_2 &= [ones(size(a_2,1),1) \;a_2];
z_3 = a_2\Theta_2^T;
a_3 =h_\theta(x)= sigmoid(z3);
\end{align}
$$





## 5 参考资料 ##
- [《Deep Learning》](http://deeplearning.net/);
- [《Microsoft Azure Machine Learning》](https://azure.microsoft.com/en-us/services/machine-learning/);
- [《A Visual Introduction to Machine Learning》](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/);





