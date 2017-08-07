---
layout: post
title: Algorithm(五)：String Part I：Sorting
categories: [-01 Algorithm]
tags: [Graph]
number: [-2.2.4]
fullview: false
shortinfo: String Sorting的下限是否可以比第二章的通过比较的Sorting的O(NlogN)做的更好，又是String的什么特质导致这种不同呢。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}



## 1 String Sorting ##

### 1.1 Counting Sorting  ###

计数排序的算法下界是**O(N)**。这是如何做到的呢？我们在第二章知道通过两两比较``compareTo()``的排序，决策树模型告诉我们最少需要**NlogN**次比较。显然计数排序不是通过比较来实现的。它是通过字母作为数组下标``key()``来实现的。

通俗的说，N个元素，有N!种组合，其中只有1种组合符合升序。我们之前的比较排序，都是尽可能1次比较降低一半组合次数成${ N! \over 2 } $种。因此，最多需要h次比较，使得${ 2^h > N! } $，根据斯特灵公式，就有$h >NlogN $。而计数排序就好像桌子上有13个升序的位置，对应A,2,3,...,Q,K的扑克牌。给你一张牌，你放入对应的位置；13张不同的牌只需要13次放入就可以排成升序。本来13张牌有13!种组合，给第1张牌，你立马确定了13个位置中的某个位置，因此将可能性降为$ {13! \over 13 } = 12!  $；给第2张牌，可能性降为$ {12! \over 12 } = 11! $；直到最后一张牌,唯一的可能性确定。因此计数排序由于提前建立了被排序的字母相互之间的大小和应该去的位置(即，字母作为数组下标，对应上面就是桌上的牌的对应空位)，使得每个字母都只需要一次取``key()``就能找到对应的位置，从而比比较排序更有效，将比较排序的O(NlogN)的下界将低为O(N)。

计数排序的代码如下。

{: .img_middle_hg}
![Counting Sorting](/assets/images/posts/01_Algorithm/2015-09-09_Algorithm(Part V)：String(一)：Sorting/1.1 Counting Sorting.png)

什么情况下可以用counting sorting？

### 1.2 LSD(Least Sigfinicant Digits) Radix Sorting  ###

以上是对于1位的计数排序，对于String字符串有M个字符来说，需要依次对每一位进行计数排序。那么问题来了，该依据什么顺序对每一位进行计数排序呢，String的长度需要一样吗？

> **LSD(Least Sigfinicant Digits) Radix Sorting**:从最小位(因为字母的ASCII码等同于数字),即右边开始向左边，依次进行计数排序。

{: .img_middle_hg}
![LSD](/assets/images/posts/01_Algorithm/2015-09-09_Algorithm(Part V)：String(一)：Sorting/1.2 LSD.png)

计数排序需要所有String拥有相同的长度。

### 1.3 MSD(Most Sigfinicant Digits) Radix Sorting  ###

> **MSD(Most Sigfinicant Digits) Radix Sorting** : 从最大位,即左边开始向右，依次进行计数排序。MSD不需要String拥有相同的长度。

{: .img_middle_hg}
![MSD](/assets/images/posts/01_Algorithm/2015-09-09_Algorithm(Part V)：String(一)：Sorting/1.3 MSD.png)

MSD的缺点是当subarray很小的时候，创建``int[] count``很昂贵，这个时候可以用insertion sorting来cutoff。

MSD和快排很类似，最高位先作为pivot，来partition，然后pivot左边，中间，和右边分别递归调用这个过程。


### 1.4 3-way Radix Quicksort  ###

> **3-way String Quicksort**:Do 3-way partitioning on the $ d_th $ character. Less overhead than R-way partitioning in MSD string sort.

{: .img_middle_hg}
![MSD](/assets/images/posts/01_Algorithm/2015-09-09_Algorithm(Part V)：String(一)：Sorting/1.4 3-way string quicksort.png)


### 1.5 Suffix Arrays  ###


{: .img_middle_hg}
![Longest Repeated String](/assets/images/posts/01_Algorithm/2015-09-09_Algorithm(Part V)：String(一)：Sorting/1.5.0 Longest Repeated String.png)

{: .img_middle_hg}
![Manber-Myers MSD algorithm](/assets/images/posts/01_Algorithm/2015-09-09_Algorithm(Part V)：String(一)：Sorting/1.5 Manber-Myers MSD algorithm.png)

### 1.6 Summary ###

{: .img_middle_hg}
![String Sorting Summary](/assets/images/posts/01_Algorithm/2015-09-09_Algorithm(Part V)：String(一)：Sorting/1.6 String Sorting Summary.png)


## 2 总结 ##





## 3 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);





