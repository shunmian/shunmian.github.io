---
layout: post
title: CNN for VR Part I：线性分类(一)：图片分类问题及KNN
categories: [-07 Machine Learning]
tags: [Neural Networks]
number: [-11.1]
fullview: false
shortinfo: 相比较机器学习要显性输入feature，卷积神经网络(Convolutional Neural Networks)可以自己提取feature。本文是Stanford "CS231n： Convolutional Neural Networks for Visual Recognition"的第一篇笔记。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Image Classification Introduction ##

> **Image Classification**：给出1幅图片，在固定的标签(Label)中给出图片中对应物体的概率分布。这个任务定义看起来简单，实现起来却不容易。而且很多Computer Vision task最后都可化约为**Image Classification**问题。

**Image Classification**有如下几个主要难点：

1. **视角变化(Viewpoint variation)**

2. **大小变化(Scale variation)**

3. **变形(Deformation)**

4. **遮盖(Occlusion)**

5. **光照变化(Illumination Conditions)**

6. **背景杂波(Background Clutter)**

7. **类内变化(Intra-class Variation)**

{: .img_middle_lg}
![Image Classification Challenges](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/Image Classification Challenges.jpeg)

### 1.1 Data-driven Approach ###

那么如何实现1个算法来进行**image classification**任务呢。不像通常的排序算法，**image classification**并不能写出1个显示的算法来教计算机如何识别例如猫。我们这里采用的方法和教小孩子学习的方法并无两样。

> **Data-driven Approach**：提供大量对应各个label的数据，来开发一种学习算法用以识别各个label对应数据的的特征。

下图是这种带有label的数据的例子。

{: .img_middle_lg}
![Train Set](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/trainset.jpg)


### 1.2 Pipeline ###

我们已经看到**image classification**是输入代表某一图形的像素的矩阵来获取相应label的过程。一个完整的过程如下。

1. **Input**：输入包括N幅图片，每一幅都被标上了K个标签其中的1个。我们将这个数据对称为**training set**。

2. **Learning**：对**training set**进行学习。我们将此过程称为**训练1个分类器(training a classifier)**。

3. **Evaluation**：最后，我们来评估第二步分类器的准确度，通过对一组从未见过的图片(**test set**)进行label。我们希望绝大多数**test set**能有正确的label，对应的准确度我们称之为高。


### 1.3 一种基本的方法：Nearest Neighbor Classifier ###

> **最近邻分类器(Nearest Neighbor Classifier)**：将test image依次和training set里的每一幅image比较，比较方法是计算各个对应像素之间的**某种形式的距离**并求和，值最小的对应的label就是该test image的label。

下图是CIFAR-10数据库最近邻的一个预测图，右边从左到右是与预测图片最近的前10张图片。

{: .img_middle_lg}
![nn](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/nn.jpg)

**某种形式的距离**有多种可能，比如**L1 distance**，
$$d_1(I_1,I_2) = \sum_p |I_1^p-I_2^p| $$。


{: .img_middle_lg}
![L1 distance.jpeg](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/L1 distance.jpeg)

{% highlight py linenos %}
import numpy as np
import pickle

def unpickle(file):

    with open(file, 'rb') as fo:
        dict = pickle.load(fo)
    return dict

def load_CIFAR10(file):
    results = []
    Xtr = np.zeros([10000,3072])
    Ytr = []
    for i in range(5):
        print("{}/data_batch_{}".format(file,i+1))
        dict = unpickle("{}/data_batch_{}".format(file,i+1))
        Xtr =  np.concatenate((Xtr,dict[b"data"]))
        Ytr = Ytr + dict[b"labels"]
    Ytr = np.array(Ytr)

    dict = unpickle("{}/test_batch".format(file, i + 1))
    Xte = dict[b"data"]
    Yte = dict[b"labels"]

    return (Xtr[10000:],Ytr,Xte,Yte)

class NearestNeighbor(object):
    def __init__(self):
        pass

    def train(self,X,y):
        '''
        X is N*D where each row is an exmaple. y is 1-dimension of size N
        '''
        self.Xtr = X
        self.ytr = y

    def predict(self,X):
        '''
        X is N*D where each row is an exmaple we wish to predict label for
        '''
        num_test = X.shape[0]
        Ypred = np.zeros(num_test, dtype=self.ytr.dtype)

        for i in range(num_test):
            distances = np.sum(np.abs(self.Xtr-X[i,:]),axis=1)
            min_index = np.argmin(distances)
            Ypred[i] = self.ytr[min_index]
        return Ypred

Xtr,Ytr, Xte,Yte = load_CIFAR10("cifar-10-batches-py")
nn = NearestNeighbor()
nn.train(Xtr,Ytr)
Yte_prdict = nn.predict(Xte)
print("accuracy:{}".format(np.mean(Yte_prdict == Yte)))

#该程序跑的时间较久，建议在google cloud的VM实例上跑，8 core CPU跑了近3小时。
#Output
#accuracy: 38.59%

{% endhighlight %}

用最近邻的方法的准确率最后是38.59% on CIFAR-10。这比随机猜测的准确度(10%，因为有10个类)高出不少，但是比人类的准确度94%依然相差巨大，而CNN的准确度可以达到95%，可以和人类完全匹敌。这个最近邻的方法只是让我们感受下**Image Classification**早期的一些思路。

