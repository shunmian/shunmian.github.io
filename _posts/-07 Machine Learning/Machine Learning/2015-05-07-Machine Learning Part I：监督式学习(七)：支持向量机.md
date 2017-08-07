---
layout: post
title: Machine Learning(一)：监督式学习 Part VII：支持向量机
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: SVM(Support Vector Machine)是分类中的一个强大的算法。相比于Logistic Regression和Netural Network，Support Vector Machine有时候可以提供更清晰和强大的方式来学习复杂的非线性函数。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 支持向量机 ##

### 1.1 支持向量机介绍 ###

机器学习的**分类**算法有以下主要三种。

1. **Logistic Regression (逻辑回归)**；

2. **Netural Network (神经网络)**：用于减少**Logistic Regression**带来多项式Feature个数的复杂度的问题。例如2次项复杂度是$O(\frac {n^2} 2)$，3次项的复杂度是$O({n^3})$；

3. **Support Vector Machine (支持向量机)**：相比于**Logistic Regression**和**Netural Network**，**Support Vector Machine**有时候可以提供更清晰和强大的方式来学习复杂的非线性函数。

> SVM：are supervised learning models with associated learning algorithms that analyze data used for classification and regression analysis.

{: .img_middle_lg}
![SVM](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/SVM.png)


#### 1.1.1 SVM成本函数 ####

对于**Logistic Regression**，它的成本函数是：

$$
\\
J(\theta) = \frac 1 m  \sum_{i=1}^m [-y^{(i)}log(h_\theta(x^{(i)}))+(1-y^{(i)})log(1-h_\theta(x^{(i)}))] +\frac \lambda {2m} \sum_{j=1}^n\theta_j^2
\\
$$

我们用另一种cost function：

{: .img_middle_mid}
![Logistic Regression_Cost Function](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/Logistic Regression_Cost Function.png)

$$

\begin{align}

\\

J(\theta) &= \frac 1 m  \sum_{i=1}^m [-y^{(i)}cost_1(\theta^Tx^{(i)})+(1-y^{(i)})cost_0(\theta^Tx^{(i)})] +\frac \lambda {2m} \sum_{j=1}^n\theta_j^2
\\
&= \frac C m \sum_{i=1}^m [-y^{(i)}cost_1(\theta^Tx^{(i)})+(1-y^{(i)})cost_0(\theta^Tx^{(i)})] +\frac 1 {2} \sum_{j=1}^n\theta_j^2 \text {, where $C = \frac 1 {\lambda}$}

\end{align}

$$



#### 1.1.2 SVM Margin ####

> SVM直观理解：增大Margin。SVM别名是Large Margin Classifier。

{: .img_middle_lg}
![Large Margin Classifier](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/Large Margin Classifier.png)

#### 1.1.3 SVM 成本函数最小值和Margin关系 ####

为获得$J_{\theta}$最小值。

$$
\begin{align}
\\
_{min}J(\theta) &= _{min}(\frac C m \sum_{i=1}^m [-y^{(i)}cost_1(\theta^Tx^{(i)})+(1-y^{(i)})cost_0(\theta^Tx^{(i)})]) +_{min}(\frac 1 {2} \sum_{j=1}^n\theta_j^2) \text {, where $C = \frac 1 {\lambda}$}
\\
&=\begin{cases}
0, &\text{if $\theta^Tx^{(i)} ≥ 1$  so $y = 1$} \\
0,&\text{if $\theta^Tx^{(i)} ≤ -1$ so $y = 0$} \\ 
\end{cases} + _{min}(\frac 1 {2} ||\theta||^2)  
\\
&\text {where $\theta^Tx^{(i)}$ is the inner product of $\theta$ and $x^{(i)}$, }
\\
&\text {$\theta$ is the vector perpendicular to the decision boundary }
\end{align}
$$

{: .img_middle_lg}
![SVM costfunction vs. margin](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/SVM costfunction vs. margin.png)

### 1.2 Kernels ###

> **Kenel(核函数)**：允许我们应用SVM做出复杂非线性的分割。

{: .img_middle_lg}
![Kernel](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/Kernel.png)


{: .img_middle_hg}
![Kernel详解](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/Kernel详解.png)

### 1.3 SVM实用建议 ###

{: .img_middle_hg}
![SVM实用建议](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/SVM实用建议.png)





## 2 作业 ##

见[这里](https://github.com/shunmian/-11-Machine-Learning)。附上一张跑分图。

{: .img_middle_lg}
![assignment6](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/assignment6.png)


## 3 总结 ##

{: .img_middle_hg}
![SVM总结](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-07/SVM总结.png)

## 4 参考资料 ##
- [《支持向量机(SVM)是什么意思》](https://www.zhihu.com/question/21094489);

- [《An Idiot's Guide to Support Vector Machines》](http://web.mit.edu/6.034/wwwbob/svm-notes-long-08.pdf);




