---
layout: post
title: CNN for VR Part I：线性分类(三)：优化(梯度下降)
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

## 1 Optimization ##

[上一篇文章]({{site.baseurl}}/-07%20machine%20learning/2015/06/02/CNN-for-Visual-Recognition-Part-I-线性分类(二)-分数函数和损失函数(SVM,Softmax))介绍了图片分类问题中的两个key components：

1. **评分函数(Score Function)**：将原始数据映射到预测label；

2. **损失函数(Loss Function)**：测量评分函数和真实值的差距。

更具体的说，SVM损失函数具有如下格式：

$$
L = \frac{1}{N} \sum_i \sum_{j\neq y_i} \left[ \max(0, f(x_i; W)_j - f(x_i; W)_{y_i} + 1) \right] + \alpha R(W)
$$

我们将要介绍第3种也是最后1种component，即**Optimization**，用来最小化上述损失函数来获得最优评分函数。一旦我们理解了这3个component如何协同工作，我们会重新回顾第1个component来扩展比线性分类其更复杂的形式：即首先整个Neural Networks，其次是Convolutional Neural Networks。在这过程中，损失函数和optimization过程相对保持不变。

### 1.1 Visualizing the Loss Function ###

对于$$10 \times 3073$$的weight，可视化比较困难。我们可以用另一个视角来看loss和weight的关系，即用1维的直线，或者2维的平面来切多维的维度。

1. **1维直线**。我们随机选取某个$$W_r$$，然后再取1个1维的单位方向$$W_1$$,那么$$W_r+aW_1$$就是通过$$W_r$$沿$$W_1$$直线的任一点，而该点对应的损失就是$$L(W_r+aW_1)$$。画出$$L(W+aW_1)$$ vs $$a$$的图像就如下图左图所示。

2. **2维平面**。同理，我们随机选取某个$$W_r$$，然后取某个2维平面的正交单位向量$$W_1$$和$$W_2$$，那么该平面上所有的点可以表示成$$W_r+aW_1+bW_2$$，其对应的损失就是$$L(W_r+aW_1+bW_2)$$。画出$$L(W_r+aW_1+bW_2)$$ vs $$(W_r+aW_1+bW_2)$$的图像就如下图中图所示。如果取多个loss的平均值，即$$\frac{1}{N}\sum_i {L_i(W_r+aW_1+bW_2)}$$，则如下图右图所示。

{: .img_middle_lg}
![pixelspace](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-03-CNN for Visual Recognition Part I：线性分类(三)：优化(随机梯度下降)/loss visualization.png)

对于线性分类，损失函数是1个凹函数(convex function)，而最小点沿着梯度很容易找到。而对于NN或者CNN，损失函数就不再是凹函数了，而是复杂的起伏不定的形状，对其寻找最低点的过程，我们后面会细讲，这里暂时按下不表。

### 1.2 Optimization Strategies ###

对于线性分类的函数，optimization的过程可以有3种策略，所有策略的目的都是寻找最小loss，但是效率不同。

#### 1.2.1 Random Search ####

1个最"naive"的实现是随机选取n个$$W$$，依次比较其loss，找出n个$$W$$中损失最小的那个。这种策略类似穷举法，效率非常低。

#### 1.2.2 Random Local Search ####

另外1个更加高效的策略是开始时选取1个随机的$$W$$，然后给定1个很小的步长$$\delta$$和一个随机的方向$$W_1$$：若$$W+\delta W_1$$的损失比$$W$$小，则将$$W$$更新至$$W+\delta W_1$$，然后重复上述过程；否则再取1个随机的$$W_1$$，进行比较。这种策略的效率比第1种高，但是仍然不是最好的。

#### 1.2.3 Following the Gradient ####

在第2种策略的基础，我们可以进行优化，即我们不需要随机选$$W_1$$，而是可以根据其梯度，来选$$W_1$$，这样可以保证每次的$$W+\delta W_1$$的损失都比$$W$$小，进行更新。而且可以保证该方向loss是下降最快的，可以这样理解，1维的时候，梯度告诉我们下降的方向是沿着变量变大的方向还是变小的方向；2维的时候，梯度告诉我们下降的方向是两个正交变量的1维方向的矢量和，沿着该矢量方向，loss下降最快；n维的时候，loss下降最快的是n个1维的梯度的矢量和的方向。

### 1.3 Computing the Gradient ###

第3种策略的关键在于如何计算梯度，通常来说有以下两种:

1. 数值梯度(Numerical Gradient)，实现简单，但是计算缓慢;

2. 解析梯度(Analytical Gradient)，计算快但是容易出错。

#### 1.3.1 Numerically with Finite Differences ####

n维数值梯度可以用1维数值梯度来理解，即对于函数$f(x)$，给定某个$x_0$，则$f$在$x_0$的数值梯度是：

$$\frac{df(x)}{dx}|_{x=x_0} = \lim_{h\ \to 0} \frac{f(x_0 + h) - f(x_0)}{h}$$

Python 代码如下。
{% highlight py linenos %}
def eval_numerical_gradient(f, x):
  """ 
  a naive implementation of numerical gradient of f at x 
  - f should be a function that takes a single argument
  - x is the point (numpy array) to evaluate the gradient at
  """ 

  fx = f(x) # evaluate function value at original point
  grad = np.zeros(x.shape)
  h = 0.00001

  # iterate over all indexes in x
  it = np.nditer(x, flags=['multi_index'], op_flags=['readwrite'])
  while not it.finished:

    # evaluate function at x+h
    ix = it.multi_index
    old_value = x[ix]
    x[ix] = old_value + h # increment by h
    fxh = f(x) # evalute f(x + h)
    x[ix] = old_value # restore to previous value (very important!)

    # compute the partial derivative
    grad[ix] = (fxh - fx) / h # the slope
    it.iternext() # step to next dimension

  return grad

