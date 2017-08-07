---
layout: post
title: Machine Learning(一)：监督式学习 Part V：神经网络学习
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: 本文介绍神经网络求解$\Theta$的算法：Back Propagation Algorithm.

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 神经网络学习 ##

### 1.1 成本函数 ###

我们先来规定几个变量：

$$
\begin{align}
L &= \text {total number of layers in the network;}
\\
s_l &= \text {number of units (not counting bias unit) in layer l;}
\\
K &= \text {number of output units/classes;}
\end{align}
$$

对于**Logistic Regression**二分类问题，，其成本函数$J(\theta)$在正则化条件下如下：

$$
J(\theta) =-\frac 1 m  \sum_{i=1}^m [y^{(i)}log(h_\theta(x^{(i)}))+(1-y^{(i)})log(1-h_\theta(x^{(i)}))] + \frac \lambda {2m}  \sum_{j=1}^n \theta_j^2
$$


对于**Netural Network**多分类问题，其成本函数$J(\theta)$在正则化条件下可以表示成如下形式：

回顾一下在**Neutral Network**中，我们的output layer 可能有很多nodes。我们将其第$k$个node表示成$h_\theta(x)_k$

$$
J(\Theta) =-\frac 1 m  \sum_{i=1}^m \sum_{k=1}^K [y^{(i)}log(h_\theta(x^{(i)}))+(1-y^{(i)})log(1-h_\theta(x^{(i)}))] +  \frac \lambda {2m}  \sum_{l=1}^{L-1} \sum_{i=1}^{s_l} \sum_{j=1}^{s_l+1} (\Theta_{j,i}^l)^2
$$

其中：

1. double sum 表示对output layer每一行logistic regression costs的累加；

2. triple sum 表示将entire network中的$\Theta^2$累加；

### 1.2 反向传播算法 ###

> **Backpropagation**： an abbreviation for "backward propagation of errors", is a common method of training artificial neural networks used in conjunction with an optimization method such as gradient descent. The method calculates the gradient of a loss function with respect to all the weights in the network.

**Backpropagation(反向传播)**是**Netural Network**中用来获取
$
\frac {\partial J(\Theta)} {\partial{\Theta_{i,j}^{(l)}}} 
$，类似于$
\frac {\partial J(\theta)} {\partial{\theta_{j}}} = \frac 1 m \sum_{i=1}^m \lbrace[h_\theta(x^{(i)})-y^{(i)}]x^{(i)}\rbrace 
$在**Linear Regression**和**Logistic Regression**中的作用。


#### 1.2.1 反向传播算法介绍 ####

$$
\begin{align}
&\text {Given training set $\lbrace (x^{(1)},y^{(1)}),\cdots,(x^{(m)},y^{(m)}) \rbrace$;}
\\
&\text {Set $\Delta_{i,j}^{(l)} :=0$ for all $(l,i,j)$};
\\
&\text {For training example t = 1 to m:}
\\ 
&\text {$\qquad$Set $a^{(1)}$ := $x^{(t)}$,}
\\ 
&\text {$\qquad$Perform forward propagation to compute $a^{(1)}$ for l =$2,3,\cdots,L$,}
\\ 
&\text {$\qquad$Using $y^{(t)}$, compute $\delta^{(L)} = a^{(L)}-y^{(t)}$(difference between predicted and real value in layer L),}
\\ 
&\text {$\qquad$Compute $\delta^{(L-1)},\delta^{(L-2)},\cdots,\delta^{(2)} $ using $\delta^{(l)}=((\Theta^{(l)})^T\delta^{(l+1)}).*a^{(l)}.*(1-a^{(l)})$,}
\\ 
&\text {$\qquad$Set $\Delta_{i,j}^{(l)} :=\Delta_{i,j}^{(l)}+a_j^{(l)}\delta_i^{(l+1)}$ or with vecotorization $\Delta^{(l)}:=\Delta^{(l)}+\delta^{(l+1)}(a^{(l)})^T$};
\\
&\text {$\qquad (\Delta_{i,j}^{(l)}$ as an "accumulator" to add up values as we go along and finally compute our $D_{i,j}^{(l+1)}$ ,)}
\\ 
&D_{i,j}^{(l+1)}:=\frac 1 m (\Delta_{i,j}^{(l)}+\lambda\Theta_{i,j}^{(l)})\text{ if $j≠0$};
\\ 
&D_{i,j}^{(l+1)}:=\frac 1 m \Delta_{i,j}^{(l)}\text{ if $j=0$};
\\ 
&\text {Finally, the $D_{i,j}^{(l+1)}=\frac {\partial{J(\Theta)}} {\partial\Theta_{i,j}^{l}}$; is the partial derivative of the $J{(\Theta)}$ we are looking for.} 
\end{align}
$$

#### 1.2.2 反向传播算法直觉 ####

