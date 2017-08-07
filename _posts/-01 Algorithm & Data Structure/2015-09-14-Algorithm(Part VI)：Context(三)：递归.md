---
layout: post
title: Algorithm(六)：Context Part III：递归
categories: [-01 Algorithm]
tags: [Graph]
number: [-2.2.4]
fullview: false
shortinfo: 递归是算法中影响极其深远的一种思想，所以另开一篇文章谈谈递归。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 递归 ##

## 1.1 穷举 ##

穷举通常还有另一个词叫暴力(Brute Force)。是费时最长但又考虑所有情况的方法。

## 1.2 回溯(backtracking, 图的DFS) ##

回溯在问题的解空间树种，按**深度优先**策略，从根节点触发搜索空间树。算法搜索至解空间树的任意一点时，先判断该节点是否包含问题的解。如果肯定不包含，则结束对该节点为根的子树的搜索(这路口不对，不宜深入，立即返回上个路口)，逐层向祖先节点回溯；否则，进入该子树，继续按深度优先策略搜索。

因此回溯是做最坏的打算(穷举)，见机行事。

## 1.3 递归(recursion) ###

递归和回溯穷举个人认为不能比较。回溯和穷举是一种思想，递归是实现的具体方式。穷举可以用递归或者循环来实现，回溯也可以用递归或者循环来实现。只不过递归看起来更简洁，清晰，是表示复杂procedure的一种非常有效的方式，符合人脑的逻辑推理。更本质的说，递归是组织procedure的一种方式，是循环的近义词(两者都提取了通用的pattern)，只不过前者调用了自己通过不断向base靠拢来通过return返回，后者通过主程序里的i不断向边界靠拢返回。

### 1.3.1 递归形状 ###

#### 1.3.1.1 线性递归 ####

#### 1.3.1.1.1 单递归 ####

#### 1.3.1.1.1 互递归 ####

#### 1.3.1.2 树递归 #####

### 1.3.2 单个递归和调用栈的关系 ###

#### 1.3.2.1 非尾递归 ###

#### 1.3.2.2 尾递归 ####


{: .img_middle_hg}
![1.2 Designing Algorithm.png](/assets/images/posts/01_Algorithm/2015-09-13_Algorithm(Part VI)：Context(二)：Reduction/1.2 Designing Algorithm.png)


### 1.3 Establishing Lower Bounds ###

### 1.4 Classifying Problems ###

## 5 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);





