---
layout: post
title: Machine Learning(一)：监督式学习 Part III：分类
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: 本文我们来了解下监督式学习的分类问题。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Logistic Regression介绍##

现在我们从**回归**问题转到**分类**问题。别被**Logistic Regression**这个词迷惑了(之所以叫这个名字是由于历史问题，见[这里](http://papers.tinbergen.nl/02119.pdf))，它是**分类**问题，而不是**回归问题**。

## 2 Logistic Regression详解##

不像输出y是一个连续的函数，**Logistic Regression**的输出是离散数据，即$y\in \lbrace 0,1, \cdots, n \rbrace $。

### 2.1 二分类 ###

我们现在首先考虑最简单的情况，即即$y\in \lbrace 0,1 \rbrace $。我们称之为**二分类**问题。

有人会说这好办，我们只要给原来的线性回归的输出map到一个$\lbrace 0,1, \cdots, n \rbrace$值即可，需要做的是找一个map的边界。但是这会过度简化模型导致输出不准确，请看下图例子。

{: .img_middle_lg}
![线性回归map到二分类](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/线性回归map到二分类.png)

新加一对**Training Data**就会使得原来的预测函数不准确。

那么该如何确定预测函数的形式呢，请往下接着看。

#### 2.1.1 预测函数 ####

$$
g(z) = \frac 1 {1+ e^{-z}}
$$

其中$g(z)$的图形如下所示，其将任何实数转换成$\lbrace 0,1\rbrace$之间的值。g(z)被称之为双弯曲函数(Sigmoid Function)或者逻辑函数(Logistic Function)。

{: .img_middle_lg}
![logistic function](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/logistic function.png)

我们将线性回归的预测函数$h_\theta(x)$代入到$z$：

$$
h_\theta(x) = g(\theta^Tx) = \frac 1 {1+ e^{-\theta^Tx}}
$$

那么该如何解读$h_\theta(x)$的函数呢？如果$h_\theta(x)=0.7=70\%$，就意味着有$70\%$的概率我们的输出是1，即：

$$
h_\theta(x) = p(y=1 |x;\theta) = 1 -  p(y=0 |x;\theta)
$$

那么我们来看下决定边界(Decision Bouhdary)：

$$
z = 0
$$

$$
h_\theta(x) ≥ 0 ⇒ y= 1
$$

$$
h_\theta(x) < 0 ⇒ y= 0
$$

> 决定边界(Decision Bouhdary)：是分开$y=0$和$y=1$的边界。

例如：

$$
\theta=

\begin{bmatrix}
5 \\
-1 \\
0 \\
\end{bmatrix}
\\

$$

当 $y=1$，即：

$$
h_\theta(x) = (\theta_0 + \theta_1x1 + \theta_2x2) ≥ 0 
\\
5+(-1)x_1+0x_2 ≥ 0
\\
x_1≤ 5
$$

在这个例子中，决定边界是$x_1=5$的直线，直线左边是$y=1$，右边是$y=0$。
当然决定边界不一定是直线，也可以是二次曲线，比如圆，抛物线等。

#### 2.1.2 成本函数 ####

如果我们用同样的线性回归的成本函数来描述分类问题，会导致成本函数有很多局部最低点。换句话说，该成本函数将不是一个凸函数(Convext Function),，如下图

Convex vs Non-convex Function

{: .img_middle_lg}
![Convex vs Non-convex Function](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/Convex vs Non-convex Function.png)


##### 2.1.2.1 复杂成本函数 ##### 

我们将成本函数重新表示成下面形式。

$$
J(\theta) = \frac 1 {m} \sum_{i=1}^m cost(h_\theta(x^{(i)}),y^{(i)})  
$$

对于线性回归，

$$
cost(h_\theta(x^{(i)}),y^{(i)})  = \frac 1 2(h_\theta(x^{(i)})-y^{(i)})^2
$$

对于分类，

$$
cost(h_\theta,y) =
\begin{cases}
-log(h_\theta(x)), &\text{if $y = 1$} \\
-log(1-h_\theta(x)), &\text{if $y = 0$} \\
\end{cases}
$$


{: .img_middle_lg}
![Classification Cost Function.png](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/Classification Cost Function.png)

##### 2.1.2.2 简单成本函数 ##### 

我们可以将上面**复杂成本函数**简化成下面**简单成本函数**的形式，
即$y=0$和$y=1$和**复杂成本函数**一样。

$$
cost(h_\theta,y) =-ylog(h_\theta(x))+(1-y)log(1-h_\theta(x))
$$

#### 2.1.3 梯度下降 ####

**分类**结合梯度下降算法如下：



$$
\begin{align}
\text{repreat until convergence:}
\\
\theta_j :& = \theta_j - \alpha \frac \partial {\partial \theta}J(\theta)
\\
\text{where:}
\\
J(\theta)&=\frac 1 m  \sum_{i=1}^m cost(h_\theta(x^{(i)})-y^{(i)}) 
\\
& =\frac 1 m  \sum_{i=1}^m [-ylog(h_\theta(x))+(1-y)log(1-h_\theta(x))]
\\
\alpha \frac \partial {\partial \theta}J(\theta)&=\alpha \frac 1 m \sum_{i=1}^m \lbrace[h_\theta(x^{(i)})-y^{(i)}]x_j^{(i)}\rbrace
\\
h_\theta(x)&= \frac 1 {1+e^{-\theta^Tx}}
\end{align}
$$

#### 2.1.4 高级优化 ####


除了**梯度下降**算法外，还有其他算法可以让我们逐步接近$J_{min}(\theta)$，比如下面这三种方法：

1. [Conjudated gradient](https://en.wikipedia.org/wiki/Conjugate_gradient_method)；

2. [BFGS(Broyden–Fletcher–Goldfarb–Shanno)](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm)；


3. [L-BFGS(Broyden–Fletcher–Goldfarb–Shanno using using a limited amount of computer memory)](https://en.wikipedia.org/wiki/Limited-memory_BFGS)；

这三种方法比**梯度下降**的优势是可以更快接近$J_{min}(\theta)$且不需要人工选择$\alpha$的值；缺点是实现起来稍显复杂。我们如果要用，可以直接用octave或matlab里的库。

下面是**matlab和Octave**的**fminunc**函数，用于计算局部最小值。

> **fminunc函数**：第一个参数是需要计算局部最小值的函数$J$，第二个参数是初始的该函数值$θ$，第三个函数是$options$(梯度开启，最多循环次数)。其中函数$J$返回函数值和梯度。


{: .img_middle_lg}
![fminunc](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/fminunc.png)

### 2.2 多分类 ###

**二分类**的问题比较简单，但是对于结果是多种类的**多分类**问题，我们该如何处理呢？比如天气的分类是Sunny，Cloudy，Rainny，Snowy等。

>**多分类**：可以简化成二分类。

{: .img_middle_lg}
![多分类vs二分类](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/多分类vs二分类.png)

**多分类**的问题可以简化成多个**二分类**问题，如上图所示。用$y=0$(该类)和$y=1$(非该类)应用到每一个class上，分别求出他们的预测函数。

## 3 正则化 ##



### 3.1 拟合问题 ###

在**监督式学习**的**回归**和**分类**中，都会碰到**Under Fitting (High Bias，太少Feature)**和**Over Fitting (High Variance，太多Feature)**的问题，见下图。


{: .img_middle_lg}
![拟合问题](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/拟合问题.png)

> **过度拟合(Over Fitting)**：使用了过多的参数，虽然完美地拟合到已有数据，但是对于未来的预测不准确。

通常来说有两种解决**过度拟合(Over Fitting)**的方法：

1. 减少Feature数量，通过人工筛选或者筛选算法(后面的笔记会介绍)；

2. **正则化(Regularization)**，保持所有Feature，但是减小Feature的权重，即$\theta_(j)$。这当我们有很多略有用的Feature时解决的效果很好。


### 3.2 成本函数 ###

那么如何应用正则化，它对原来的cost fucntion$J_{\theta}$的影响又是什么呢，我们可以从下面这个例子来用直觉理解。

当预测函数是4个Feature时，如果给成本函数加上$1000\theta^3+1000\theta^4$，可以看到为了使$J_{\theta}$为0，$\theta_3$和$\theta_4$必须要比原来没有这两项要小得多，从而降低$\theta_3$和$\theta_4$的权重。

$$
h_\theta(x) = \theta_0 + \theta_1x + \theta_2x^2 + \theta_3x^3 + \theta_4x^4   
$$

$$
J(\theta) = \frac 1 {2m} \sum_{i=1}^m (h_\theta(x^{(i)})-y^{(i)})^2 + 1000x^3+1000x^4
$$

{: .img_middle_hg}
![regularization](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/regularization.png)

有时候我们为了让预测函数更平滑，给所有的Feature都加上这么一项，即：

$$
J(\theta) = \frac 1 {2m} \sum_{i=1}^m (h_\theta(x^{(i)})-y^{(i)})^2 + \lambda \sum_{i=1}^m\theta_i^2
\\
\text {where $\lambda$ is called regularization parameter}
$$

但是若$\lambda$取值过大，有可能会将**过度拟合**削弱为**刚好拟合**并且进一步变成**不足拟合**，导致$h_\theta(x)= \theta_0$，而其他$\theta_i≈0$。

### 3.3 正则化应用 ###

#### 3.3.1 正则化线性回归 ####

##### 3.3.1.1 梯度下降 #####

$$
\begin{align}
\text{repreat until convergence:}
\\
\theta_0 : &= \theta_0 - \alpha \frac 1 m \sum_{i=1}^m [h_\theta(x^{(i)})-y^{(i)}]x_0^{(i)}
\\
\theta_j : &= \theta_j - \lbrace \alpha \frac 1 m \sum_{i=1}^m [h_\theta(x^{(i)})-y^{(i)}]x_j^{(i)} + \frac \lambda m \theta_j\rbrace, \text{$\theta_j \in \lbrace1,2,\cdots,n \rbrace $}
\\
&=\theta_j(1-\alpha \frac \lambda m) - \alpha \frac 1 m \sum_{i=1}^m [h_\theta(x^{(i)})-y^{(i)}]x_j^{(i)}, \text{$\theta_j \in \lbrace1,2,\cdots,n \rbrace $}\end{align}
$$

如何理解上面的方程呢。在任何情况下$(1-\alpha \frac \lambda m)<1$都成立，直觉上可以理解为正则化的$\lambda$使得每一步的$\theta_j$在一次更新过程中都相应地减小为原来的$(1-\alpha \frac \lambda m)$。

##需要注意的是$theta_0$不能同时被正则化，否则所有$theta$都被正则化就相当于没有正则化。##

##### 3.3.1.2 标准方程 #####

我们在[第二篇笔记]({{ site.baseurl}}/machine%20learning/2015/05/02/Machine-Learning(二)-线性回归之多变量.html#normal-equation)中提到过可以由标准方程来求出预测函数，即：

$$
\begin{align}
\theta & = (X^TX)^{-1}X^Ty
\\
\text{where:}
X &= \begin{bmatrix}x^{(1)}\\x^{(2)}\\ \vdots\\ x^{(m)} \end{bmatrix},x^{(i)} = \begin{bmatrix}x^{(1)}_i\\x^{(2)}_i\\ \vdots\\ x^{(n)}_i \end{bmatrix},y=\begin{bmatrix}y^{(1)}\\y^{(2)}\\ \vdots\\ y^{(n)} \end{bmatrix}
\end{align}
$$

但是有两种情况下$(X^TX)$是退化矩阵(不可逆的)，1是$m≤n$，2是$n$有重复(比如一个是面积 in $m^2$，一个是 in $feet^2$)。但是正则化可以让第一种$m≤n$退化矩阵变为可逆矩阵，如下：


$$
\begin{align}

\theta & = (X^TX + \lambda L)^{-1}X^Ty
\\
\text{where:}
X &= \begin{bmatrix}x^{(1)}\\x^{(2)}\\ \vdots\\ x^{(m)} \end{bmatrix},x^{(i)} = \begin{bmatrix}x^{(1)}_i\\x^{(2)}_i\\ \vdots\\ x^{(n)}_i \end{bmatrix},y=\begin{bmatrix}y^{(1)}\\y^{(2)}\\ \vdots\\ y^{(n)} \end{bmatrix}
\\
L &= \begin{bmatrix}
     0 & 0 & 0 & \cdots & 0 \\
     0 & 1 & 0& \cdots & 0 \\
     0 & 0 & 1& \cdots & 0 \\
     \vdots & \vdots &\vdots & \cdots & \vdots \\
     0 & 0 &0& \cdots & 1 \\
\end{bmatrix}
\end{align}
$$




#### 3.3.2 正则化逻辑回归 ####

{: .img_middle_lg}
![regluarization classification cost function](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/regluarization classification cost function.png)

##### 3.3.2.1 梯度下降 #####

和[回归](#section-12)一样。


## 4 作业 ##

见[Classification](https://github.com/shunmian/-11-Machine-Learning)。

附上一张跑分图。


{: .img_middle_lg}
![assignment2](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/assignment2.png)

## 5 总结 ##


{: .img_middle_hg}
![supervised learning summary.jpg](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-03/supervised learning summary.jpg)

## 6 参考资料 ##

- [《Deep Learning》](http://deeplearning.net/);
- [《Microsoft Azure Machine Learning》](https://azure.microsoft.com/en-us/services/machine-learning/);
- [《A Visual Introduction to Machine Learning》](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/);





