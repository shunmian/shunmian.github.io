---
layout: post
title: OC Runtime(零)：Runtime概述
categories: [01 Objective-C]
tags: [Sending Message]
number: [0.14.1.2]
fullview: false
shortinfo: runtime是objective-c的operating system，是如何在c的基础上搭建OOP语言的基石。本文是《OC Runtime》系列的概述，对runtime做一个全局的介绍。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Runtime 概述 ##

### 1.1 什么是Runtime ###

如果用一句话介绍什么是Objective-C，下面这个公式再恰当不过了。

> OC = C + Preprocessor + Runtime

三十几年前，Brac Cox为了在C语言基础上扩展OOP，借用了Smalltalk的思想，开发了Objective-C语言(参考[《Object-Oriented Programming: An Evolutionary Approach》](https://www.amazon.com/Object-Oriented-Programming-Evolutionary-Brad-Cox/dp/0201548348)）进行了介绍。为了便于parse类似Smalltalk的语法，发明了中括号的消息发送``[Dog name:name]``，然后通过一个Preprocessor预处理成C语言。虽然语法进行了转换，但是如何在此基础上搭建OOP模型呢，即：

1. 实现OOP对象模型(Preprocessor)；
2. 实现类的消息发送(Runtime)。

**对象模型**直接照搬Smalltalk的设计思想就可以，即Object/Class/MetaClass等，直接用C的Struct实现即可；以及某些self/super/nil等关键字的实现。上述步骤通过Preprocessor即可实现。但是Instance Method/Class Method这些消息机制，通过向target(instance/class)发送消息名(selector)动态寻找到函数的实现(IMP)，以及向父类传递消息和消息转发等，并不能通过Preprocessing或Build Time实现，需要通过运行时的C函数支持，这些函数打包一起，便是Runtime。

### 1.2 Runtime大局观 ###

Runtime主要有**Object Model**和**Messaging**两大主题组成，笔者总结成下表，后面的文章会详细介绍。

{: .img_middle_hg}
![Runtime Object Model & Messaging](/assets/images/posts/01 Objectiev C/2016-03-12-OC Runtime(零)_runtime/Runtime Object Model & Messaging.png)

### 1.3 Runtime API ###

熟悉runtime的API有助于后面文章的学习，笔者将它总结成下图。

{: .img_middle_hg}
![Runtime API](/assets/images/posts/01 Objectiev C/2016-03-12-OC Runtime(零)_runtime/Runtime API.png)

### 1.4 Runtime使用场景 ###

通常我们有3个level来使用Runtime：

1. **Objecttive-C Source Code**，大部分情况下，runtime在OC代码背后自动执行，开发者只需要写可以编译通过的OC源码即可。

2. **NSObject Methods**，大部分类在Cocoa里都是``NSObject``的子类，因此，它们都继承了``NSObject``的methods(``NSProxy``是例外)。这些方法包括``description``，``isKindOfClass``，``isMemberOfClass``，``respondsToSelector``，``conformsToProtocol``，``methodForSelector``等。这些方法给object自省(introspect)的功能。（顺便吐槽下，人如果有自省功能就好了，知道自己可行的行为，父类；或者说人有更高一级自省的功能，就是在不断探索的工程中发现认识自己。）

3. **Runtime API**，runtime头文件在``/usr/include/objc``路径下，包括了各种数据结构和方法。它可以让开发者用c语言重复编译器对OC源码进行的工作。具体请参看[《Runtime Programming Guide》](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)。

## 2 总结 ##

本文对runtime从整体上进行了介绍，包括runtime在OC中的角色，两大主题(Object Model & Messaging)和实现的API，以及Runtime的使用场景。


## 3 Reference ##

- [《Runtime Programming Guide》](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html);

- [《Understanding the Objective-C Runtime》](http://cocoasamurai.blogspot.hk/2010/01/understanding-objective-c-runtime.html);

- [《重识 Objective-C Runtime - Smalltalk 与 C 的融合》](http://blog.sunnyxx.com/2016/08/13/reunderstanding-runtime-0/);



