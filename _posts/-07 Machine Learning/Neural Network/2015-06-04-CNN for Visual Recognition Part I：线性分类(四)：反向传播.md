---
layout: post
title: CNN for VR Part I：线性分类(四)：反向传播
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: 机器学习(Machine Learning)是通过非显性编程让计算机获得学习的能力，这在现代计算机科学中有着广泛的应用，从google的搜索分类，到OCR的训练以及AlphaGo的人工智能等等。本文是Coursera上吴恩达教授的《Machine Learning》系列课程的第一篇笔记：线性回归之单变量。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Backpropagation ##

前文我们介绍了单个梯度的计算方法，本文在这基础上，介绍如何迭代使用链式原则来计算表达式的梯度，即**反向传播(Backpropagation)**。这对于我们理解，开发，设计和调试**Neural Networks**至关重要。

本文要探讨的主要问题如下。

> 反向传播主要问题：给定某个函数$f(x)$，其中$x$是输入向量，我们如何计算$f$对$x$的梯度，即$\triangledown f(x)$。

该问题在神经网络下的应用是$f$对应loss function，$x$对应training set中的data $(x_i,y_i), i=1 \ldots N$，权重$W$和偏置$b$。由于training set中的data是固定的，因此我们一般只需要计算$W$和$b$的梯度来更新它们的值来减少loss。

### 1.1 Introduction 

#### 1.1.1 Simple Interpreting the Gradient ###

一些简单的梯度我们总结如下。

$$ \begin{align} 
f(x,y) = x y \hspace{0.5in} & \rightarrow \hspace{0.5in} \frac{\partial f}{\partial x} = y \hspace{0.5in} \frac{\partial f}{\partial y} = x \\

f(x,y) = x + y \hspace{0.5in} & \rightarrow \hspace{0.5in} \frac{\partial f}{\partial x} = 1 \hspace{0.5in} \frac{\partial f}{\partial y} = 1 \\

f(x,y) = \max(x, y) \hspace{0.5in} & \rightarrow \hspace{0.5in} \frac{\partial f}{\partial x} = \mathbb{1}(x >= y) \hspace{0.5in} \frac{\partial f}{\partial y} = \mathbb{1}(y >= x) 

\end{align} $$


#### 1.1.2 Compound Expression, Chain Rule, Backpropagation ###

复合表达式的梯度遵循链式原则。

$$\begin{align}  
f(x,y) = （x + y） \times z \hspace{0.5in} & \rightarrow \hspace{0.5in} \frac{\partial f}{\partial x} = \frac{\partial f}{\partial (x + y)} \times \frac{\partial (x + y)}{\partial x} = z \times 1 = z \\
                                                                     & \rightarrow \hspace{0.5in} \frac{\partial f}{\partial y} = \frac{\partial f}{\partial (x + y)} \times \frac{\partial (x + y)}{\partial y} = z \times 1 = z \\
                                                                     & \rightarrow \hspace{0.5in} \frac{\partial f}{\partial z} =  x + y \\
\end{align} $$

{: .img_middle_lg}
![pixelspace](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-04-CNN for Visual Recognition Part I：线性分类(四)：反向传播/Chain Rule.png)

### 1.2 Intuition ###

> Backgroupagation is an elegant local process. The basic unit is gate. During forward pass, each gate's gradient, $\triangledown \frac {G_{out_i}}{G_{in_i}}$ and value ${G_{out_i}}$ is computed with knwon ${G_{in_i}}$. During backward pass, the previous computed gradient is chained by multiplication to calculate the final output's gradient on initial input, $\triangledown \frac {G_{out_N}}{G_{in_0}}$.

### 1.3 Best Practice###

#### 1.3.1 Modularity: Sigmoid Example ####

我们可以将多个gate模块化成1个gate，这里用1个**Sigmoid函数**举例。

{: .img_middle_lg}
![pixelspace](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-04-CNN for Visual Recognition Part I：线性分类(四)：反向传播/Modular Gate.png)

#### 1.3.2 Staged Computation ####

下面这个形式只是用于练习利用中间值(Staged Value)来进行backpropagation。中间值使得我们的计算大大简化，而没有必要算出每一个微小gate的gradient。

$$f(x,y) = \frac{x + \sigma(y)}{\sigma(x) + (x+y)^2}$$

{% highlight py linenos %}
x = 3 # example values
y = -4

# forward pass
sigy = 1.0 / (1 + math.exp(-y)) # sigmoid in numerator   #(1)
num = x + sigy # numerator                               #(2)
sigx = 1.0 / (1 + math.exp(-x)) # sigmoid in denominator #(3)
xpy = x + y                                              #(4)
xpysqr = xpy**2                                          #(5)
den = sigx + xpysqr # denominator                        #(6)
invden = 1.0 / den                                       #(7)
f = num * invden # done!                                 #(8)

# backprop f = num * invden
dnum = invden # gradient on numerator                             #(8)
dinvden = num                                                     #(8)
# backprop invden = 1.0 / den 
dden = (-1.0 / (den**2)) * dinvden                                #(7)
# backprop den = sigx + xpysqr
dsigx = (1) * dden                                                #(6)
dxpysqr = (1) * dden                                              #(6)
# backprop xpysqr = xpy**2
dxpy = (2 * xpy) * dxpysqr                                        #(5)
# backprop xpy = x + y
dx = (1) * dxpy                                                   #(4)
dy = (1) * dxpy                                                   #(4)
# backprop sigx = 1.0 / (1 + math.exp(-x))
dx += ((1 - sigx) * sigx) * dsigx # Notice += !!                  #(3)
# backprop num = x + sigy
dx += (1) * dnum                                                  #(2)
#This follows the multivariable chain rule in Calculus, which states that if a variable branches out to different parts of the circuit, then the gradients that flow back to it will add

dsigy = (1) * dnum                                                #(2)
# backprop sigy = 1.0 / (1 + math.exp(-y))
dy += ((1 - sigy) * sigy) * dsigy                                 #(1)
# done! phew

{% endhighlight %}

### 1.4 Patterns in Backward Flow ###

{: .img_middle_lg}
![pixelspace](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-04-CNN for Visual Recognition Part I：线性分类(四)：反向传播/Gate Patterns.png)

### 1.5 Gradients for Vectorized Operations ###

前面我们考虑的都是单变量，但是相关概念可以smooth地扩展到多变量(矩阵)。

{% highlight py linenos %}

# forward pass
W = np.random.randn(5, 10)
X = np.random.randn(10, 3)
D = W.dot(X)

# now suppose we had the gradient on D from above in the circuit
dD = np.random.randn(*D.shape) # same shape as D
dW = dD.dot(X.T) #.T gives the transpose of the matrix
dX = W.T.dot(dD)

{% endhighlight %}

## 2 Summary ##

{: .img_middle_hg}
![pixelspace](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-04-CNN for Visual Recognition Part I：线性分类(四)：反向传播/线性分类Summary.png)


## 3 参考资料 ##
- ["CS231n: Convolutional Neural Networks for Visual Recognition"](http://cs231n.stanford.edu/);
- ["CS231n Course Video 2016 Winter"](https://www.youtube.com/watch?v=g-PvXUjD6qg&index=1&list=PLIUoqCcJd2BjsI11qafvMWv_UqiH1Wu3Q);
- ["Deep Learning"](http://www.deeplearningbook.org/);



