---
layout: post
title: CNN for VR Part III：CNN(一)：Architectures, Convolution/Pooling Layers.
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



### 1.1 Nearest Neighbor Classifier ###

#### 1.1.1 k-Nearest Neighbor ####

### 1.2 Train/Val/Test Split ###

### 1.3 Pros/Cons of Nearest Neighbor ###

## 2 Summary ##

### 2.1 ###

### 2.2 Applying kNN in practice ###

{: .img_middle_lg}
![Image Classification Challenges](/assets/images/posts/07_Machine Learning/Convolutional Neural Network/2015-06-01-CNN for Visual Recognition Part I：入门(一)：图片分类/Image Classification Challenges.jpeg)

repeat until convergence：

$$
\theta_0 := \theta_0 - \alpha \frac 1 m \sum_{i=1}^m[h_\theta(x^{(i)})-y^{(i)}]
$$


$$
\theta_1 := \theta_1 - \alpha \frac 1 m \sum_{i=1}^m \lbrace[h_\theta(x^{(i)})-y^{(i)}]x^{(i)}\rbrace
$$



## 6 参考资料 ##
- ["CS231n: Convolutional Neural Networks for Visual Recognition"](http://cs231n.stanford.edu/);
- ["CS231n Course Video 2016 Winter"](https://www.youtube.com/watch?v=g-PvXUjD6qg&index=1&list=PLIUoqCcJd2BjsI11qafvMWv_UqiH1Wu3Q);
- ["Deep Learning"](http://www.deeplearningbook.org/);