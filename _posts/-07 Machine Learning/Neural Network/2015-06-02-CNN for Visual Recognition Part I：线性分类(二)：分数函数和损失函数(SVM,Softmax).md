---
layout: post
title: CNN for VR Part I：线性分类(二)：评分函数和损失函数(SVM,Softmax)
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

## 1 Linear Classification ##

### 1.1 Introduction ###

前一篇文章[《线性分类(一)：图片分类问题及KNN》]({{site.baseurl}}/-07%20machine%20learning/2015/06/01/CNN-for-Visual-Recognition-Part-I-线性分类(一)-图片分类问题及KNN.html)我们介绍了**Image Classification**问题，即从固定的一组标签了给某幅图片选取对的标签。更进一步，我们用KNN(k最近邻)分类器尝试解决这一问题。但是KNN有着如下明显的弊端：

1. **空间昂贵**。分类器必须记住所有的training set。这在实际应用中依据具体traing set的大小会耗费几G甚至更多的存储空间。

2. **时间昂贵**。评估test set需要和每一项training set里的数据比较，在实际应用中不现实。

下面我们将要介绍一种比KNN更强大的方法，**线性分类(Linear Classification)**，并在此基础上，最终发展成**Neural Networks**和**Convolutional Neural Networks**。**线性分类**包括两个主要部分，**评分函数(Score Function，映射原始数据到每个class的分数)**和**损失函数(Function，评估预测label和真实label之间的距离)**。在这基础上，我们需要做的就是**optimize**评分函数，使得损失函数最小。

### 1.2 评分函数 ###

线性分类的评分函数可以用如下公式准确描述：

$$
f(x,\mathbf{W},b) = \mathbf{W}_{N\times D} x_{D \times 1}+b_{N \times 1} = y_{N \times 1}
$$

$$
\begin{array}{lcr}
\begin{bmatrix} w_{11} & w_{12} & ... &w_{1D} \\ w_{21} & w_{22} & ... &w_{2D} \\ ... & ... & ... &... \\ w_{N1} & w_{N2} & ... &w_{ND} \\ \end{bmatrix}
\begin{bmatrix} x_{1} \\ x_{2} \\ ... \\ ...\\ x_{D}  \\ \end{bmatrix}
+
\begin{bmatrix} b_{1} \\ b_{2} \\  ...\\ b_{N}  \\ \end{bmatrix}
=
\begin{bmatrix} y_{1} \\ y_{2} \\ ... \\ y_{N}  \\ \end{bmatrix}
\end{array}

$$


或者可以更近一步将b放入$$\mathbf{W}$$。

$$
f(x,\mathbf{W}) = \mathbf{W}_{N \times (D+1)}x_{(D+1) \times 1}= y
$$

$$
\begin{array}{lcr}
\begin{bmatrix} w_{11} & w_{12} & ... & w_{1D} & b_1 \\ w_{21} & w_{22} & ... &w_{2D} & b_2 \\ ... & ... & ... &... &... \\ w_{N1} & w_{N2} & ... &w_{ND} &b_N \\ \end{bmatrix}
\begin{bmatrix} x_{1} \\ x_{2} \\ ... \\ ...\\ x_{D} \\ 1  \\ \end{bmatrix}

=
\begin{bmatrix} y_{1} \\ y_{2} \\ ... \\ y_{N}  \\ \end{bmatrix}
\end{array}

$$

> Foreshadowing: Convolutional Neural Networks will map image pixels to scores exactly as shown above, but the mapping (f) will be more complex and will contain more parameters.

#### 1.2.1 Interpreting a Linear Classifier ####

##### 1.2.1.1 Analogy of Images as High-dimensional Points ######

因为images被伸展成了高维的列向量，我们可以认为每一幅图都是该高维空间的1个点(比如每一幅CIFAR-10的图片都是$$32 \times 32 \times 3 = 3072$$维度的1个点)。类似的，所有的图片就是所有点的集合。而评分函数只x的线性函数加偏置，在2维平面上(x是2维的点)，w的每一行都是一条分类器直线。

