---
layout: post
title: 网络实战(二)：应用层 Part VIII：Telnet and SSH
categories: [-02 Networks]
tags: [Networks, Telent, SSH]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第16篇笔记《Telnet and SSH》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Command-Line ##

见[Unix command line]({{site.baseurl}}/unix/2015/02/01/Unix-Command-Line.html)。

### 1.1 在Python中运行命令行：subprocess ###


在Python中，我们通过标准库中的[17.5 subprocess](https://docs.python.org/3/library/subprocess.html#subprocess.run)包来fork一个子进程，并运行一个外部的程序。

{% highlight python linenos %}
import subprocess

def test1():
    args = ['echo', 'Sometimes', '*', 'is just an asterisk']
    subprocess.call(args)

def test2():
    while True:
        args = input('[ ').strip().split() #'['是提示符
        if args is None:
            continue
        elif args == ['exit']:
            break
        elif args[0] == 'show':
            print(args[1:])
        else:
            try:
                subprocess.call(args)
            except Exception as e:
                print(e)

if __name__=='__main__':
    test2()
{% endhighlight %}


## 2 Telnet ##

> **Telnet：** 一种应用层协议，是Internet**远程登录服务的标准协议和主要方式**，使用于互联网及局域网中，为用户提供了在本地计算机上完成远程主机工作的能力。

Telnet有以下几个特点：

1. **原理：**在终端使用者的电脑上使用telnet程序，用它连接到服务器。用户首先在电脑运行Telnet程序，连接至目的地服务器，然后输入账号和密码以验证身份。用户可以在本地主机输入命令，然后让已连接的远程主机运行，就像直接在对方的控制台上输入一样。

2. **安全性：**传统Telnet会话所传输的数据并未加密，账号和密码等敏感数据容易会被窃听，因此很多服务器都会封锁Telnet服务，改用更安全的SSH。

3. **入侵者：**对于入侵者而言，Telnet只是一种远程登录的工具。一旦入侵者与远程主机建立了Telnet连接，入侵者便可以使用目标主机上的软、硬件资源，而入侵者的本地机只相当于一个只有键盘和显示器的终端而已。

为了测试网络客户端和服务端，本书作者建立了1个虚拟的服务端，称为playgraound。运行该playground需要安装[virtualBox](https://www.virtualbox.org/wiki/Downloads)和[vagrant](https://www.vagrantup.com/)。具体步骤见[playgraound](https://github.com/brandon-rhodes/fopnp/tree/m/playground)。下面是整个框架供参考。

{: .img_middle_hg}
![Network Data & Error Summary](/assets/images/posts/2015-04-17-网络实战(十六)：Telnet & SSH/FOPNP playground.png)

我们进入SSH进入到h1，用telnet连接ftp.example.com看看。

{% highlight python linenos %}
vagrant@vagrant-ubuntu-vivid-64:~$ ssh h1 
Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.19.0-25-generic x86_64)
 * Documentation: https://help.ubuntu.com/ Last login: Thu Oct 13 07:57:33 2016 from 192.168.1.12 

root@h1:~# telnet ftp.example.com 
Trying 10.130.1.2... 
Connected to ftp.example.com. 
Escape character is '^]'. 
Ubuntu 14.04.3 LTS 
ftp.example.com login: brandon 
Password: 
Last login: Thu Oct 13 08:06:53 UTC 2016 from modemA on pts/0 Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.19.0-25-generic x86_64) 
 * Documentation: https://help.ubuntu.com/

{% endhighlight %}

