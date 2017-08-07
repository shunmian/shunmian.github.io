---
layout: post
title: 网络实战(一)：传输层 Part IV：Hostname和DNS
categories: [-02 Networks]
tags: [Networks, DNS]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第4篇笔记《Hostname和DNS》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Hostname ##

### 1.1 主机名称###


{: .img_middle_hg}
![Hostname](/assets/images/posts/2015-04-05-网络实战(四)：Hostname & DNS/Hostname.png)


#### 1.1.1 Hostname, IP, Domain Name, DNS ####

这四个概念通常比较容易混淆，我们将他们之间的关系用一张图表示：

1. DNS表示将Hostname转换成IP。

2. IP有IPv4和IPv6，IPv6不仅仅解决IPv4数量不够的问题，还有传输上的提升。

3. Domain Name(域名)是网络的域的名字，域表示不同的区域。“root-level” name servers知道所有“top-level domains(TLDs,顶级域名，包括.com,.org,.edu等)”。域名分等级，从右到左，等级细分，由“.”分开。

4. machine name是机器的名字，加上其所在的域名就是hostname。


#### 1.1.2 5个套接口坐标 ####

一个socket由5个坐标完全确定：

1. address family(地址簇)：规定了地址的格式。通常用AF_INET，对应的地址格式是(IP,port)

2. type(类型)：常用的有SOCK_DGRAM(UDP)和SOCK_STREAM(TCP)。

3. protocol：常用的有IPROTO_UDP和TPPROTO_TCP。

4. IP。

5. port。

#### 1.1.3 IPv4 vs. IPv6 ####

IPv6不仅仅解决IPv4数量不够的问题，还有传输上的提升。IPv4到IPv6的过渡需要开发者写更General的代码来兼容。
### 1.2 进阶IP解析: getaddrinfo() ###

为了平滑的完成IPv4到IPv6的转变带来的代码上的不一致，socket模块提供了一个统一的接口方法``getaddrinfo()``，它是该模块的顶层方法。

``getaddrinfo(host, port, family=0, type=0, proto=0, flags=0)``，返回5个套接口坐标。

### 1.2 getaddrinfo() ###

#### 1.2.1 基本用法 ####

{% highlight python linenos %}
#1.基本用法
infolist = socket.getaddrinfo('gatech.edu','www')
pprint(infolist)

info = infolist[0]
sock = socket.socket(*info[0:3])
pprint(info[4])
sock.connect(info[4])
#output:
[(<AddressFamily.AF_INET: 2>,
  <SocketKind.SOCK_DGRAM: 2>,
  17,
  '',
  ('130.207.160.173', 80)),
 (<AddressFamily.AF_INET: 2>,
  <SocketKind.SOCK_STREAM: 1>,
  6,
  '',
  ('130.207.160.173', 80))]
{% endhighlight %}

#### 1.2.2 不同Flag ####

##### 1.2.2.1 AI_ADDRCONFIG, AI_V4MAPPED #####

{% highlight python linenos %}
#2_Connect to a Service
info = socket.getaddrinfo('iana.org','www',0,socket.SOCK_STREAM,0,socket.AI_ADDRCONFIG|socket.AI_V4MAPPED)
print(info)
#output: [(<AddressFamily.AF_INET: 2>, <SocketKind.SOCK_STREAM: 1>, 6, '', ('192.0.43.8', 80))]
{% endhighlight %}

##### 1.2.2.2 AI_CANONNAME #####

{% highlight python linenos %}
#3_Canonical Hostname
info = socket.getaddrinfo('iana.org','www',0,socket.SOCK_STREAM,0,socket.AI_CANONNAME)
print(info)

#output:[(<AddressFamily.AF_INET: 2>, <SocketKind.SOCK_STREAM: 1>, 6, 'iana.org', ('192.0.43.8', 80)), (<AddressFamily.AF_INET6: 30>, <SocketKind.SOCK_STREAM: 1>, 6, 'iana.org', ('2001:500:88:200::8', 80, 0, 0))]
{% endhighlight %}

##### 1.2.2.3 Other Flags #####

略。

#### 1.2.3 Primitive Name Service Routines ####

{% highlight python linenos %}
import socket

print(socket.gethostname())                     #LALdeMacBook-Pro.local
print(socket.getfqdn())                         #laldemacbook-pro.local

print(socket.gethostbyname('cern.ch'))          #188.184.9.234
print(socket.gethostbyaddr('188.184.9.234'))    #('webrlb01.cern.ch', ['234.9.184.188.in-addr.arpa'], ['188.184.9.234'])

print(socket.getprotobyname('UDP'))             #17
print(socket.getservbyname('www'))              #80
print(socket.getservbyport(80))                 #http

print(socket.gethostbyname(socket.getfqdn()))   #192.168.0.101
{% endhighlight %}

### 1.3 应用到自己代码 ###

