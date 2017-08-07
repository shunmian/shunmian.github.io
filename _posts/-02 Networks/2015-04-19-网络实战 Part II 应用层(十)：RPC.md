---
layout: post
title: 网络实战(二)：应用层 Part X：RPC
categories: [-02 Networks]
tags: [Networks, HTTP]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第18篇笔记《RPC》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 RPC Basics ##

> **PRC**(Remote Procedure Call，远程过程调用)：是一个计算机通信协议。该协议允许client的程序通过internet调用server，就像在本地调用server一样。



RPC区别与HTTP或者SMTP有以下3个方面：

1. **RPC的调用目的是宽泛的，由用户定义**。而HTTP调用目的是获取documents，SMTP是发送messgage。

2. **RPC的函数是宽泛的，由用户定义**。而HTTP预定义了一些基本函数，如GET，PUT等；SMTP如EHLO，MAIL等。

3. **RPC的客户端和服务端的代码和本地函数调用无异**。

RPC有以下5个特点：

1. **RPC调用目的的宽泛性使得它能支持的数据格式是严格的**。对于OOP语言，object有时候也可以在RPC上先序列化后反序列化来传输。但是实时的client的object，像file，socket，就不能传输。

2. **RPC的server可以将异常发回给client**。

3. **RPC提供introspection机制**。client和server在连接初始会交换双方共同支持的函数即其参数。

4. **RPC提供寻址机制**。有些client的远程调用会自动寻找server。

5. **RPC提供身份认证，权限控制**。不同用户可以在server上有不同的函数和data access。

 
### 1.1 XML-RPC ###

#### 1.1.1 Server ####

{% highlight python linenos %}
import operator, math
from xmlrpc.server import SimpleXMLRPCServer
from functools import reduce

def main():
    server = SimpleXMLRPCServer(('127.0.0.1',7001))  #create ROC Server
    server.register_introspection_functions()        #enable introspection
    server.register_multicall_functions()            #enable mutlcall
    server.register_function(addtogether)            #register 3 functions
    server.register_function(quadratic)
    server.register_function(remote_repr)
    print("Server ready")
    server.serve_forever()                           #server run forever


def addtogether(*things):
    '''Add together everying in the list `things`.'''
    return reduce(operator.add,things)

def quadratic(a,b,c):
    '''Determing `x` values satisfying `a` *x*x + `b`*x + c == 0 '''
    b24ac = math.sqrt(b*b-4.0*a*c)
    return list(set([(-b-b24ac)/2.0*a, (-b+b24ac)/2.0*a]))  #RPC function can only return 1 argument or list

def remote_repr(arg):
    '''Return the `repr()` rendering of the supplied `arg`.'''
    return  arg

if __name__ == '__main__':
    main()

{% endhighlight %}

#### 1.1.2 Client ####

##### 1.1.2.1 列举Server函数 #####

{% highlight python linenos %}
import  xmlrpc.client

def main():
    proxy = xmlrpc.client.ServerProxy('http://127.0.0.1:7001')

    print('Here are the functions supported by this server:')
    for method_name in proxy.system.listMethods():

        if method_name.startswith('system.'):
            continue

        signatures = proxy.system.methodSignature(method_name)
        if isinstance(signatures,list) and signatures:
            for signature in signatures:
                print('%s(%s)' % (method_name,signature))

        else:
            print('%s(...)' % (method_name,))

        method_help = proxy.system.methodHelp(method_name)
        if method_help:
            print(' ',method_help)

if __name__ == '__main__':
    main()
{% endhighlight %}

##### 1.1.2.2 调用server函数 #####

{% highlight python linenos %}
import  xmlrpc.client

def main():
    proxy = xmlrpc.client.ServerProxy('http://127.0.0.1:7001')
    print(proxy.addtogether('x','y','z'))
    print(proxy.addtogether(20,30,4,1))
    print(proxy.quadratic(2,-4,0))
    print(proxy.quadratic(1,2,1))
    print(proxy.remote_repr((1,2.0,'three')))
    print(proxy.remote_repr([1,2.0,'three']))
    print(proxy.remote_repr({'name': 'Arthur',
                             'data':{'age':42,'sex':'M'}}))
    print(proxy.quadratic(1,0,1))

if __name__ == '__main__':
    main()
{% endhighlight %}

通过上面代码，可见RPC函数有以下几个特点：

1. 函数参数类型没有限制。可以是number或者string，个数可以变化。

2. 函数返回参数个数只能是1。

3. 输入和返回参数支持的sequence类型只有1种，即list。

4. 复杂数据结构可以是递归的，例如上面的dict。

5. server的error可以传递回client。

##### 1.1.2.3 调用server函数，一次性打包 #####

{% highlight python linenos %}
import  xmlrpc.client

def main():
    proxy = xmlrpc.client.ServerProxy('http://127.0.0.1:7001')
    multicall = xmlrpc.client.MultiCall(proxy)
    multicall.addtogether('x','y','z')
    multicall.addtogether(20,30,4,1)
    multicall.quadratic(2,-4,0)
    multicall.remote_repr({'name': 'Arthur',
                             'data':{'age':42,'sex':'M'}})

    for answer in multicall():
        print(answer)

if __name__ == '__main__':
    main()

{% endhighlight %}

当将多个client的function call打包成1个时，我们可以看到server端只收到1此http连接。

{% highlight python linenos %}
localhost - - [04/Oct/2010 00:16:19] "POST /RPC2 HTTP/1.0" 200 -
{% endhighlight %}


#### 1.1.3 XML Based Request & Response ####

对于xml-RPC的Client的请求和Server的reponse到底是什么样子，我们可以通过下面的例子一窥究竟。