看完上面代码我们来理解下面python里[21.19 telnetlib](https://docs.python.org/3/library/telnetlib.html)就比较容易了。


{% highlight python linenos %}
import argparse, getpass, telnetlib

def main(hostname, username, password):
    t = telnetlib.Telnet(hostname)      #telent ftp.example.com
    t.read_until(b'login:')
    t.write(username.encode('utf-8'))   #输入用户名
    t.write(b'\r')                      #输入回车
    t.read_until(b'assword:')           
    t.write(password.encode('utf-8'))   #输入密码
    t.write(b'\r')                      #输入回车
    n , match, previoust_text = t.expect([br'Login incorrect', br'\$',10])
    if n == 0:
        print("Username and password failed - giving up")
    else:
        t.write(b'exec uptime\r')
        print(t.read_all().decode('utf-8'))

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Use Telnet to log in')
    parser.add_argument('hostname', help='Remote host to tlenet to')
    parser.add_argument('username', help="Remote username")
    args = parser.parse_args()
    password = getpass.getpass('Password: ')
    main(args.hostname,args.username,password)

{% endhighlight %}


## 3 SSH: The Secure Shell ##

### 3.1 An Overview of SSH ###

> SSH：是加密后访问远程主机的application层的协议。通常用于,<br/>
1. 访问主机(加密后的Telnet);<br/>
2. 传输文件(加密后的FTP);<br/>
3. 发送邮件(加密后的POP)<br/>

 传统的网络服务程序，如rsh、FTP、POP和Telnet其本质上都是不安全的；因为它们在网络上用明文传送数据、用户帐号和用户口令，很容易受到中间人（man-in-the-middle）攻击方式的攻击。就是存在另一个人或者一台机器冒充真正的服务器接收用户传给服务器的数据，然后再冒充用户把数据传给真正的服务器。 而SSH是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用SSH协议可以有效防止远程管理过程中的信息泄露问题。通过SSH可以对所有传输的数据进行加密，也能够防止DNS欺骗和IP欺骗。 SSH之另一项优点为其传输的数据可以是经过压缩的，所以可以加快传输的速度。SSH有很多功能，它既可以代替Telnet，又可以为FTP、POP、甚至为PPP提供一个安全的“通道”。

 对于SSH的加密方式需要特别注意。如果读者了解[SSL]({{site.baseurl}}/networks/2015/04/07/网络实战-Part-I-传输层(五)-TLS&SSL.html)的加密：

 1. 对称公钥传输加密内容；

 2. 对称公钥的传输需要非对称公钥的加密和私钥的解密；

 3. 对于发送非对称公钥的身份认证需要第三方CA发放的证书)。

 SSH的加密稍显不同，重点在第三步，没有第三方的证书认证。那么它如何验证非对称公钥发送者的信息呢？答案是通过以下两个方案：

 1. 通常在局域网，SSH给每台主机运行一个脚本来预装记非对称公钥；

 2. SSH不验证对方身份，但是记录该身份对应的公钥。这样可以防止连接某个主机后，该主机被中间人替代。一旦某一时刻公钥和当时记录的不一样，就警告可能对IP欺骗了。


### 3.2 SFTP: File Transfer Over SSH ###

> SFTP：是Secure File Transfer Protocol的缩写，安全文件传送协议。可以为传输文件提供一种安全的加密方法。sftp 与 ftp 有着几乎一样的语法和功能。SFTP 为 SSH的一部分，是一种传输档案至 Blogger 伺服器的安全方式。其实在SSH软件包中，已经包含了一个叫作SFTP(Secure File Transfer Protocol)的安全文件传输子系统，SFTP本身没有单独的守护进程，它必须使用sshd守护进程（端口号默认是22）来完成相应的连接操作，所以从某种意义上来说，SFTP并不像一个服务器程序，而更像是一个客户端程序。SFTP同样是使用加密传输认证信息和传输的数据，所以，使用SFTP是非常安全的。但是，由于这种传输方式使用了加密/解密技术，所以传输效率比普通的FTP要低得多，如果您对网络安全性要求更高时，可以使用SFTP代替FTP。

## 4 总结 ##


{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-17-网络实战(十六)：Telnet & SSH/Telnet & SSH Summary.png)


## 5 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