反向传播算法初一看稍显复杂，那么如何从直觉上理解它呢。关键在于理解$
\delta_j^{(l)}=\frac {\partial cost(i)} {\partial{z_j^{(l)}}} = a_j^{(l)}-y_j^{(l)}
$是预测值和实际值在layer L中的差值。

成本函数是：

$$
J(\Theta) =-\frac 1 m  \sum_{i=1}^m \sum_{k=1}^K [y^{(i)}log(h_\theta(x^{(i)}))+(1-y^{(i)})log(1-h_\theta(x^{(i)}))] +  \frac \lambda {2m}  \sum_{l=1}^{L-1} \sum_{i=1}^{s_l} \sum_{j=1}^{s_l+1} (\Theta_{j,i}^l)^2
$$

在只有一个输出和没有正则化情况下，成本函数是：

$$
J(\Theta) =-\frac 1 m  \sum_{i=1}^m[y^{(i)}log(h_\theta(x^{(i)}))+(1-y^{(i)})log(1-h_\theta(x^{(i)}))] 
$$

对于该成本函数：

$$
\begin{align}
\\ 
\frac {\partial J(\Theta)} {\partial\Theta_{i,j}^{(l)}}&←\delta_j^{(l)}=\frac {\partial cost(i)} {\partial{z_j^{(l)}}};
\\
&\text {where $J(\Theta) =-\frac 1 m  \sum_{i=1}^m cost(i);cost(i)=y^{(i)}log(h_\theta(x^{(i)}))+(1-y^{(i)})log(1-h_\theta(x^{(i)}));$}
\\
&\text {and$z_j^{(l)}=\sum_{i=1}^{s_j} \Theta_{i,j}^{(l)}a_{j}^{(l-1)}$}
\end{align}
$$


{: .img_middle_hg}
![Back Propagation Intuition](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-05/Back Propagation Intuition.png)



### 1.3 实现细节 ###

#### 1.3.1 展开参数 ####

由于**fminunc**和**costfunction**的**theta**是**vector**而不是**matrix**，因此当用**神经网络的Back Propagation算法**时，需要将**matrix**展开成**vector**，计算完再将**vector**转回成**matrix**.

{: .img_middle_hg}
![Back Propagation Intuition](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-05/parameter unrolling.png)

#### 1.3.2 梯度检查 ####

由于Back Propagation实现起来比较tricky，有时候看起来$J(θ)$在下降，但是结果却是不对的。为了避免这种情况，我们利用数值斜率来校准Back Propagation算法的实现(称为**Gradient Checking**)。当两者在多种情况下都相近的时候，我们就认定Back Propagation算法的实现没有问题。这个时候再关掉**Gradient Checking**用Back Propagation算法来算J的gradient就在正确性的基础上又有速率(因为Back Propagation远比数值斜率快)

{: .img_middle_hg}
![Back Propagation Intuition](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-05/Gradient Checking.png)

#### 1.3.3 Theta随机初始化 ####

由于fminunc需要一个initialTheta值，而Neutral Network若用0作为所有theta的初始值，则会导致每一个hidden layer里的unit都是一样，使得最终输出错误。因此为了打破这种对称，我们需要随机生成$[-ϵ，+ϵ]$范围的initialTheta。

{: .img_middle_hg}
![Back Propagation Intuition](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-05/random initialization.png)


#### 1.3.4 整合 ####


{: .img_middle_hg}
![Putting Together](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-05/Putting Together.png)


最后请观看一个利用Neural Network实现
[无人驾驶](https://www.youtube.com/watch?v=ilP4aPDTBPE)的视频。


## 2 作业 ##

见[这里](https://github.com/shunmian/-11-Machine-Learning)。附上一张跑分图。


{: .img_middle_lg}
![assignment4](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-05/assignment4.png)


## 3 总结 ##

神经网络的$\Theta$求解可以通过梯度下降结合**Back Propagation Algorithm**(该算法用来计算$\frac {\partial J(\Theta)} {\partial{\Theta_{i,j}^{(l)}}}$)来获取$J_{min}(\Theta)$下的$\Theta$。**Back Propagation Algorithm**稍显复杂，一种直观理解是先用**Forward Propagation**算出$a^L$，然后计算$\delta^{(L)} = a^{(L)}-y$，再将$\delta^{(L)}$反向传播到$a_j^{(l)}$，求出$\delta_j^{(l)}$。最后将$\delta_j^{(l)}$表示成和$\Theta_{i,j}^{(l)}$相关的值，即$\frac {\partial J(\Theta)} {\partial{\Theta_{i,j}^{(l)}}}$。


## 4 参考资料 ##
- [《Deep Learning》](http://deeplearning.net/);
- [《Microsoft Azure Machine Learning》](https://azure.microsoft.com/en-us/services/machine-learning/);
- [《A Visual Introduction to Machine Learning》](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/);





