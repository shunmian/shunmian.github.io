---
layout: post
title: 网络实战(一)：传输层 Part I：Introduction to Client-Server
categories: [-02 Networks]
tags: [Networks, Client-Server]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第1篇笔记《Client-Servre介绍》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Introduction to Client-Server Networking ##

第一章用了两种Layer中的4个库，由高级到低级，一层层来实现如何使用Google的Geocoder服务，即写一个Client：

1. Application Layer。(1) pygeocoder; (2)requests; (3)http。这三种都是建立在HTTP协议上。

2. Socket Interface。

更严格的说，Socket不是计算机网络中的5层模型中的任何一层，它是Application层和Transport层之间的interface。

{: .img_middle_hg}
![chapter 1 summary](/assets/images/posts/2015-04-02/chapter 1 summary.png)

有3点需要注意：

1. json的使用。json可以理解为一种特殊的dict(拥有特定格式)，它可以与string互换。

2. string vs. bytes。bytes是8个位的bit，是字节，它和string可以相互编码和解码。

3. 在Network中传输的数据主角是bytes。`HTTPResponse的read()`和`socket的recv()`返回都是bytes。

{: .img_middle_hg}
![bytes](/assets/images/posts/2015-04-02/bytes.png)

第一章[code](https://github.com/shunmian/-8.1-Foundations-of-Python-Networking-Programming)。







## 2 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);

- [《Web Scraping Part II：Advanced Scrapers (三)：表单和登录》]({{site.baseurl}}/web%20scraping/2015/12/09/Web-Scraping-Part-II-Advanced-Scrapers-(%E4%B8%89)-%E8%A1%A8%E5%8D%95%E5%92%8C%E7%99%BB%E5%BD%95.html);