{% highlight python linenos %}
# the first call to quadratic() that the sample client program makes:
<?xml version='1.0'?>
<methodCall>
<methodName>quadratic</methodName>
  338
<params>
<param>
<value><int>2</int></value>
</param>
<param>
<value><int>-4</int></value>
</param>
<param>
<value><int>0</int></value>
</param>
</params>
</methodCall>


#The response to the preceding call looks like this:
<?xml version='1.0'?>
<methodResponse>
<params>
<param>
<value><array><data>
<value><double>0.0</double></value>
<value><double>8.0</double></value>
</data></array></value>
</param>
</params>
</methodResponse>
{% endhighlight %}

### 1.2 JSON-RPC ###

上面xML-RPC传输的数据看起来太过臃肿，而JSON-RPC就显得简洁多了。

#### 1.2.1 Server ####

{% highlight python linenos %}
from jsonrpclib.SimpleJSONRPCServer import SimpleJSONRPCServer

def main():
    server = SimpleJSONRPCServer(('127.0.0.1',7001))    #create JSON Server
    server.register_introspection_functions()           #enable introspection
    server.register_function(lengths)
    print('server in running......')
    server.serve_forever()                              #run forever


def lengths(*args):
    '''Measure the length of each argument'''
    results = []

    for arg in args:
        try:
            arglen = len(arg)
        except TypeError:
            arglen = None
        results.append((arg,arglen))
    return results


if __name__ == '__main__':
    main()

{% endhighlight %}

#### 1.2.2 Client ####

{% highlight python linenos %}
from jsonrpclib import Server

def main():
    proxy = Server('http://127.0.0.1:7001')
    print(proxy.lengths(*['hi',27,'itsgoodday']))

if __name__ == '__main__':
    main()
{% endhighlight %}


#### 1.2.3 JSON Based Request & Response ####

用wireshark可以嗅探到Request和Response如下，比XML简洁多了。
{% highlight python linenos %}
{"version": "1.1",
 "params": [[1, 2, 3], 27, {"Rigel": 0.12, "Sirius": -1.46}],
 "method": "lengths"}
{"result": [[3, [1, 2, 3]], [null, 27],
            [2, {"Rigel": 0.12, "Sirius": -1.46}]]}
{% endhighlight %}

### 1.3 Self-Documenting Data ###

略

## 2 RPC Advanced ##

在RPC上传递Object比较复杂和tricky：

1. Object在不同语言里(Client和Server在不同语言下运行的情况)的语义不一样(方法，属性)，难以统一。

2. Object的数据传递多少是个问题。全部传递开销太大；部分传递，如何选择？

关于在RPC上传递Object，业内有两个主导方法**SOAP**和**CORBA**。两者都比较复杂，需要一整本书来讨论。

但是对于Client和Server都是Python语言来说，却有两个简单的方法。

1. [Pyro](https://pythonhosted.org/Pyro/1-intro.html)，建立在pickle模块上，任何pickle-able的object都可以在client和server里传(unpickle-able的object也可以通过实现具体方法变成pickle-able)；

2. [RPyc](https://rpyc.readthedocs.io/en/latest/)，这个更像CORBA，比Pyro复杂。在client和server间传递的object不是object本身，而是其ID。当然这种情况下网络traffic和security的要求就更高。

### 2.1 Talking About Objects: Pyro and RPyC ###

#### 2.1.1 RPyC Example ###


{% highlight python linenos %}
import rpyc

def main():
    from rpyc.utils.server import ThreadedServer
    t = ThreadedServer(MyService, port = 18861)
    t.start()

class MyService(rpyc.Service):
    def exposed_line_counter(self, fileobj, function):      # let client call the exposed_method
        print('Client has invoked exposed_line_counter()')  # pass the filobj ID back to client to call the function.
        for linenum, line in enumerate(fileobj.readlines()):
            function(line)
        return linenum+1

if __name__ == '__main__':
    main()
{% endhighlight %}


{% highlight python linenos %}
import rpyc

def main():

    config = {'allow_public_attrs': True}
    proxy = rpyc.connect('localhost',18861,config=config)
    fileobj = open('testfile.txt')
    linecount = proxy.root.line_counter(fileobj,noisy)
    print('The number of lines in the file was', linecount)

def noisy(string):
    print('Noisy:',repr(string))

if __name__ == '__main__':
    main()

{% endhighlight %}

### 2.2 RPC, Web Frameworks, and Message Queues ###

Python中的xmlrpc模块实际上对于想要使用RPC的开发者来说很少使用到。通常他们将RPC服务集成到大型网站而不是运行1个单独的port。下面介绍3种方法集成到网站的方法：

1. 在WSGI安装RPC插件。

2. 某些web framework已经实现了RPC库。

3. Message Queuqe。

### 2.3 Recovering From Netowrk Errors ###

网络问题是RPC比较复杂的1个问题，有可能函数已完成调用，server数据库已更改，但是网络问题导致client以为request fail，重复1遍request。因此我们通常将request设成幂等的，也就是说多次操作对server影响一样，比如“perform transaction 312312 to remove ￥10”就比“remove ￥10”安全(前者幂等，后者会多次扣除￥10)。同时将根据子任务来使用``try:except``比根据函数来使用更有指导性。

## 3 总结 ##

{: .img_middle_hg}
![Network Data & Error Summary](/assets/images/posts/2015-04-19-网络实战(十八)：RPC/Telnet_SSH_FTP_RPC Summary.png)

## 4 全书总结 ##

{: .img_middle_hg}
![fopnp](/assets/images/posts/2015-04-19-网络实战(十八)：RPC/fopnp.png)

## 5 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





