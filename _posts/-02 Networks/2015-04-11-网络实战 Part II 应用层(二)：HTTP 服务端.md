---
layout: post
title: 网络实战(二)：应用层 Part II：HTTP 服务端
categories: [-02 Networks]
tags: [Networks, HTTP]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第8篇笔记《缓存和消息队列》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}


前4篇笔记我们介绍了如何设置和关闭TCP，UDP连接，如何通过DNS将hostname换成IP等。但是我们退后一步，思考一下：在网络传输数据之前，我们该如何准备，编码和格式化数据呢；并且对于网络错误我们程序要如何提前准备处理呢。

## 1 HTTP Server ##

### 1.1 WSGI ###

> **CGI(Common Gateway Interface，公共网关接口)**：外部应用程序（CGI程序）与Web服务器之间的接口标准。是WWW技术中最重要的技术之一，可以让一个客户端，从网页浏览器向执行在网络服务器上的程序请求数据。CGI描述了服务器和请求处理程序之间传输数据的一种标准。我们看一个实际例子：现在的个人主页上大部分都有一个留言本。留言本的工作是这样的：先由用户在客户端输入一些信息，如评论之类的东西。接着用户按一下“发布或提交”（到目前为止工作都在客户端），浏览器把这些信息传送到服务器的CGI目录下特定的CGI程序中，于是CGI程序在服务器上按照预定的方法进行处理。在本例中就是把用户提交的信息存入指定的文件中。然后CGI程序给客户端发送一个信息，表示请求的任务已经结束。此时用户在浏览器里将看到“留言结束”的字样。整个过程结束。

但是CGI技术要求每一个HTTP请求都需要运行1个新的进程来执行CGI程序，因此CGI支持的每秒点击率并不高，所以这个时候很多语言开始实现自己的HTTP server。Python实现了http.server内建库。

HTTP Server的技术最早有以下三种：

1. **CGI**：外部应用程序（CGI程序）与Web服务器之间的接口标准。要求每一个HTTP请求都需要运行1个新的进程来执行CGI程序，因此CGI支持的每秒点击率并不高，严重限制了它的性能。

2. **Python的http.server**：子类化``BASEHTTPRequestHandler``并实现``do_GET()``和``do_POST()``。

3. **mod_python**：Apache的python HTTP Server库，提供了认证，登录，文档等功能。它的API为Apache定制。

以上三种方法意味着CGI编写的程序需要重写来运行于http.server，而这两者都需要重写来运行在Apache上。这使得Python编写的HTTP server在不同platform上的移植性非常差。

因此Python社区实现了**WSGI**。

> **WSGI(Python Web Server Gateway Interface，Web服务器网关接口)**：是为Python语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口。WSGI 没有官方的实现, 因为WSGI更像一个协议. 只要遵照这些协议,WSGI应用(Application)都可以在任何服务器(Server)上运行, 反之亦然。WSGI在解耦了Web Server和Web App，称为它们中间的interface。因此任何Web App，从CGI到http.server到mod_python，只要实现了WSGI接口，就可以运行在任何Web Server上。

{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-11-网络实战(十)：HTTP Server/WSGI.png)

下面代码创建了1个``WSGIServer``。

{% highlight python linenos %}
from pprint import pformat
from wsgiref.simple_server import make_server

def app(environ, start_response):
    headers = {'Content-type': 'text/plain; charset=utf-8'}
    start_response('200 OK', list(headers.items()))
    yield 'Here is the WSGI environment:\r\n\r\n'.encode('utf-8')
    yield pformat(environ).encode('utf-8')

if __name__=='__main__':
    httpd = make_server('',8000,app)
    host, port = httpd.socket.getsockname()
    print('Serving on', host, 'port', port)
    httpd.serve_forever()
{% endhighlight %}