将上面所有put together，可以写一个更general的client方法。 

{% highlight python linenos %}
import argparse,socket,sys

def connect_to(hostname_or_IP):

    try:
        infolist=socket.getaddrinfo(hostname_or_IP,'www',0,socket.SOCK_STREAM,0,socket.AI_ADDRCONFIG|socket.AI_V4MAPPED|socket.AI_CANONNAME)
    except socket.gaierror as e:
        print("Name service failure:",e.args[1])
        sys.exit(1)
    else:
        print("infolist get successful: {}".format(infolist))


    info = infolist[0]
    sock = socket.socket(*info[0:3])
    try:
        sock.connect(*info[4:])
    except socket.error as e:
        print('client connectio error: {}'.format(e))
    else:
        print('{} is listening on port 80'.format(info[4:]))

if __name__=='__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('hostname',help='the hostname or IP for connection')
    arg = parser.parse_args()
    connect_to(arg.hostname)
{% endhighlight %}



## 2 DNS ##

> DNS(Domain Name Service): The Domain Name System (DNS) is a hierarchical decentralized naming system for computers, services, or any resource connected to the Internet or a private network. DNS is in Application Layer of Network Stack, using UDP.


### 2.1 为什么不用原始DNS###

{: .img_middle_mid}
![DNS request](/assets/images/posts/2015-04-05-网络实战(四)：Hostname & DNS/DNS request.png)

DNS一次请求的路径：本地DNS->root DNS->top-level DNS(TLD)->Authoritive DNS。

你可以在terminal中输入``whois``命令，来直接向TLD请求DNS。
{% highlight python linenos %}

$ whois python.org
Domain Name:PYTHON.ORG
Created On:27-Mar-1995 05:00:00 UTC
Last Updated On:07-Sep-2006 20:50:54 UTC
Expiration Date:28-Mar-2016 05:00:00 UTC
...
Registrant Name:Python Software Foundation
...
Name Server:NS2.XS4ALL.NL
Name Server:NS.XS4ALL.NL
{% endhighlight %}


由于完整的DNS请求路径要经过4词请求，因此DNS比较费时间。通常依赖OS的DNS服务要好于自己使用原始DNS，因为OS有cache，可以提供更好的服务。

### 2.2 用python来请求DNS ###

唯一一种情况开发者需要自己请求DNS的情况：邮件服务器或者邮件客服想直接发送邮件而不通过本地邮件中转。

#### 2.2.1 dnspython3 module ####

dnspython3是建立在os上的DNS module。可用于上述情况。

{% highlight python linenos %}
import argparse, dns.resolver

def lookup(name):
    for qtype in 'A','AAAA','CNAME','MX','NS':
        answer = dns.resolver.query(name,qtype,raise_on_no_answer=False)
        if answer.rrset is not None:
            print(answer.rrset)

if __name__=='__main__':
    parser = argparse.ArgumentParser(description='Resolve a name using DNS')
    parser.add_argument('name',help='name that you want to lookup in DNS',nargs='?',default='python.org')
    arg = parser.parse_args()
    lookup(arg.name)

#output:
#python.org. 34208 IN A 23.253.135.79
#python.org. 46294 IN AAAA 2001:4802:7901:0:e60a:1375:0:6
#python.org. 600 IN MX 50 mail.python.org.
#python.org. 82077 IN NS ns1.p11.dynect.net.
#python.org. 82077 IN NS ns2.p11.dynect.net.
#python.org. 82077 IN NS ns3.p11.dynect.net.
#python.org. 82077 IN NS ns4.p11.dynect.net.

{% endhighlight %}

5种DNS query：

1. 'A'表示IPv4，如果你要通过IPv4，HTT访问python.org，需要访问23.253.135.79；

2. ‘AAAA’表示IPv6，如果你要通过IPv6，HTT访问python.org，需要访问2001:4802:7901:0:e60a:1375:0:6；

3. ‘CNAME’表示Canonical Name，即别名，这里没有；

4. ‘MX’表示邮件地址，如果你要发@python.org邮件，需要发到mail.python.org；

5. ‘NS’表示name server，如果你要请求python.org的子域名，需要发到ns1.p11.dynect.net或者ns1-4:


#### 2.2.2 解析邮件域名 ####

略

## 3 总结 ##

{: .img_middle_lg}
![Hostname & DNS Summary](/assets/images/posts/2015-04-05-网络实战(四)：Hostname & DNS/Hostname & DNS Summary.png)


## 4 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);

- [《Web Scraping Part II：Advanced Scrapers (三)：表单和登录》]({{site.baseurl}}/web%20scraping/2015/12/09/Web-Scraping-Part-II-Advanced-Scrapers-(%E4%B8%89)-%E8%A1%A8%E5%8D%95%E5%92%8C%E7%99%BB%E5%BD%95.html);