$$
f(x,\mathbf{W},b) = \mathbf{W}_{N\times 2} x_{2 \times 1}+b_{N \times 1} = y_{N \times 1}
$$

$$
\begin{array}{lcr}
\begin{bmatrix} w_{11} &w_{12}& b_1 \\ w_{21}  & w_{22}  & b_2 \\... &... \\ w_{N1} &w_{N2} &b_N \\ \end{bmatrix}
\begin{bmatrix} x_{1} \\ x_{2} \\ 1  \\ \end{bmatrix}

=
\begin{bmatrix} y_{1} \\ y_{2} \\ ... \\ y_{N}  \\ \end{bmatrix}
\end{array}

$$


1. 在直线上，$$
f(x,\mathbf{W}_i,b) = \mathbf{W}_{i\times 2} x_{2 \times 1}+b_{i \times 1} - y_{i \times 1} = 0$$；

2. 在直线一边$$
f(x,\mathbf{W}_i,b) = \mathbf{W}_{i\times 2} x_{2 \times 1}+b_{i \times 1} - y_{i \times 1} > 0$$；

3. 在直线另一边$$
f(x,\mathbf{W}_i,b) = \mathbf{W}_{i\times 2} x_{2 \times 1}+b_{i \times 1} - y_{i \times 1} < 0$$；


{: .img_middle_lg}
![pixelspace](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-02-CNN for Visual Recognition Part I：线性分类(二)：分数函数和损失函数(SVM,Softmax)/pixelspace.jpeg)

##### 1.2.1.2 Interpretation of Linear Classifiers as Template Matching #####

另外一种对权重**W**的解读是其每一行都是1个类的模板(template)。而1幅图每个类的分数就是该类的模板和该图像素的内积。从这个角度来说，linear classifier就是在做**template matching**；而**template**就是从training set中学习得来的。或者换个角度来说，我们还是在做**Nearest Neighbor**，但是不同于对所有training set里的图片进行比较，我们只对学习得来的每个类的template进行比较，并且**距离**是**内积**而不是**L1**或**L2**。

{: .img_middle_lg}
![templates](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-02-CNN for Visual Recognition Part I：线性分类(二)：分数函数和损失函数(SVM,Softmax)/templates.jpg)

上图是CIFAR-10数据获得的10个类各自的template。该如何解读这些模板呢。

1. 拿ship来举例，它有很多蓝色像素(水的颜色)，所以当评估的图片有很多蓝色像素在对应的位置的时候，内积的评分就会高，说明template match率高。

2. 再看horse。貌似horse有左右两个头，这意味着training set里的horse有一部分朝左，一部分朝右。因此template将两者merge。

3. 再来看car。template显示红色，并且面向前面，这表明大部分CIFAR-10里的car类别的图片都是面向前面的红色car。可以看到linear classifier对于不同颜色的car的评估会比较弱，我们后面的**Neural Network**将会试图解决这一问题。

### 1.3 损失函数 ###

上面介绍的评分函数是1个classifier($$f(x,\mathbf{W}_i,b)$$)对任意图片的评估。我们不能改变$$x$$，只能改变$$\mathbf{W}_i和b$$。那么如何评价该classifier和准确值之间的差距呢(即该classifier的好坏)，这个时候就需要用到损失函数了。

#### 1.3.1 SVM ####

$$
L_i = \sum_{j\neq y_i} \max(0, s_j - s_{y_i} + \Delta), \text{where $s_j = f(x_i, W)_j,y_i \text{ is the true label index}$} 
$$

$$
L_i = \sum_{j\neq y_i} \max(0, w_j^T x_i - w_{y_i}^T x_i + \Delta)
$$

例如$s = [13, -7, 11]$时，true label index是0，且假设$\Delta$是10，那么：

