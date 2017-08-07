---
layout: post
title: 网络实战(二)：应用层 Part III：The World Wide Web
categories: [-02 Networks]
tags: [Networks, HTTP, Flash, Django]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第11篇笔记《The WOrld Wide Web》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

前面[《HTTP Client》]({{site.baseurl}}/networks/2015/04/11/网络实战-Part-II-应用层(二)-HTTP-服务端.html)和[《HTTP Server》]({{site.baseurl}}/networks/2015/04/11/网络实战-Part-II-应用层(二)-HTTP-服务端.html)两篇文章介绍了**Hypertext Transfer Protocol(HTTP)**用来让client request docuemnts以及让server response来提供documents。document包括：book，image，video，audio。更进一步说，HTTP的Hypertext指的是通过相互交叉引用来形成一个单个的信息源，即the World Wide Web。

> HTTP: built to deliver the World Wide Web.

## 1 WWW: Baiscs ##

### 1.1 Hypermedia and URLs ###

书本中引用其他书本这种形式已经存在上千年。对于这种传统的引用，读者需要自己找到某本书的某一页来查看引文。但是WWW完全改变了这种情况，它将寻找原文的任务交给了计算机，如果你对引文感兴趣，你只需要鼠标一点就会转到引用的原文。

> Hypermedia：超媒体，包括了文字，图像，视频，音频等超链接。

> URL：用来描述超媒体的引用地址，通常包括hostname，path，query，例如：http://www.google.com/search?q=ipod&bTNI=yes。

#### 1.1.1 Parsing and Building URLs ####

**Parsing URLs Step1:**分成scheme，hostname，path，query。

{% highlight python linenos %}
>>> from urllib.parse import urlsplit
>>> u = urlsplit('https://www.google.com/search?q=apod&btnI=yes')
>>> tuple(u)
('https', 'www.google.com', '/search', 'q=apod&btnI=yes', '')
{% endhighlight %}

**Parsing URLs Step2:**path和query可能包含界定符，比如特殊符号``&(%26)``，``/(2F%)``，``+(space)``需要进一步转换。

{% highlight python linenos %}
>>> from urllib.parse import parse_qs, parse_qsl, unquote
>>> u = urlsplit('http://example.com/Q%26A/TCP%2FIP?q=packet+loss')
>>> path = [unquote(s) for s in u.path.split('/')]
>>> query = parse_qsl(u.query)
>>> path
['', 'Q&A', 'TCP/IP']
>>> query
[('q', 'packet loss')]
{% endhighlight %}

**Building URLs:**将shceme，hostname，path，query合成一个URLs。

{% highlight python linenos %}
>>> from urllib.parse import quote, urlencode, urlunsplit
>>> urlunsplit(('http', 'example.com',
...           '/'.join(quote(p, safe='') for p in path),
...           urlencode(query), ''))
'http://example.com/Q%26A/TCP%2FIP?q=packet+loss'

{% endhighlight %}


#### 1.1.2 Relative URLs ####


terminal中的文件系统和我们这里说的URL是一样的，如果/开头，则是绝对路径；否则是相对路径。

{% highlight python linenos %}
$ wc -l /var/log/dmesg
977 dmesg
$ wc -l dmesg
wc: dmesg: No such file or directory
$ cd /var/log
$ wc -l dmesg
977 dmesg
{% endhighlight %}

Python中URL对于相对路径转换成绝对路径的函数是``urllib.parse.urljoin()``，这和``os.path.join()``是一样的。
{% highlight python linenos %}
>>> from urllib.parse import urljoin
>>> base = 'http://tools.ietf.org/html/rfc3986'
>>> urljoin(base, 'rfc7320')
'http://tools.ietf.org/html/rfc7320'
>>> urljoin(base, '.')
'http://tools.ietf.org/html/'
>>> urljoin(base, '..')
'http://tools.ietf.org/'
>>> urljoin(base, '/dailydose/')
'http://tools.ietf.org/dailydose/'
>>> urljoin(base, '?version=1.0')
'http://tools.ietf.org/html/rfc3986?version=1.0'
>>> urljoin(base, '#section-5.4')
'http://tools.ietf.org/html/rfc3986#section-5.4'
{% endhighlight %}


### 1.2 Hypertext Markup Language ###

[http://www.w3.org/TR/html5/](http://www.w3.org/TR/html5/)

[http://www.w3.org/TR/CSS/](http://www.w3.org/TR/CSS/)

[https://developer.mozilla.org/en-US/docs/Web/JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

[https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)

[Chrome Developer Tool](https://developer.chrome.com/devtools)

### 1.3 Reading and Writing to a Database ###

python自带的SQLite库是[sqlite3](https://docs.python.org/3/library/sqlite3.html?highlight=sqlite3)；对于第三方库MySQL的使用，见[这里]({{site.baseurl}}/database/2015/06/01/MySQL入门.html)。


## 2 WWW: Python Web Framework ##

### 2.1 Flask ###

> [Flask](http://flask.pocoo.org/): The most popular of the microframeworks, built atop solid tools and supporting many modern features (if the programmer knows to look for and take advantage of them). Often combined with SQLAlchemy or a nonrelational database back end.

### 2.2 Django ###

> [Django](https://www.djangoproject.com/): a high-level Python Web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of Web development, so you can focus on writing your app without needing to reinvent the wheel. It’s free and open source.

### 2.3 Tornado ###

> [Tornado](http://www.tornadoweb.org/en/stable/): a web framework like none of the others listed here because it uses the asynchronous callback pattern from Chapter 9 to allow many dozens or hundreds of client connections to be supported per operating system thread, instead of just one client per thread. It also stands out because it is not tied to supporting WSGI—it has direct support for WebSockets (described in the next section). The cost is that many libraries have difficulty working with its callback pattern, so the programmer has to find async alternatives to the usual ORM or database connector that they would choose.


## 3 WWW: Other Topics ##

### 3.1 WebSockets ###

> WebSocket 规范定义了一种 API，可在网络浏览器和服务器之间建立“套接字”连接。简单地说：客户端和服务器之间存在持久的连接，而且双方都可以随时开始发送数据。

### 3.2 Web Scraping ###

见[《Web Scraping with Python》]({{site.basrurl}}/web%20scraping/2015/12/01/Web-Scraping-Part-I-Basic-Scrapers-(一)-BeautifulSoup入门.html)

## 4 总结 ##


{: .img_middle_hg}
![Network Data & Error Summary](/assets/images/posts/2015-04-12-网络实战(十一)：The World Wide Web/HTTP&WWW Summary.png)


## 5 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





