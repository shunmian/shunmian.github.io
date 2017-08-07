---
layout: post
title: Computer System(零)：Overview 
categories: [-00 Computer System]
tags: [Computer System]
number: [-0.0]
fullview: false
shortinfo: 本系列是对《Computer Systems - A Programmer's Perspective》读书总结，作为计算机科学其他课程的基础。本文是第1篇笔记-概述。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Overview ##

本书通过全面而深入地揭示程序背后运行的原理，来帮助程序员从一个普通的码农成长成一位编程老司机。如果说cs的书本你只能读一本的话，那么大部分人心中会有这本书。

我们从最简单的Hello world开始。

{% highlight c linenos %}
#include <stdio.h>

int main()
{
	printf("hello, world\n");
}

{% endhighlight %} 

### Part I Program ###

#### 1 Data(bits + encoding)  ####

hello.c文件通过bit的格式以ASCII编码(Text File)存储在disk上，如下图所示。

{: .img_middle_lg}
![Data](/assets/images/posts/2014-07-01-Computer System：Overview/Data.png)

#### 2 Compilation to Machine Code ####

hello.c通过人类可以阅读的文本存储c代码(高级语言)，为了让机器阅读，必须将其转化成机器码(低级语言)。这个转换的过程称之为编译，见下图。

{: .img_middle_lg}
![Compilation](/assets/images/posts/2014-07-01-Computer System：Overview/Compilation.png)

正确理解Compliation各个阶段背后的原理是提高编程水平的必经阶段。它可以提供以下几个好处：

1. **优化系统性能**。我们不需要写机器码，但是读懂机器码可以帮助我们判断是switch还是if，while还是for, 局部变量还是引用传递，指针解引用还是数组下标，等哪个更有效。

2. **理解link-time error**。"cannot resolve a reference"意味着什么，static和global variable有哪些不同，如果定义两个相同名字的全局变量在不同的c文件里会出现什么后果，为什么有些link-time error直到运行时才出现？

3. **避免security holes**。深刻理解stack overflow发生的机制。

#### 3 Running Machine Code ####

##### 3.1 Processor #####

当在terminal输入下面命令后，屏幕会输出"Hello World"。

{% highlight c linenos %}
unix> ./hello
{% endhighlight %} 

这个过程看似简单和快速，后面涉及了非常复杂的硬件协作。我们将其简单总结成下图。

{: .img_middle_hg}
![Run hello](/assets/images/posts/2014-07-01-Computer System：Overview/Run hello.png)

##### 3.2 Caches & Storage Hierarchy #####

上面运行hello文件的过程中，大部分时间是花在将Data从一个地方运送到另一个地方。离processor越近，则被processor读写的速度越快。因此根据速度从快到慢，存储类型从多级Cache memeory，到main memeory，到disk排列。具体见下图。

{: .img_middle_lg}
![Memory Hierarchy](/assets/images/posts/2014-07-01-Computer System：Overview/Memory Hierarchy.png)

### Part II: Program on Operating System ###

#### 1 Operating System ####

操作系统抽象了硬件，作为application和hardware之间的界面，主要有以下2个作用：

1. 保护硬件免受Application破坏；

2. 给Application提供硬件的界面。

操作系统主要抽象了I/O, main memory和Processor，见下图。

{: .img_middle_hg}
![Operating System](/assets/images/posts/2014-07-01-Computer System：Overview/Operating System.png)

#### 2 Networks ###

Network is another I/O(File) between different systems.

{: .img_middle_lg}
![Network](/assets/images/posts/2014-07-01-Computer System：Overview/Network.png)

### Part III: Important Themes ###

#### 1 Concurrency and Parallelism ####

Concurrency指system能同时执行多个任务。Parallelism指应用Concurrency让system运行更快。
Parllelism包括3个level，从高到低分别是：

1. **Thread-Level Concurrency**. 在同一个proccess里面多线程Concurrency。

2. **Instruction-Level Concurrency**. Processor execute multiple instructions at one time.

3. **Single-Instruction, Multiple-Data(SIMD) Concurrency**. 运用vector。

#### 2 Virtual Machine: abstraction of computer ####

最后我们增加一层对整个computer的抽象-虚拟机。

{: .img_middle_lg}
![Virtual Machine.png](/assets/images/posts/2014-07-01-Computer System：Overview/Virtual Machine.png)


## 2 总结 ##

{: .img_middle_lg}
![Virtual Machine.png](/assets/images/posts/2014-07-01-Computer System：Overview/Chapter 1 Overview Summary.png)

## 3 Reference ##

- [《Computer Systems - A Programmer's Perspective》](https://www.amazon.com/Computer-Systems-Programmers-Perspective-2nd/dp/0136108040);