$$
L_i = \max(0, -7 - 13 + 10) + \max(0, 11 - 13 + 10) = 8
$$

对于第一项，正确的类的分数(13)远大于非正确的分数(-7)by a margin 10，因此该项loss为0；对于第二项，虽然正确的分数(13)大于非正确的分数(11)，但是大于的margin不够10，因此该项loss为8。**SVM loss**的本质是量化某classifier对training set的准确度，即正确项必须比其他项大于一定的余量，否则视为loss，用下图表示。

{: .img_middle_lg}
![SVM margin](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-02-CNN for Visual Recognition Part I：线性分类(二)：分数函数和损失函数(SVM,Softmax)/SVM margin.jpg)

##### 1.3.1.1 Regularization #####

如果有1个$W$可以正确分类所有training set的data，那么请问该$W$是唯一的吗？答案是否定的，任何$\lambda W$也是该解，而且可能存在其他的解。因此如何给我们最终的$W$定1个规范呢，我们想要的是所有的在$W 里的 W_{ij}$都是在1个数量级上且数量级都比较小。因此我们加入了**Regularization Penalty**$R(W)$：

$$R(W) = \sum_k\sum_l W_{k,l}^2$$

最后的SVM loss function变成：

$$
L =  \underbrace{ \frac{1}{N} \sum_i L_i }_\text{data loss} + \underbrace{ \lambda R(W) }_\text{regularization loss} \\\\
$$


完整的形式是：


$$
L = \frac{1}{N} \sum_i \sum_{j\neq y_i} \left[ \max(0, f(x_i; W)_j - f(x_i; W)_{y_i} + \Delta) \right] + \lambda \sum_k\sum_l W_{k,l}^2
$$


1个正则化最大的特点是削弱了权重之间的差距，防止过度拟合(overfit)。比如$x = [1,1,1,1]$对应两个权重$w_1 = [1,0,0,0]$和$w_2 = [0.25,0.25,0.25,0.25]$。那么$w_1^Tx = w_2^Tx = 1$因此两者的内积相同，但是正则项前者是1后者是0.25，因此我们倾向选$w_2$。从另一个角度说，$w_2$考虑了4个维度而$w_1$只考虑了1个维度，因此$w_2$更全面。

##### 1.3.1.2 Practical Consideration #####

1. **Setting Delta**。我们容易得出结论$\Delta$的值和$W$成正比，即若$\Delta$对应某个$W$，则$\lambda\Delta$对应$\lambda W$。因此我们可以设置$\Delta$为1。它的值的大小对$W$无影响。正如$\Delta$影响**data loss**，$\lambda$影响**regularization loss**；这两项在相互角力。所以当我们设置$\Delta$为1后，真正需要关心的便是$\lambda$，即我们允许weights的取值上限。

2. **Relation to Binary Support Vector Machine**。略。

3. **Aside:Optimizationin primal**。略。

4. **Aside:Other Multicalss SVM formulations**。略。


#### 1.3.2 Softmax ####

Machine Learning有两个常见的loss funcition，1个是**SVM**，另一个便是我们这里要讲的**Softmax**。从它的名字可以看出它会给出一列值里的最大值，但是不同于我们常用的max(这里称之为hardmax)只给出最大的那个，softmax给出一列值里每个值的出现概率。Softmax的本质是**cross-entropy loss**，即和正确值之间的交叉熵。Softmax的形式如下：

$$
L_i = -\log\left(\frac{e^{f_{y_i}}}{ \sum_j e^{f_j} }\right) \hspace{0.5in} \text{or equivalently} \hspace{0.5in} L_i = -f_{y_i} + \log\sum_j e^{f_j}
$$

完整的形式是：

$$
L =  \underbrace{ \frac{1}{N} \sum_i L_i }_\text{data loss} + \underbrace{ \lambda R(W) }_\text{regularization loss} \\\\
$$

其中$f_j$对应j-th score。