# to use the generic code above we want a function that takes a single argument
# (the weights in our case) so we close over X_train and Y_train
def CIFAR10_loss_fun(W):
  return L(X_train, Y_train, W)

W = np.random.rand(10, 3073) * 0.001 # random weight vector
df = eval_numerical_gradient(CIFAR10_loss_fun, W) # get the gradient

loss_original = CIFAR10_loss_fun(W) # the original loss
print 'original loss: %f' % (loss_original, )

# lets see the effect of multiple step sizes
for step_size_log in [-10, -9, -8, -7, -6, -5,-4,-3,-2,-1]:
  step_size = 10 ** step_size_log
  W_new = W - step_size * df # new position in the weight space
  loss_new = CIFAR10_loss_fun(W_new)
  print 'for step size %f new loss: %f' % (step_size, loss_new)

# prints:
# original loss: 2.200718
# for step size 1.000000e-10 new loss: 2.200652
# for step size 1.000000e-09 new loss: 2.200057
# for step size 1.000000e-08 new loss: 2.194116
# for step size 1.000000e-07 new loss: 2.135493
# for step size 1.000000e-06 new loss: 1.647802
# for step size 1.000000e-05 new loss: 2.844355
# for step size 1.000000e-04 new loss: 25.558142
# for step size 1.000000e-03 new loss: 254.086573
# for step size 1.000000e-02 new loss: 2539.370888
# for step size 1.000000e-01 new loss: 25392.214036

{% endhighlight %}

stepsize的选择需要小心对待，调小了update不够有效，太大了可能loss会上升。

#### 1.3.2 Analytically with Calculus ####

数值梯度计算昂贵，解析梯度相比就便宜的多，但是却有易出错的问题。因此一般是先用数值梯度验证解析梯度的准确性，然后再只用解析梯度，这个过程我们称之为**梯度验算(Gradient Check)**。

对于SVM的损失函数，解析梯度是如下形式：


$$ \begin{align} 
L_i & = \sum_{j\neq y_i} \left[ \max(0, w_j^Tx_i - w_{y_i}^Tx_i + \Delta) \right] \\

\nabla_{w_{y_i}} L_i & = - \left( \sum_{j\neq y_i} \mathbb{1}(w_j^Tx_i - w_{y_i}^Tx_i + \Delta > 0) \right) x_i \\

\nabla_{w_j} L_i & = \mathbb{1}(w_j^Tx_i - w_{y_i}^Tx_i + \Delta > 0) x_i 

\end{align} $$

最终的形式如下：

$$\triangledown L = \frac{1}{N} \sum_i 

\begin{cases}
- \left( \sum_{j\neq y_i} \mathbb{1}(w_j^Tx_i - w_{y_i}^Tx_i + \Delta > 0) \right) x_i,  & \text{if $j= y_i$ } \\
\mathbb{1}(w_j^Tx_i - w_{y_i}^Tx_i + \Delta > 0) x_i , & \text{if $j\neq y_i$}  \\
\end{cases} +

 \lambda \sum_k\sum_l W_{k,l}^2$$

### 1.4 Gradient Descent ###

上节我们介绍如何计算损失函数的**梯度**，而通过反复计算梯度并且优化$$W$$的过程称之为**Gradient Descent**，一段典型的代码如下：

{% highlight py linenos %}
# Vanilla Gradient Descent
while True:
  weights_grad = evaluate_gradient(loss_fun, data, weights)
  weights += - step_size * weights_grad # perform parameter update
{% endhighlight %}

除了**Gradient Descent**，还有一些其他形式的optimization，比如**LBFGS**。而**Gradient Descent**是优化**Neural Networks**最常见的方法，这门课我们后面用到的优化和上述代码无异，只是在细节上会有不同。

#### 1.4.1 Mini-batch Gradient Descent ####

在实际大规模应用中(如ILSVRC Challenge)，training set会有上百万个examples。所以每一次优化update $$W$$如果需要计算每一个example就会非常昂贵。1个常用的方法是从training set中取一小批数据进行优化，比如ConvNets中1个典型的batch包括了从1.2 billion 中选取的256个example，然后用该batch进行1次update。

{% highlight py linenos %}
# Vanilla Minibatch Gradient Descent
while True:
  data_batch = sample_training_data(data, 256) # sample 256 examples
  weights_grad = evaluate_gradient(loss_fun, data_batch, weights)
  weights += - step_size * weights_grad # perform parameter update
{% endhighlight %}

**Mini-batch Gradient Descent**可以工作的原因是example之间是相互联系的，256个example是对1.2million example的一种近似。因此通过**Mini-batch Gradient Descent**来进行update我们可以更快的收敛。通常batch的example数量最好是2的n次方，比如，32，64或者128，因为很多向量操作的实现对于2的指数的数据操作更快。

## 2 Summary ##

{: .img_middle_hg}
![pixelspace](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-03-CNN for Visual Recognition Part I：线性分类(三)：优化(随机梯度下降)/Optimization Summary.png)





## 6 参考资料 ##
- ["CS231n: Convolutional Neural Networks for Visual Recognition"](http://cs231n.stanford.edu/);
- ["CS231n Course Video 2016 Winter"](https://www.youtube.com/watch?v=g-PvXUjD6qg&index=1&list=PLIUoqCcJd2BjsI11qafvMWv_UqiH1Wu3Q);
- ["Deep Learning"](http://www.deeplearningbook.org/);


