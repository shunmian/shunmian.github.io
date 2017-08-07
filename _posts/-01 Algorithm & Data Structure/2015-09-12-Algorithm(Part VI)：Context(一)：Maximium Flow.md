---
layout: post
title: Algorithm(六)：Context Part I：Maximium Flow
categories: [-01 Algorithm]
tags: [Graph]
number: [-2.2.4]
fullview: false
shortinfo: 图是重要的一种数据结构，能巧妙优雅地解决许多其他数据结构和算法不能解决的问题。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 最大流 ##

### 1.1 介绍 ###

{: .img_middle_hg}
![1.1 introduction](/assets/images/posts/01_Algorithm/2015-09-12_Algorithm(Part VI)：Context(一)：Maximium Flow/1.1 introduction.png)

### 1.2 Ford-Fulkerson Algorithm ###

{: .img_middle_hg}
![1.1 introduction](/assets/images/posts/01_Algorithm/2015-09-12_Algorithm(Part VI)：Context(一)：Maximium Flow/1.2 Ford-Fulkerson Algorithm.png)

需要注意的是backward edge指某个和foward方向一样的edge B共同进入某个vertext的edge A。此时B flow的减少就能增加A flow，因此B称为A的backward edge。

### 1.3 maxflow-mincut theorem ###

{: .img_middle_hg}
![1.3 maxflow-mincut theorem](/assets/images/posts/01_Algorithm/2015-09-12_Algorithm(Part VI)：Context(一)：Maximium Flow/1.3 maxflow-mincut theorem.png)

### 1.4 running time analysis ###

{: .img_middle_hg}
![1.4 running time analysis](/assets/images/posts/01_Algorithm/2015-09-12_Algorithm(Part VI)：Context(一)：Maximium Flow/1.4 running time analysis.png)

### 1.5 Java implementation ###

{: .img_middle_hg}
![1.5 Java implementation](/assets/images/posts/01_Algorithm/2015-09-12_Algorithm(Part VI)：Context(一)：Maximium Flow/1.5 Java implementation.png)


### 1.6 Application ###

{: .img_middle_hg}
![1.6 Application](/assets/images/posts/01_Algorithm/2015-09-12_Algorithm(Part VI)：Context(一)：Maximium Flow/1.6 Application.jpg)


## 2 总结 ##




## 3 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);





