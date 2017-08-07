---
layout: post
title: OC Runtime(A1)：配置Runtime源码环境
categories: [01 Objective-C]
tags: [Sending Message]
number: [0.14.1.2]
fullview: false
shortinfo: runtime在通常的Xcode工程中是用汇编码存储的，因此调试runtime想查看其源码对于汇编不熟悉的同学比较不适应。好消息是苹果开源了objc runtime源码。本文介绍如何自己添加objc runtime源码到Xcode工程中。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Runtime源码 ##

### 1.1 添加 ###

### 1.2 调试 ###

## 2 完整Project ##

对于不想自己一步步调试的同学，这里有1份完整的[Runtime 源码Project](https://github.com/RetVal/objc-runtime)，Clone下来后:

1. manage scheme选``debug-objc``；

2. 在``main``函数随便写几行代码；

3. 添加symbolic breakpoint即可(如``objc_msgSend``)。

## 3 总结 ##

祝玩的开心。


## 3 Reference ##

- [《Runtime Programming Guide》](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html);

- [《ObjC Runtime（二）：配置调试环境》](https://xiuchundao.me/post/config-runtime-debug-environment);

- [《objc - 编译Runtime源码objc4-680》](http://blog.csdn.net/wotors/article/details/52489464);

- [《objc - 编译Runtime源码objc4-706》](http://blog.csdn.net/WOTors/article/details/54426316);

