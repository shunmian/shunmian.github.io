---
layout: post
title: Computer System(一)：Program Part II：Machine-Level Execution
categories: [-00 Computer System]
tags: [Computer System]
number: [-0.2]
fullview: false
shortinfo: 本系列是对《Computer Systems - A Programmer's Perspective》读书总结，作为计算机科学其他课程的基础。本文是第3篇笔记-《Machine-Level Execution》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Machine-Level Execution ##

> **为什么要学汇编？** 理解编译器的优化，分析代码低效的来源，暴露高级语言(如C)隐藏的信息(比如Chapter 12不同并发程序中的变量在内存中的地址)，写出更安全的代码(防止stack
 overflow attack)等。当前对一个专业的程序员的要求已经从**会写**汇编转移到**会读**汇编。因此我们要学会汇编。

> **本章重点**：熟悉IA32指令，会用gcc将.c生成.o生成可执行文件(并反汇编成.o)，会用gdb调试；熟悉C Code Pattern(if else, loop, switch, array, structure, union)；深刻理解pointer，stack overflow attack。

> **Program Encodings**：".c->.s->.o->link后可执行"见[这里]({{site.baseurl}}/00%20c/2014/06/30/A1-Linux-C-%E5%B7%A5%E5%85%B7.html)。

> **C Data Formats in IA32**。所有指针都是4Byte(32Bit)；没加unsigned就是默认是sign，如int a是sign int(4Byte)，unsigned int b是无符号整数(4Byte)。

> **Accessing Information**。寄存器里的值用``%eax``，内存里的值可以间接寻址为``B(%eax,%ecx,A)``，即(x+Ay+B)，表示x+Ay+B地址里的值，其中``%eax``是x，``%ecx``是y；数据转移指令有mov，push，pop等，suffix b w l分布对应byte(1Byte) word(2Byte) double word(4Byte)。

> **Arithmetic and Logical Operations**。``leafl 3(%eax,%ecx,2), %edx`` 是将x+2y+3放入``%edx``，其中``%eax``是x，``%ecx``是y；leafl和movel格式相同，意义不同，前者是放地址，后者是放地址里的值；通常leaf会用于快速的加法和乘法，如``leafl 3(%eax,%ecx,2), %edx``里``%eax``和``%ecx``放的x和y是要计算的变量而不是地址，则结果是x+2y+3就是要的值而不是地址。加减乘除的指令见书本。左移指令sal(算术左移)和shl(逻辑左移)功能一样，右移指令sar(算术右移)和shr(逻辑右移)功能不一样(前者高位填符号位，后者填0)，也就是说只有右移才需要考虑符号位而左移不需要。


## 2 总结 ##

{: .img_middle_hg}
![Machine Level Representation of Programs](/assets/images/posts/2014-07-03-Computer System Part I Program(二)：Machine-Level Representation of Programs/Chapter 3_Machine Level Representation of Programs.png)


## 3 lab ##

本章有两个作业**bomb**(通过反汇编，找到相关信息，需要熟练应用control(swtich)，函数调用，递归等的assembly code pattern)和**bufbomb**(通过buf overflow方法进行攻击，需要熟练掌握函数调用中，call和ret指令，stack变化，及%eax,%ebp,%esp,%eip寄存器的关系)。具体code见[这里](https://github.com/shunmian/00-CSAPP-Labs)。

下面主要对**bufbomb**的5道题(level 0-4)做一个总结。

{: .img_middle_hg}
![bufbomb总结](/assets/images/posts/2014-07-03-Computer System Part I Program(二)：Machine-Level Representation of Programs/bufbomb总结.png)





## 4 Reference ##

- [《Computer Systems - A Programmer's Perspective》](https://www.amazon.com/Computer-Systems-Programmers-Perspective-2nd/dp/0136108040);





