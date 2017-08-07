---
layout: post
title: 网络实战(一)：传输层 Part VI：TLS&SSL
categories: [-02 Networks]
tags: [Networks, TLS/SSL]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第6篇笔记《TLS/SSL》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}


## 1 TLS介绍 ##

> TLS(Transport Layer Security，传输层安全协议):最早被称为SSL(Secure Sockets Layer，安全套接层)，是网络传输加密数据的一种协议，目的是为互联网通信，提供安全及数据完整性保障。

比如你登录gmail时的用户名和密码都是通过HTTPS(实现了TLS的HTTP)加密和客户端通信，而只有你和客户端拥有共同的公钥才能解密，其他中间人看到的都是毫无意义的信息。

### 1.1 TLS原理 ###

{: .img_middle_hg}
![TLS原理](/assets/images/posts/2015-04-07-网络实战(六)：TLS&SSL/TLS原理.png)

### 1.2 TLS职责 ###

TLS可以加密通信数据，但是TLS不能加密下面的信息：

1. client address(IP,port)和 Server address(IP,port)；

2. DNS request to learn servre's address

### 1.3 证书生成 ###

通常申请证书需要2个信息：

1. 描述实体的文本，包括commonName，即域名。

2. 公钥(一列数字)，由openssl生成，比如用RSA算法。

具体见[基于OpenSSL自建CA和颁发SSL证书](http://seanlook.com/2015/01/18/openssl-self-sign-ca/)。

### 1.4 TLS使用方式 ##

通常TLS的使用有两种方式：

1. 直接在Python里使用不加密的代码，让TSL留给更低一层的服务，比如HTTPS(HTTP+TLS)或者Apache。将TLS剥离出来可以让代码更具有高内聚和低耦合的特性。

2. 直接在Python程序里使用ssl module；

本文下面内容将讨论第二种方式。

## 2 Python ssl module ##

> Opinionated API vs. Enterprise API：前者指API作者自己认为最好的设置作为默认推荐给使用者；后者指需要使用者自己设置，有大量选项可用。



### 2.1 基本ssl ###


{% highlight python linenos %}
import socket, ssl,argparse

def client(host,port,cafile=None):
    purpose = ssl.Purpose.SERVER_AUTH
    context = ssl.create_default_context(purpose,cafile=cafile)		#SSLContext object to set the configuration for client(SERVER_AUTH)

    raw_sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    raw_sock.connect((host, port))

    ssl_sock = context.wrap_socket(raw_sock,server_hostname=host)	#SSLSocket object wrap socket object with all its method
    print('SSL client binding at: {}, connecting to: {}'.format(ssl_sock.getsockname(),ssl_sock.getpeername()))

    data = b''
    while True:
        more = ssl_sock.recv(4096)
        if not more:
            break;
        data += more
    ssl_sock.recv()
    print('SSL client receive message: {}'.format(data.decode('utf-8')))
    ssl_sock.close()
    print('SSL client closed')

def server(host,port,certificate,cafile=None):
    purpose = ssl.Purpose.CLIENT_AUTH
    context = ssl.create_default_context(purpose,cafile=cafile)	#SSLContext object to set the configuration for server(CLIENT_AUTH)
    context.load_cert_chain(certificate)						#load server's certificate

    raw_sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    raw_sock.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)
    raw_sock.bind((host,port))
    raw_sock.listen(1)
    print('server listen at: {}'.format(raw_sock.getsockname()))

    raw_conn, client = raw_sock.accept()
    ssl_sock = context.wrap_socket(raw_conn,server_side=True)
    print('SSL server bindigg at: {}'.format(ssl_sock.getsockname()))
    ssl_sock.sendall(('Beatiful is better than ugly'*10000).encode('utf-8'))
    ssl_sock.close()
    print('SSL server closed')

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='SSL demonstration')
    choices = ('server','client')
    parser.add_argument('role',choices=choices,help='the server or client role')
    parser.add_argument('host',help='the server\'s interface or client\'s host)')
    parser.add_argument('port',help='port number',default=1060,type=int)
    parser.add_argument('-ca',help='the Certificate Authorities')
    parser.add_argument('-c',help='server\'s certificate')

    args = parser.parse_args()

    if args.role == 'server':
        server(args.host,args.port,args.c,args.ca)
    else:
        client(args.host,args.port,args.ca)
{% endhighlight %}

