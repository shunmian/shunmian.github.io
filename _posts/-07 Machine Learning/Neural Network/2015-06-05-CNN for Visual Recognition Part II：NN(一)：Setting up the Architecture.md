---
layout: post
title: CNN for VR Part II：NN(一)：Setting up the Architecture
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

## 1 Setting Up the Architecture ##

### 1.0 Quick Intro without Brain Analogies ###

在线性分类中，我们通过将输入进行线性转换$s=Wx$来获取评分函数，多次线性转换操作是无意义的，因为最终都可合并为1次等价的线性操作；而在Neural Network中，我们可以有多个线性转换的操作，但是在各个线性转换操作之间，加入了非线性转换操作(即激励函数，activation function)，这样，NN就可以接近任何(包括线性和非线性)函数，而不会像线性分类一样，多少次线性操作合并起来都等同于1个线性操作。具体来说，1个2层的NN不是计算$S=W_2(W_1x) = (W_2W_1)x = W_{comb}x$，而是计算$s=W_2\max(0,W_1x)$，其中$\max(0,-)$函数就是非线性函数(激励函数)。同理，1个三层的NN可以表示成$s=W_3\max(0,W_2\max(0,W_1))$，其中$W_1$，$W_2$，$W_3$可以通过反向传播优化获得，而它们各自的size属于超参，本文后面会介绍如何获得合适的超参。因此Softmax Classifier就是个1层的NN，它的激励函数是每个label的概率，即$\frac{e^{f_{y_i}}}{ \sum_j e^{f_j} }$。SVM Classifier也是个1层的NN，它的激励函数是与对的label之间的余量，即$\max(0, w_j^T x_i - w_{y_i}^T x_i + \Delta)$ 。

### 1.1 Modeling One Neuron ###

#### 1.1.1 Biological Motivation and Connections ####

下面我们来从神经元的角度来解释神经网络，一个简单的粗糙的对神经元的近似如下图。

{: .img_middle_lg}
![Neuron](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-05-CNN for Visual Recognition Part II：NN(一)：Setting up the Architecture/Neuron.png)

1个神经元的工作原理可以粗糙的概括成将输入->线性变化->激励函数(值足够大的时候，激发信号)->输出的过程，即$f(Wx+b)$。

#### 1.1.2 Single Neuron as a Linear Classifier ####

单个神经元是1个线性分类器，可以从下面三方面来理解：

1. **Binary Softmax Classifier**。当激励函数是$\frac{e^{f_{y_i}}}{ \sum_j e^{f_j} }$时，是Softmax classifier。

2. **Binary SVM Classifier**。当激励函数是$\max(0, w_j^T x_i - w_{y_i}^T x_i + \Delta)$时，是SVM classifier。

3. **Regularization**。从神经元这种生物学的角度来看，正则化可以看做**逐步遗忘(gradual forgetting)**的过程，因为每次$W$的update都会被正则化趋向的零。

#### 1.1.3 Commonly used Activation Functions #### 

常用的激励函数总结成下图。

{: .img_middle_hg}
![Activation Function](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-05-CNN for Visual Recognition Part II：NN(一)：Setting up the Architecture/Activation Function.png)


**TLDR**: “What neuron type should I use?” Use the ReLU non-linearity, be careful with your learning rates and possibly monitor the fraction of “dead” units in a network. If this concerns you, give Leaky ReLU or Maxout a try. Never use sigmoid. Try tanh, but expect it to work worse than ReLU/Maxout.


### 1.2 NN Architecture ###

#### 1.2.1 Layer-wise Organization ####

**NN**是由neurons组成的layer的acyclic graph；同1个layer内的neurons不会有数据交流。

{: .img_middle_lg}
![NN size参数](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-05-CNN for Visual Recognition Part II：NN(一)：Setting up the Architecture/NN size参数.png)

#### 1.2.2 Example Feed-forward Computation ####

NN组织成layer的形式可以使得前向传播的计算形式简化成多次应用矩阵相乘后经激励函数。

> **The forward pass** of a fully-connected layer corresponds to one matrix multiplication followed by a bias offset and an activation function.

{% highlight py linenos %}

# forward-pass of a 3-layer neural network:
f = lambda x: 1.0/(1.0 + np.exp(-x)) # activation function (use sigmoid)
x = np.random.randn(3, 1) # random input vector of three numbers (3x1)
h1 = f(np.dot(W1, x) + b1) # calculate first hidden layer activations (4x1)
h2 = f(np.dot(W2, h1) + b2) # calculate second hidden layer activations (4x1)
out = np.dot(W3, h2) + b3 # output neuron (1x1)

{% endhighlight %}

#### 1.2.3 Representation Power ####

1个随之而来的问题是NN是否能近似所有的函数，或者说是不是存在某种函数不能由NN来近似？事实上1个1层的NN可以近似任何连续函数(e.g. see [Approximation by Superpositions of Sigmoidal Function](http://www.dartmouth.edu/~gvc/Cybenko_MCSS.pdf), or this [intuitive explanation](http://neuralnetworksanddeeplearning.com/chap4.html) from Michael Nielsen)。

既然1个1层的NN可以近似任何连续函数，那么我们为什么要用多层NN呢？理论上说，多层NN和单层NN的表达能力是相同的，即都有可以近似任何连续函数；但是实际上，多层NN比单层NN的实现更有效。

#### 1.2.4 Setting Number of Layers and Their Sizes ####

实际实现中，小的NN(少的hidden layer层数和neuron个数)，有一定概率你可以获得很小的loss或者很大；而在大的NN(多的hidden layer层数和neuron个数)，不同的local minima之间的loss差距很小，很大概率你可以获得效果几乎一样的最优解。

{: .img_middle_lg}
![x vs lambda](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-05-CNN for Visual Recognition Part II：NN(一)：Setting up the Architecture/x vs lambda.png)

因此，对于如何设置layer层数很neuron个数，本文的建议是在你的计算机能力范围内，层数尽量大，neuron个数尽量多，然后用regulariztion(或者higher weight decay)防止overfitting。

## 2 Summary ##

{: .img_middle_hg}
![NN Architecture Summary](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-05-CNN for Visual Recognition Part II：NN(一)：Setting up the Architecture/NN Architecture Summary.png)


## 3 参考资料 ##
- ["CS231n: Convolutional Neural Networks for Visual Recognition"](http://cs231n.stanford.edu/);
- ["CS231n Course Video 2016 Winter"](https://www.youtube.com/watch?v=g-PvXUjD6qg&index=1&list=PLIUoqCcJd2BjsI11qafvMWv_UqiH1Wu3Q);
- ["Deep Learning"](http://www.deeplearningbook.org/);



