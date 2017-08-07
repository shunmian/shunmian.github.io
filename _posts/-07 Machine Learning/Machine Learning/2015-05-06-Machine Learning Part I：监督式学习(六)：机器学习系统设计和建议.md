---
layout: post
title: Machine Learning(一)：监督式学习 Part VI：监督式学习(六)：机器学习系统设计和建议
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: 如何调试机器学习算法？在判断了是不足拟合和过度拟合的基础上，该如何选择下一步？是增加feature个数，增加training example，还是减小正则化λ值？本文我们来系统介绍如何调试机器学习算法。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 机器学习调试 ##

### 1.1 调试学习算法 ###

#### 1.1.1 调试介绍 ####

> 当你的一个机器学习的算法实现后，你发现error,即$J(\theta)$过大，这个时候，你应该如何选择下一步呢？是扩大训练数据规模还是降低$\lambda$呢?

通常来说我们有以下几种方式可以尝试：

1. 获取更多训练数据；

2. 减少feature；

3. 增加feature；

4. 增加高次项；

5. 降低$\lambda$；

6. 增加$\lambda$。

但是他们之间哪个更有效呢？很多机器学习的实践者通过自己的感觉来随机选择优先级，使得浪费大量时间才发现这一项没有太多用处。比如如果选择扩充训练数据，通常可以占据你6个月以上时间。6个月后才发现我可能只需要增加feature就能解决问题就已经浪费了大量的时间。我们下面介绍几种方法可以帮我们剔除掉这个list中的一半以上的选项并将剩下的选项设置，使得我们可以用更短的时间focus在最有可能解决问题的选项上。


{: .img_middle_lg}
![Feature个数 vs Generalized Error](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/调试算法.png)



#### 1.1.2 哪个error ####


> 评估一个预测函数的准确性，得用error这个指标，即$J(\theta)$。问题是该用$J_{train}(\theta)$还是$J_{test}(\theta)$；如果是后者，如何解决收集额外test数据的时间问题呢？

答案：

1. 是该用$J_{test}(\theta)$。考虑overfit的情况，虽然$J_{train}(\theta)$很小，但是$J_{test}(\theta)$很大，这依然是不准确的预测函数。

2. 将已有数据随机排列；选取前70%作为训练data，后30%作为预测data；我们先用70%的data来获得$J_{train}(\theta)$最小的时候的$\theta$，即预测函数；然后用剩下的30%来计算$J_{test}(\theta)$。

$J_{test}(\theta)$的计算可以分为下面两种情况：