可以看到以上代码遵循下面的pattern：

1. 创建SSLContext object，设置purpose，client为``Purpose.SERVER_AUTH``，server为``Purpose.CLIENT_AUTH``。同时server需要设置证书``context.load_cert_chain(certificate)``。

2. 用``wrap_socket``方法封装原始socket成SSLSocket object，该object拥有原始socket几乎所有的方法。需要注意的是wrap server的时候需要设置``server_side=True``。

3. 用SSLSocket发送和接收数据。

如果用tcpdump来抓包，你可以看到输出的都是加密过的乱码。

> tcpdump：Linux中强大的网络数据采集分析首选工具之一，根据使用者的定义对网络上的数据包进行截获的包分析工具。

{% highlight python linenos %}
$ sudo tcpdump -n port 1060 -i lo -X
{% endhighlight %}

除了刚开始的几个packet里有public key等数据是没有加密的，后面的‘Beatiful is better than ugly’数据显示是乱码。需要注意的是client address(IP,port)和server address(IP,port)TLS是不能加密的。

{: .img_middle_mid}
![TLS原理](/assets/images/posts/2015-04-07-网络实战(六)：TLS&SSL/tcpdump response.png)

### 2.2 进阶ssl ###

#### 2.2.1 手选加密和Perfect Foward Security ####

> Perfect Foward Security(前向安全)：保护过去进行的通讯不受密码或密钥在未来暴露的威胁。如果系统具有前向安全性，就可以保证万一密码或密钥在某个时刻不慎泄露，过去已经进行的通讯依然是安全，不会受到任何影响，即使系统遭到主动攻击也是如此。

如果你对数据安全要求非常严格，一个重要的问题是考虑Perfect Foward Security(前向安全)：即Eve截取并存储了加密后的会话信息，并且在未来或得了Bob和Alice的Public Key或者Bod的Private Key，就可能解密之前存储的回话信息，PFS就是防止这种情况发生。

那么你就要定制SSLContext的加密方法而不是使用默认的SSLContext(由``create_default_context``返回)。

{% highlight python linenos %}
context = ssl.SSLContext(ssl.PROTOCOL_TLSv1_2)
context.verify_mode = ssl.CERT_NONE
context.options |= ssl.OP_CIPHER_SERVER_PREFERENCE  # choose *our* favorite cipher
context.options |= ssl.OP_NO_COMPRESSION            # avoid CRIME exploit
context.options |= ssl.OP_SINGLE_DH_USE             # for PFS
context.options |= ssl.OP_SINGLE_ECDH_USE           # for PFS
context.set_ciphers('ECDH+AES128 ')                 # choose over AES256, says Schneier
{% endhighlight %}

#### 2.2.2 TLS支持的协议 ####

Python标准库里支持TLS协议的库包括：

1. http.client。

2. smtplib。

3. poplib。

4. imaplib。

5. ftblib。

6. nntlib。

需要注意的是以上6个库支持TLS的方式有两种。：

1. 在已有协议方法中间插入TLS握手，加密，解密等步骤，来升级已有协议。经TLS升级后的协议的Port Number不变。

2. 分配一个新的Port Number给经TLS升级后的协议，例如HTTP(Port：80)转HTTPS(Port:443)。

#### 2.2.3 TSL参数获取 ####

略。

## 3 总结 ##

{: .img_middle_lg}
![TLS原理](/assets/images/posts/2015-04-07-网络实战(六)：TLS&SSL/
TSLSummary.png)

## 4 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





