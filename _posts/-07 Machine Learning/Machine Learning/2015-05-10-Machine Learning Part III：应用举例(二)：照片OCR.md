---
layout: post
title: Machine Learning(三)：应用举例 Part II：照片OCR
categories: [-07 Machine Learning]
tags: [Machine Learning]
number: [-11.1]
fullview: false
shortinfo: 本文通过Photo OCR这个具体例子，我们可以学到:机器学习实际问题的复杂性;如何通过任务分解解决复杂性；以及如何通过天花板分析分配时间来更有效提高机器学习的准确度。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Photo OCR ##

{: .img_middle_hg}
![照片OCR](/assets/images/posts/07_Machine Learning/Machine Learning/2015-05-10/照片OCR.png)


## 2 作业 ##

本节课无作业。

## 3 总结 ##

照片OCR可以用于图片搜索：输入文本，输出相应图片。这个例子展现了实际Machine Learning问题的**复杂性**。通过将任务分解可以将复杂问题转换成相对简单的子问题，比如照片OCR可以分解成**文本检测**，**文本分割**，**文本识别**等三个问题。实现好三个子问题就自动实现好整个问题。在实际时间分配中，通过天**花板分析(用正确输入替代某个pipline部分)**可以获得该pipeline部分若正确可以提高准确率的百分比，从而根据百分比的大小安排时间来最有效的提高最终准确度。

## 4 参考资料 ##

- [《MapReduce》](https://en.wikipedia.org/wiki/MapReduce);

- [《A  Very  Brief  Introduction  to  MapReduce》](http://hci.stanford.edu/courses/cs448g/a2/files/map_reduce_tutorial.pdf);

- [《Apache Hadoop》](https://en.wikipedia.org/wiki/Apache_Hadoop);