1. 对于线性拟合，$
J_{test}(\Theta) = \frac 1 {2m_{test}} \sum_{i=1}^m (h_{\Theta}(x_{test}^{(i)}-y_{test}^{(i)})^{2}
$

2. 对于分类，
$$
\begin{align}
&J_{test}(\Theta)= \frac 1 {m_{test}} \sum_{i=1}^m err((h_{\Theta}(x_{test}^{(i)},y_{test}^{(i)}) 
\\
&\text{ where $err(h_{\Theta}(x_{test}^{(i)},y_{test}^{(i)}) =
\begin{cases}
1, &\text{if $h_{\Theta}(x_{test}^{(i)} ≥ 0.5 $ and $y=0$ or $h_{\Theta}(x_{test}^{(i)} < 0.5 $ and $y=1$  } \\
0, &\text{otherwise} \\
\end{cases}$}
\end{align}
$$

#### 1.1.3 哪个error：考虑Feature个数 ####

> 如果用1-10个不同feature个数来获取$J_{train}(\theta)$最小时候的$\theta$，用哪个error来选择最优feature个数呢？如果用$J_{test}(\theta)$会有什么问题，为什么会有这个问题；如果不用该如何解决。

答案：

1. 用$J_{test}(\theta)$来选取feature个数的问题在于相当于将test data用来fit feature个数。因此在此基础上算出来的$J_{test}(\theta)$并不能作为未来崭新的数据的预测的评估。

2. 用Train(60%)/Validation(20%)/Test(20%)来分割数据。用Train(60%)来计算不同feature个数下的$J_{train}(\theta)$最小时候的$\theta$；然后代入Validation(20%)，选出最小的$J_{cv}(\theta)$下的feature个数；最后用Test(20%)的数据的$J_{test}(\theta)$作为对未来数据预测的评估。

> 对于未来数据预测的评估，test data要独立出来，不能作为预测函数fit的一部分。

{: .img_middle_hg}
![哪个error_考虑Feature个数](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/哪个error_考虑Feature个数.png)


### 1.2 Bias vs. Variance ### 

> **High Bias**和**High Variance**是机器学习算法**error**过大的唯一两个原因。判断对了，就可以减少之前list中6个方法的一半。判断的途径有多种，请看下面。

#### 1.2.1 error vs. feature个数 ####

> 如何用**error vs. feature个数**诊断**Bias/Variance**?

{: .img_middle_hg}
![Feature个数和baisvariance](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/Feature个数和baisvariance.png)

#### 1.2.2 error vs. $λ$ ####

> 如何用**error vs. $λ$**诊断**Bias/Variance**?

正则化是用来解决overfitting的问题，但是$λ$过大会导致underfitting问题，$λ$过小会有overfitting问题。如何选择$λ$的大小呢。

{: .img_middle_hg}
![lambda和baisvariance](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/lambda和baisvariance.png)

#### 1.2.3 error vs. training number (learning curve)####

> 如何用**error vs. training number**诊断**Bias/Variance**?

{: .img_middle_hg}
![lambda和baisvariance](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/traning number和baisvariance.png)

>在什么情况下，收集更多的traning example可以降低$J_{CV}$？答案是：在**High Variance**的情况下而不是**High Bias**的情况下。

### 1.3 调试小结 ###

我们首先介绍了error的种类，**training(70%)/test(30%)**到**training(60%)/validation(20%)/test(20%)**；然后如何用**feature个数**，**$λ$**，**training number**画error曲线，且用该曲线来判断是**Bias**和**Variance**问题。判断对了后，上面list中的6个选项就可以减少一半，供我们进一步优化算法，降低error。

## 2 机器学习系统设计 ##

### 2.1 垃圾邮件分类器 ###

面对一个崭新的问题，例如用机器学习编写一个垃圾邮件分类器，有许多方式，例如：

1. 收集大量数据(例如，“honeypot”项目来用大量垃圾邮件的发送地址作为数据，但是并不是总是work well)；

2. 设计复杂的features(例如，用email header data)；

3. 设计算法来处理输入(例如，识别spam邮件中的错误拼写)。

但是很难分别哪一种更有效。

### 2.2 错误分析 ###

有一个建议的方法来处理机器学习问题如下：

1. 用一个简单的算法开始，快速实现它，快速调试它；

2. 画出学习曲线来决定应该增加数据(High Variance)，还是增加feature(High Bias)；

3. **错误分析**：人工检查cross validation data的errors的来源。

用一个单一的数字来衡量error很重要，不然很难评估算法的性能。



### 2.3 错误矩阵(倾斜类情况) ###

> 考虑一种情况，肿瘤的分类(良性和恶性)。假如你的算法的错误率为1%；而实际恶性肿瘤是0.5%，意味着若一个算法简单输出$y=0$在任何$x$的情况下，那么算法的错误率为0.5%。那么该用什么更准确的指标来替代通常的error呢？这种情况称为skewed classes(倾斜类，某一类的概率远大于另一类)

答案是：用**Error Metrics**。

{: .img_middle_lg}
![Error Metrics](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/Error Metrics.png)

> 将Precision和Recall合成一个指标：F1。

{: .img_middle_lg}
![F1](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/F1.png)

### 2.4 算法比较 ###

> 一个劣势的算法，如果给予充足的训练数据，其性能(error)可以强于一个优势算法(在少数据情况下的优势算法)。

{: .img_middle_mid}
![Algorithm Compare](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/Algorithm Compare.png)

## 3 作业 ##

见[这里](https://github.com/shunmian/-11-Machine-Learning)。附上一张跑分图。


{: .img_middle_lg}
![assignment5](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/assignment5.png)

## 4 总结 ##

我们首先介绍了error的种类，**training(70%)/test(30%)**到**training(60%)/validation(20%)/test(20%)**；然后如何用**feature个数**，**$λ$**，**training number**画error曲线，且用该曲线来判断是**Bias**和**Variance**问题。判断对了后，下面list中的6个选项就可以减少一半，供我们进一步优化算法，降低error。

最后我们讨论了**机器学习算法系统设计**，对于其中错误的定义，用**Error Metrics**结合**F1**来更准确的界定机器学习的性能指标。

{: .img_middle_hg}
![调试总结](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-06/调试总结.png)


## 5 参考资料 ##

- [《Deep Learning》](http://deeplearning.net/);
- [《Microsoft Azure Machine Learning》](https://azure.microsoft.com/en-us/services/machine-learning/);
- [《A Visual Introduction to Machine Learning》](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/);





