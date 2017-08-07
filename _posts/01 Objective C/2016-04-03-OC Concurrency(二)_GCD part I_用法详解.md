---
layout: post
title: OC Concurrency(二)：GCD Part I：用法详解
categories: [01 Objective-C]
tags: [Sending Message]
number: [01.25.1]
fullview: false
shortinfo: Objective-C在NSThread的基础上进一步抽象出tasks之间执行的关系(串行和并行)为队列，以及tasks被执行时和Caller的关系(sync和async)为sync和async函数。这种更高一级的抽象使得开发者从NSThread的复杂的life cycle的管理中抽离出来，专注于业务逻辑(tasks)。而NSThread的复杂的life cycle的管理则交由底层系统来完成。使得多线程在Objective-C抽象程度更高，应用也更简单。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 GCD Framework Overview

### 1.1 Managing Dispatch Queues ###

### 1.2 Managing Units of Work ###

### 1.3 Prioritizing Work and Specifying Quality of Services ###

### 1.4 Using Dispatch Groups ###

### 1.5 Using Dispatch Barriers ###

### 1.6 Using Dispatch Data ###

### 1.7 Using Dispatch Time ###

### 1.8 Managing Dispatch Source ###

### 1.9 Managing Dispatch I/O ###

### 1.10 Working with Dispatch Objects ###

### 1.11 Protocols ###

### 1.12 Dispatch Functions ###


## 2 总结 ##

{: .img_middle_hg}
![Chapter 3.2 Data Processing Summary](/assets/images/posts/01 Objectiev C/2016-04-03-OC Concurrency(二)_GCD part I_用法详解/GCD总结.png)

## 3 Reference ##

- [《Grand Central Dispatch In-Depth: Part 1/2》](https://www.raywenderlich.com/60749/grand-central-dispatch-in-depth-part-1);

- [《Grand Central Dispatch In-Depth: Part 2/2》](https://www.raywenderlich.com/63338/grand-central-dispatch-in-depth-part-2);

- [《Computer Systems - A Programmer's Perspective》](https://www.amazon.com/Computer-Systems-Programmers-Perspective-2nd/dp/0136108040);
