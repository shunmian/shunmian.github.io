---
layout: post
title: Category Theory(一) Category：the essence of composition
categories: [Category Theory]
tags: [Functional Programming]
number: [-2.1]
fullview: false
shortinfo: 要学好函数式编程，离不开两个数学理论，一个是λ-Calculus，一个是Category Theory。前者从最小表达式的角度来合成所有的计算，后者从函数的组合角度来抽象函数。本系列是由于学习Coursera上Martin Oderskin的“Principles of Reactive Programming”中涉及到monoids, functor和monoid等Category Theory的概念，从而想从根本上了解函数式编程的数学基础而做的笔记。主要内容来自Bartosz Milewski的Category Theory的一系列文章。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. 什么是函数式编程 ##

{: .img_middle_lg}
![Programming Paradigm](/assets/images/posts/2015-10-01/Programming Paradigm.png)

编程范式的发展有两个正交的方向，一个是从**数据和函数的封装**的角度(Non-OOP to OOP), 另一个是**数据和函数的关系**角度(命令式编程(函数围绕数据)和函数式编程(数据围绕函数))。

命令式编程和函数式编程的区别有如下几点：


## 3 Assignment ##

{: .img_middle_lg}
![Assignment](/assets/images/posts/2015-10-01/assignment.png)

+ **Pascal triangle**：递归比较简单，退出条件也很明显，分别是`c==0`和`c==r`的时候；
+ **Parenthesis Balance**：用一个`acc：Int`来存储，每遇见一个`{`+1,`}`-1。退出条件是`chars.isEmpty`或者`acc < 0`；
+ **Counting Money**稍微需要仔细思考，退出条件是`Money == 0`，和`Money < 0 || coins.isEmpty`。

具体代码见[这里](https://github.com/shunmian/-2_Functional-Programming-in-Scala)。


## 4 总结 ##

本节我们首先对什么是函数式编程做了一个总结：

1. 函数无副作用；
2. 通过函数转换不可变数据而非改变原有数据来获取新数据；
3. 数据围绕函数而非函数围绕数据，函数也是一等公民。

在这基础上，我们回顾了作为函数式编程理论基础的**λ-Calculus**。**λ-Calculus**里最重要的3个概念是**&lt;expression&gt;**，**&lt;function&gt;**， **&lt;application&gt;**。**&lt;function&gt;**就是函数，**&lt;application&gt;**可以理解成函数调用。以函数为基础的**λ-Calculus**和以操作为基础的**图灵机**在计算能力上被证明是等价的，任何计算形式都可以被他们实现。现在的**函数式编程**和**命令式编程**正是基于**λ-Calculus**和**图灵机**发展过来。函数式编程的未来发展方向是和OOP结合，这也就是Scala背后的设计哲学。

对于函数，通过函数和变量的声明与定义的比较，可以看出他们同样拥有**关键词**，**名称**，**类型**，**字面值**，因此有理由在一个更抽象的层面将函数和变量统一对待。函数中**类型 + 数据(其实是函数体)**就是**匿名函数**，也就是**Block**。


## 5 参考资料 ##
- [《Structure and Interpretation of Computer Programs》](https://mitpress.mit.edu/sicp/full-text/book/book.html);
- [Martin Odersky: Scala with Style](https://www.youtube.com/watch?v=kkTFx3-duc8);
- [SF Scala: Martin Odersky, Scala -- the Simple Parts](https://www.youtube.com/watch?v=ecekSCX3B4Q);