实际应用中$e^{f_{y_i}}$和$\sum_j e^{f_j}$可能会很大，它们之间的除法会不稳定，因此我们可以同时缩小分子和分母，使得最终商不变。

$$\frac{e^{f_{y_i}}}{\sum_j e^{f_j}}
= \frac{Ce^{f_{y_i}}}{C\sum_j e^{f_j}}
= \frac{e^{f_{y_i} + \log C}}{\sum_j e^{f_j + \log C}}$$

1个通常的做法是取$\log C = -\max_j f_j$，即$C = e^{-\max_j f_j}$。This simply states that we should shift the values inside the vector ff so that the highest value is zero。

#### 1.3.3 SVM vs. Softmax ####

下图总结了**SVM**和**Softmax**的联系和区别。两种loss function下，我们共用了1中score function：

1. SVM解读score为class的score，其loss function鼓励正确的类(class 2, in blue)比其他类至少高出margin；

2. Softmax解读score为各个class的概率，其loss function鼓励正确的类的概率大于其他类。

3. 最终的loss是SVM=1.58和Softmax=1.04。这两个loss相互之间并不具有可比性，只有在同样的loss function里才有可比性。



{: .img_middle_hg}
![svm vs softmax](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-02-CNN for Visual Recognition Part I：线性分类(二)：分数函数和损失函数(SVM,Softmax)/svm vs softmax.png)

##### 1.3.3.1 Softmax Classifier Provides “Probabilities” for Each Class #####

不像SVM，Softmax对score的interpretation非常直观，即各个class出现的概率。例如SVM有$[12.5,0.6,-23.0]$的score，Softmax的概率是$[0.9,0.09,0.01]$(表示各个class出现的"概率")。但是面对该"概率"时要谨慎，因为不同的的regularization strength $\lambda$ 概率间的差距会不一样。例如某个$\lambda$对应的score是$[1,-2,0]$，那么softmax将会进行下列计算：

$$
[1, -2, 0] \rightarrow [e^1, e^{-2}, e^0] = [2.71, 0.14, 1] \rightarrow [0.7, 0.04, 0.26]
$$

如果我们让$\lambda$减半，则：

$$
[0.5, -1, 0] \rightarrow [e^{0.5}, e^{-1}, e^0] = [1.65, 0.37, 1] \rightarrow [0.55, 0.12, 0.33]
$$

可以看到强$\lambda$概率更分散，弱$\lambda$概率更均匀。因此，Softmax计算出的"概率"，我们更适合解读为不同class之间的相对顺序，而不是绝对差值(同样适用于SVM)。


##### 1.3.3.2 In Practice Performance, SVM and Softmax are Usually Comparable #####

实际运用中，SVM和Softmax的perfomance比较接近，不同的人选择不同的loss function。但是需要注意的是SVM有1个threhold，类似于粗调，只要大于该margin，loss就不会变；而Softmax类似微调，任何微小的变化都会改变loss。比如SVM时，$[10,-100,-100]$和$[10,9,9]$的loss是一样当margin为1时；但是对于softmax就不一样，$[10,-100,-100]$的loss比$[10,9,9]$的loss就会小很多。所以SVM只要满足margin，而不会微调这之后的loss。

## 2 Summary ###

{: .img_middle_hg}
![Score Function & Loss Function Summary](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-02-CNN for Visual Recognition Part I：线性分类(二)：分数函数和损失函数(SVM,Softmax)/Score Function & Loss Function Summary.png)

## 3 参考资料 ##
- ["CS231n: Convolutional Neural Networks for Visual Recognition"](http://cs231n.stanford.edu/);
- ["CS231n Course Video 2016 Winter"](https://www.youtube.com/watch?v=g-PvXUjD6qg&index=1&list=PLIUoqCcJd2BjsI11qafvMWv_UqiH1Wu3Q);
- ["Deep Learning"](http://www.deeplearningbook.org/);





