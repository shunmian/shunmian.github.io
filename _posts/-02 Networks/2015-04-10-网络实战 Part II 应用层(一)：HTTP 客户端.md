---
layout: post
title: 网络实战(二)：应用层 Part I：HTTP 客户端
categories: [-02 Networks]
tags: [Networks, HTTP]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第9篇笔记《HTTP 客户端》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

从本篇笔记开始，我们就从传输层来到应用层。首先介绍的是HTTP协议，分成HTTP 客户端(本文)，HTTP 服务端。

## 1 Python Client Libraries ##

本文介绍两个库，一个是[urllib](https://docs.python.org/3/library/urllib.html)(Python 原始的标准库用来处理http客户端的请求)，另一个是[requests](http://docs.python-requests.org/en/master/)(由于urllib不易使用，Kenneth Reitz写了requests来提供好用的HTTP client API)。

我们下面的代码都会使用[http://httpbin.org](http://httpbin.org)(Kenneth Reitz写的http server来调试我们的client)。安装和使用非常简单：

{% highlight python linenos %}
$ pip3 install gunicorn httpbin requests
$ gunicorn httpbin:app
{% endhighlight %}

我们开启服务器后，用requests和urllib来get一个url。
{% highlight python linenos %}
>>> import requests
>>> r = requests.get('http://localhost:8000/headers')
>>> print(r.text)
{
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Host": "localhost:8000",
    "User-Agent": "python-requests/2.3.0 CPython/3.4.1 Linux/3.13.0-34-generic"
} }
>>> from urllib.request import urlopen
>>> import urllib.error
>>> r = urlopen('http://localhost:8000/headers')
>>> print(r.read().decode('ascii'))
{
  "headers": {
    "Accept-Encoding": "identity",
    "Connection": "close",
    "Host": "localhost:8000",
    "User-Agent": "Python-urllib/3.4"
} }

{% endhighlight %}

### 1.1 Ports Encryption and Framing ###

1. Ports: HTTP是80, HTTPS是443；

2. Encryption: HTTP无加密，HTTPS=HTTP+TSL

3. Framing: HTTP有3种方式，dilimiter(CR+LF)，Content-Length和chunked encoding。

4. Message格式：3个部分，request/response line，header line，body。

### 1.2 Methods ###

从对服务器影响可逆性来说，主要是2种：

1. 不可逆-POST(增)。这个类里还包括DELETE(删)，PUT(改)。其中POST是非幂等操作，即多次PUT对服务器影响是不一样的；而DELETE和PUT是幂等操作，即多次对服务器和一次是一样的。

2. 可逆-GET(查)。这个类里还包括OPTION和HEAD。其中HEAD用来检查server对client的request是否工作，但返回的response只包含header而没有entity body。

### 1.3 Status Codes ###

状态码常用的有以下几个：

1. 200，表示OK；

2. 404，表示Not Found；

### 1.4 Caching and Validaton ###

服务端该如何利用客户端的缓存来缓解服务端的发送entity body的压力呢？

有以下几种方法，通过服务端第一次发送response给客户端，在response里增加下面几项headers：

1. Cache-Control: no-store，客户端不会默认缓存数据。

2. Cache-Control: max-age=3600，客户端缓存数据，但是只存在3600s；Expires: Thus, 01 Dec 1994 16:00:00 GMT，客户端缓存数据，但是有有效期。后者可以通过客户端改本地时间来越过有效期的设置，但是前者不可以。

3. Last-modified:Tue, 15 Nov 1994 12:45:26 GMT，客户端记录数据最后在服务端修改的时间。

4. E-tag:ETag: "d41d8cd98f00b204e9800998ecf8427e"，客户端记录服务端发来某个数据的版本。

当客户端第二次重新request同样的Path时，服务端会查看客户端发来的request里的：Cache-Control，Last-modified或E-tag来决定是否和当前服务端数据一样，如果一样，就返回status code 304 not modified。这个时候，客户端就可以根据304来直接从缓存中获取数据。在这过程中需要注意两点：

1. requets或urllib不提供缓存服务，需要开发者自己实现。

2. 缓存开启还是需要客户端发送接收服务端**1个来回**的消息，只是如果数据没变，服务端回来的entity body是空的而已，从而减轻服务端压力。


### 1.5 Content ###

#### 1.5.1 Transfer Encoding ####

传输编码，常见有gzip。
传输编码(gzip)和内容编码('utf-8')不一样，前者对client不可见，后者可见。

#### 1.5.2 Negotiation ####

内容协商通常用Accept字段设置，包括语言，内容编码等

#### 1.5.3 Type ####

内容类型由server返回的response的header里的Content-Type字段设置。

### 1.6 HTTP Authentication ###

HTTP认证用来确定客户身份。基本认证是一种方法。

### 1.7 Paths and Hosts ###

由于一台服务器主机可以有很多host，因此需要client在request里指定path字段。

### 1.8 Cookies ###

http没有状态，Cookies用来设置http状态来确定用户身份，同时弥补了http基础认证的UI不美观，用户不友好等问题。

### 1.9 Keep-Alive###

由于TCP需要3次握手，因此建立TCP连接比较昂贵；再加上现在的TLS的握手，就更昂贵。因此最好保持socket alive，而不是request一次建立一次，得到response就关闭。

## 2 总结 ##


{: .img_middle_hg}
![Network Data & Error Summary](/assets/images/posts/2015-04-10-网络实战(九)：HTTP Client/HTTP Client Summary.png)

## 4 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





