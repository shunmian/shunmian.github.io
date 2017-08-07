---
layout: post
title: Regular Expression
categories: [-09 String]
tags: [Regular Expression,String]
number: [-5.1.1]
fullview: false
shortinfo: Regular Expression(正则表达式)是匹配一系列匹配某个句法规则的字符串，大大简化了对于String的搜索操作。但是它的使用通常令人费解并且不好掌握。本文我们来系统梳理一遍Regular Expression，为我们以后的字符串搜索打下基础。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

**A popular quote by Jamie Zawinski**: “Let’s say you have a problem, and you decide to solve it with regular expressions. Well, now you have two problems.”

> **正则表达式(Regular Expression)**：匹配一系列匹配某个句法规则的字符串。每一个正则表达式里的字符(Token)要么是元字符(metacharacter)，要么是字面字符(with its literal meaning)。它们一起组成了匹配的规则。

正则表达式的基本概念分为：

1. 字符类；
2. 锚点(或者称为定位点)；
3. 转义序列；
4. 组与环顾；
5. 量词与替代。

以下内容来自于[regexr](http://www.regexr.com/)的总结。

## 1 Character Classes ##

> 字符类Token(Character Classes)：匹配某一**Token**表示的**集合(set)**里的所有**字符元素**。

常用的有6种：

1. ``.`` 表示任意字符，但不包括换行。

2. ``\w````\d````\s`` 分别表示word(26个大小写字母加数字加下划线即[A-Za-z0-9_])，digit[0-9]，whitespace。

3. ``\W````\D````\S`` 分别表示非word,非digit和非whitespace。

4. ``[abc]``表示a或b或c。

5. ``[^abc]``表示非a或非b或非c

6. ``[a-g]``表示a到g

## 2 Anchors ##

> 定位符Token(Anchors)：匹配字符串里的**位置**，而不是**字符**。

主要有两种：

1. ``^``表示字符串开头；``$``表示字符串结尾。

2. ``\b``表示字符串边界；``\B``表示字符串非边界。

## 3 Escaped Characters ##

> 转义序列Token(Escaped Characters)：以``\``作为转义字符，将有特殊含义的字符转换成其字面值。

主要有两种：

1. ``\.``表示.；``\|``表示|；``\\``表示]\。

2. ``\t``表示tab；``\n``表示换行；``\r``表示回车。关于换行和回车的区别见[这里](http://www.ruanyifeng.com/blog/2006/04/post_213.html)，简单来说就是以前在打印的时候回车表示打印头移到最左边，换行表示打印纸往上移一行。在现在的计算机里，两者的存在只是延续了传统，很多时候换行和回车都合并为换行了。


## 4 Groups & Lookaround ##

> 组合Token(Groups)：表示将多个字符当做一个整体一起处理。

主要包括3种：

1. ``(abc)``表示abc为一个整体。

2. ``\1``表示反向引用第1个group(其中1可以改成任意数字)。

3. ``(?:abc)``表示不将abc为一个整体；

> 环顾(Lookaround)：表示将组合作为匹配条件，但是不包括在结果里。

主要有两种：

1. ``(?=abc)``表示匹配组合的abc，但是结果里不将其包括。

2. ``(?!=abc)``表示匹配组合的非abc，但是结果里不将其包括。

## 5 Quantifiers & Alternation ##

> 量词Token(Quantifiers)：表示之前的字符出现的次数。

主要有3种：

1. ``a*``表示出现≥0次a；``a+``表示出现≥1次a；``a？``表示出现0或1次a。

2. ``a{5}``表示出现5次a；``a{2,}``表示出现≥2次a；``a{1,3}``表示出现1至3次a。

3. ``a+?``表示出现≥0次a的最少个数，即0个a；``a{2,}?``表示出现≥2次a的最少个数，即2个a。

> 替代Token(Alternation)：表示或。

1种：

1. ``a|b``表示a或者b。





## 6 总结 ##

最后用一张图对**Regular Expression**进行总结。


{: .img_middle_hg}
![regular expression](/assets/images/posts/2015-08-01/Regular Expression Summary.png)



## 7 参考资料 ##
- [《Regular Expression》](https://en.wikipedia.org/wiki/Regular_expression);