> **middleware(中间件)**：是提供操作系统软件和应用软件之间连接的软件，以便于软件各部件之间的沟通，特别是应用软件对于系统软件的集中的逻辑，在现代信息技术应用框架如Web服务、面向服务的体系结构等中应用比较广泛。

WSGI作为middleware的一类，迅速流行起来。

### 1.2 Asynchronous Web (Server) Frameworks ###

WSGI协议在Web Server和Web App间提供了通用接口，但是它没有实现asynchronous server(支持corountines和green thread)。

常用的asynchronous server有下面4个：

1. [Twisted](https://twistedmatrix.com/trac/)， event-driven networking engine，提供了多种intenet协议的handler。

2. Facebook开源的[Tornado](http://www.tornadoweb.org/en/stable/)，a Python web framework and asynchronous networking library，不同于Twisted，Tornado专注于HTTP协议。

3. [Eventlet](http://eventlet.net/)，concurrent networking library,提供了green threads。

4. [asyncio](https://docs.python.org/3/library/asyncio.html?highlight=asyncio#module-asyncio)，Python的作者Guido van Rossum开发的Python3里的统一异步接口模块，需要时间让其流行起来。

> **Web (Server) Framework**：to hide the boilerplate and infrastructural code related to handling HTTP requests and responses. 解决两个主要问题：<br/>
1. **Routing(路由)**，How do we map a requested URL to the code that is meant to handle it?<br/>
2. **Templates(模板)**，How do we create the requested HTML dynamically, injecting calculated values or information retrieved from a database?

> **Green Thread**：运行在VM上的多线程，在user space而不是OS的Kernel Space。

### 1.3 Forward and Reverse Proxies ###

> **HTTP proxy**：1类HTTP server，将从HTTP client进来的请求，让自己成为client，转发给另一个server，然后将返回的response返回给初始的client。

### 1.4 Four Architectures ###

{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-11-网络实战(十)：HTTP Server/4 Python Web Server Architectures.png)

### 1.5 Platforms as a Service ###

> PaaS(Platform as a service，平台即服务)： is a category of cloud computing services that provides a platform allowing customers to develop, run, and manage applications without the complexity of building and maintaining the infrastructure typically associated with developing and launching an app。

然而前面的artechitecture的问题并不是Python语言所特有的，这是1个普遍的Web Server的问题，例如load balancing， multiple tiers of proxy server and application deployment。PaaS的出现提供了1个更高级的抽象，将这一切交给Paas处理，你主要处理Web Server的主要业务即可，流行的支持Python的PaaS有：

1. [Heroku](https://www.heroku.com/)：a cloud Platform-as-a-Service (PaaS) supporting several programming languages and being used as a Web Application Deployment model. 

2. [Docker](https://www.docker.com/)：一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。



### 1.6 REST ###

> [REST((Resource) Representational State Transfer)](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)：给基于网络的客户端/服务器交互的架构制定了**4点相互补充又互相正交的基本约束条件**，使得基于这个风格设计的软件可以通过满足最少约束条件(4个)，达到可见(request包含了其request自身所有信息)，可靠(部分fail不会导致整体fail)，可规模化(多个client)等特点。REST=Uniform(4个以下原则)-Layered(Hierachy)-Client-Cache-Stateless-Server：<br/>
1. **每个资源有唯一ID**：例如URI；<br/>
2. **每个资源有特定表现层**：bytes+encoding (representation metadata to describe those bytes)；<br/>
3. **每个message是自文档(self-documenting)**：比如http response header里包括了编码是JSON。不需要额外的request去获取其编码；<br/>
4. **Hypermedia as the engine of application state(HATEOS)**：任何可能的情况下，使用链接指引可以被标识的事物（资源）。也正是超链接造就了现在的Web。客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。


### 1.7 WSGI Without a Framework ###

略。


## 2 总结 ##

{: .img_middle_mid}
![Network Data & Error Summary](/assets/images/posts/2015-04-11-网络实战(十)：HTTP Server/HTTP Server Summary.png)


## 4 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