当然对于上文提到**某种形式的距离**还有其他形式，比如**L2 distance**，$$d_2(I_1,I_2) = \sum_p \sqrt {(I_1^p-I_2^p)^2}$$。**L1**和**L2**的比较这里就不展开了，有兴趣的同学可以看[这里](http://cs231n.github.io/classification/)。

#### 1.3.1 K-Nearest Neighbor ####

你可能已经注意到，只用最近的neighbour来预测貌似不大合理。

>**K-Nearest Neighbor**：用最近的K个Neighbor来选出概率最大的label。K=1的情况就是上面的只取最近neighbour的情况。**K-Nearest Neighbor**可以有一个更光滑的预测，对于离群值(outlier)的处理也更平滑。

下图是**K-Nearest Neighbor**的boundary的示意图。其中左图代表training data，中图代表k=1的时候test image在图中位置的不同预测结果的边界，右图表示k=5的时候的预测结果的边界(白色边界表示最近邻里有两个label的数量达成了平手，都是2个)。可以看到k=5的时候，离群值的孤岛被平滑地处理掉了。

{: .img_middle_lg}
![knn](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/knn.jpeg)

#### 1.3.2 Pros/Cons of Nearest Neighbor ####

Pros/Cons:

1. easy to understand and implement.

2. classifier takes no time to train, evaluation needs to compare every single training example.

3. what we want is training time doesn't matter, evaluation time should be as cheaper as possible. CNN will actually fall in this category, which exactally meets our requirements. 

4. In practice, Nearest Neighbor Classifier sometimes may be a good choice in low dimensional data, but not for high dimensional data(such as image, which has many pixels). Since distance over high dimensional would be counter-intuitive.

下图展示的是后三幅图相比于第一幅图的L2 distance都一样的情况，视觉上来看，它们的差别巨大，但是L2算法却给出了相同的距离。可以进一步地说，Nearest Neighbor的本质是给出了像素分布的总差别，而不是图片中物体的high level的特征，因此虽然有38.59%的准确性，但是再往上提高就非常困难了。

{: .img_middle_lg}
![knn](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/samenorm.png)

### 1.4 Hyperparameter and Train/Val/Test Split###

> **Hyperparameter**：上述的**K Nearest Neighbor**要求我们人工设定**K**的值。但是哪个**K**最好我们得怎么确定呢？又比如**某种形式的距离**是选L1，L2还是其他呢，如何确定哪种最好呢？这些参数我们称之为**超参数(hyperparameters)**，在机器学习中非常常见，而且大部分时候选择哪个是最优的并不是那么显而易见。一种选择方式是遍历法，将K从1到n，训练training set，比较test set的准确度，选取最高的那个K。但是要注意，我们只有在最后的时候才能用test set，训练过程中用test set会导致**过度拟合(overfit)**的问题。因为K的选择倾向了test set(将test set也作为了training set)，在具体应用中会降低客观的准确度。那么如何优化**Hyperparameter**呢？我们可以将training set分为稍小一点的training set和**validation set**。用**validation set**来调试超参数。

比如下面代码

{% highlight py linenos %}
# assume we have Xtr_rows, Ytr, Xte_rows, Yte as before
# recall Xtr_rows is 50,000 x 3072 matrix
Xval_rows = Xtr_rows[:1000, :] # take first 1000 for validation
Yval = Ytr[:1000]
Xtr_rows = Xtr_rows[1000:, :] # keep last 49,000 for train
Ytr = Ytr[1000:]

# find hyperparameters that work best on the validation set
validation_accuracies = []
for k in [1, 3, 5, 10, 20, 50, 100]:
  
  # use a particular value of k and evaluation on validation data
  nn = NearestNeighbor()
  nn.train(Xtr_rows, Ytr)
  # here we assume a modified NearestNeighbor class that can take a k as input
  Yval_predict = nn.predict(Xval_rows, k = k)
  acc = np.mean(Yval_predict == Yval)
  print 'accuracy: %f' % (acc,)

  # keep track of what works on the validation set
  validation_accuracies.append((k, acc))
{% endhighlight %}

> **Hyperparameter Optimization**:split your training set into training set and a validation set. Use validation set to tune all hyperparameters. At the end run a single time on the test set and report performance.


> **Cross-Validation**：有时候在training set样本数比较少的时候，将样本分为n份，在某个K值下，轮流取1份作为validation set，将n次的validation的准确度取平均值作为比较依据。

{: .img_middle_lg}
![crossval](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/crossval.jpeg)

## 2 Summary ##

1. 介绍了**Image Classification**问题以及解决该问题的Pipeline。

2. 用一个具体的**Nearest Neighbor Classifier**根据Pipeline来解决**Image Classification**问题，了解解决过程，理解其中的局限。

3. 对于**Hyperparameter**可以用train/val/test的方法来优化，样本数少的时候可以用**Cross-Validation**。

{: .img_middle_hg}
![Image Classification Summary](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/Image Classification Summary.png)

## 3 参考资料 ##

- ["CS231n: Convolutional Neural Networks for Visual Recognition"](http://cs231n.stanford.edu/);
- ["CS231n Course Video 2016 Winter"](https://www.youtube.com/watch?v=g-PvXUjD6qg&index=1&list=PLIUoqCcJd2BjsI11qafvMWv_UqiH1Wu3Q);
- ["Deep Learning"](http://www.deeplearningbook.org/);
- ["CIFAR-10"](https://www.cs.toronto.edu/~kriz/cifar.html);
- ["A Few Useful Things to Know about Machine Learning"](http://homes.cs.washington.edu/~pedrod/papers/cacm12.pdf);
- ["Recognizing and Learning Object Categories"](http://people.csail.mit.edu/torralba/shortCourseRLOC/index.html);








