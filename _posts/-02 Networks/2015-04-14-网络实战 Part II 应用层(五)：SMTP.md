---
layout: post
title: 网络实战(二)：应用层 Part V：SMTP
categories: [-02 Networks]
tags: [Networks, HTTP]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第13篇笔记《SMTP》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}


## 1 SMTP Introduction ##

> **SMTP（Simple Mail Transfer Protocol，简单邮件传输协议)：**,它是一组用于由源地址到目的地址传送邮件的规则，由它来控制信件的中转方式。

SMTP在邮件发送的两个阶段都会使用到：

1. **Submission：E-mail client到E-mail server。**用户输入E-mail message，提交给E-mail server，由它来发送到目的地。

2. **Delivery：E-mail server到目的地E-mail server**。

### 1.1 Submission: E-mail Clients vs. Webmail Services ###

**Submission** 分两种情况，通过E-mail Clients和Webmail Services。

当用户点击**Send**后，submission+delivery就开始了。在到达最终目的地E-mail server之前，邮件需要经过多个E-mail server节点和不同的delay。

E-mail程序的发展经历了下面几个阶段：

1. **In the Beginning Was the Command Line**。最早的时候，用户通过命令行来发送邮件。通常需要输入用户名和密码，然后提交至e-mail daemon(运行email outgoing queue)。这类程序主要有mailx, elm, pine mutt等。

2. **The Rise of Clients**。在那之后的用户更习惯于图形界面而不是命令行，这导致了两个最为流行的E-mail Client的兴起，即Mozilla Thunderbird和Microsoft Outlook(直到今天仍是这样)。但是这类e-mail clients有几个明显的缺点，时刻保持网络连接来接收邮件，额外的邮件备份防止本地主机崩溃，和足够的等待来完成邮件发送完成。但是计算机正是为解决当下问题而不断向前进步的。开发者提出了两个新的协议，**POP(Post Office Protocol，邮局协议)**和**IMAP(Internet Message Access Protocol，交互邮件访问协议)**，它们通过用户名和密码认证，full-time的email server来持久化存储邮件，以及通过不同端口利用SMTP来接收(Port 587)和发送(Port 25)邮件来解决上面提到的问题。在这种情况下，e-mail client更像是1个操控远程e-mail server的窗口，来fetch和send及delete email等。只是最初配置Thunderbird或者Outlook需要比较繁琐的步骤，比如接收和发送的e-mail server的地址，用户名和密码等。

3. **The Move to Webmail**。早期的用户仍然记得要安装大量的协议来使用客户端。但是**World Wide Web**的出现使得一切都变得简单了，互联网的使用只需要1个客户端，即**web browser**。**World Wide Web**不仅代替了许多Intenet protocol，例如ftp，来浏览文件；而且取消了客户端的需要。所有一切都可以通过**web browser**来完成。**Webmail**就是在这样的背景下产生并替代**E-mail Client**的，比如现在的“Gmail”和“Yahoo！”，甚至是服务端的SquirrelMail。这种从**E-mail Clients到Webmail Services的转变**对于用户来说意味着什么呢？使得用户回到了最初的**面向用户的简单的邮件收取的使用**而根本不需要碰触和直到网络协议(网络协议属于Gmail背后的google来实现)。对于用户来说，通过Webmail来收发邮件就像1个黑箱子一样，背后的协议不用看到也看不到。


### 1.2 How SMTP Is Used ###

上面介绍的是E-mail发展的big picture，而本文重点是在SMTP协议上。

#### 1.2.1 Sending E-Mail ####

Python用户可以使用smtplib来直接发送邮件，但是这样做有1个弊端，就是该主线程会阻塞直到邮件发送成功到目的地服务器，而这个过程可以是几秒，几分钟，几个小时甚至几天。一般情况下我们不会用Python的smtplib直接发送邮件至目的地server，而是发送到1个该邮件客户端用户名和密码认证过的e-mail server，然后通过它的retry queue来发送至目的地e-mail server。这样就解放了当前的Python主线程，干其他事情。

#### 1.2.2 Headers and the Envelope Recipient ####

大部分邮件使用者都被会被以下几个术语所困惑：

1. **To**：收件人，即该邮件主要负责人

2. **Cc(Carbon Copy)**：抄送，所有收件人都会看到该邮件被发至Cc这个收件人。Cc主要用来提醒该邮件内容到涉及的人员。

3. **Bcc(Blind Carbon Copy)**：密送。和Cc一样，只是其他收件人不知道Bcc也是收件人。

举个栗子，领导发送1封办公室打扫的邮件To小明，Cc小红和小花，Bcc给小军。那么很有可能小明是负责办公室打扫的，小红和小花是受办公室打扫影响的人(比如来帮小明)，小军是整个公司监督小明工作的人(细思极恐啊...)。

**To，Cc，Bcc**并不是SMTP协议的一部分，而是由**E-mail client**实现的。这里引出两个概念：

1. **Message's Header**：SMTP的一部分，message的header里包括了收件人，寄件人等。

2. **Envelope's Recipient**：投递的时候信封上的收件人和寄件人，真正涉及到投递的地址。和**Message‘s Header**不一样(只是显示在Message里，不参与具体投递工作)。

在这两个概念基础上，Bcc的实现就比较容易理解了，即Bcc不在**Message‘s Header**里，但是由E-mail Client加入到**Envelope's Recipient**。

这样的投递与message的寄件人和收件人的解耦带来了许多好处，使得Bcc以及其它类似mailing group list功能的实现只涉及到客户端，而不涉及信封里面的SMTP协议本身。

#### 1.2.3 Multiple Hops ####

一封邮件从A发送到brandon@gatech.edu需要经过多次Hop才能到达，而每次Hop的地址通过不断改变Envelope Recipient来实现。在这个情形中：

1. **1st Hop：**A的 e-mail provider 会查看gatech.edu的DNS，获取IP地址，然后发送去该地址。

2. **2nd Hop：**但是gatech.edu的邮件server负责整个大学的email，因此它会查表，找到brandon所在的department，最后发现官方完整email地址是：brandon.rhodes@oit.gatech.edu。然后获取该OIT地址的DNS，并投递。

3. **3rd Hop：**OIT会首先将邮件投递到spam3.oit.gatech.edu.进行垃圾邮件过滤。

4. **4th Hop：**如果通过，则随机投递到8个冗余e-mail server之一，例如这里是mail7. oit.gatech.edu。

5. **5th Hop：**mail7然后查看最终收件箱被哪个邮件系统存储，然后投递到brandon.rhodes@oit.gatech.edu。

这就是为什么邮件的投递经常会历经几十秒甚至几分钟。在这个multiple hop的过程中，message header里的**Received header**会记录Hop的路径，因此它是邮件系统管理员debug的重要信息来源。



## 2 SMTP Library ##

Python的[21.17 smtplib](https://docs.python.org/3.4/library/smtplib.html)是对SMTP协议的内建实现。

{% highlight python linenos %}
import sys, smtplib
message_template = """To: {}
From: {}
Subject: Test Message from simple.py
Hello,
This is a test message sent to you from the simple.py program
in Foundations of Python Network Programming.
"""
def main():
    if len(sys.argv) < 4:
        name = sys.argv[0]
        print("usage: {} server fromaddr toaddr [toaddr...]".format(name))
        sys.exit(2)
    server, fromaddr, toaddrs = sys.argv[1], sys.argv[2], sys.argv[3:]      
    message = message_template.format(', '.join(toaddrs), fromaddr)         #message header
    connection = smtplib.SMTP(server)
    connection.sendmail(fromaddr, toaddrs, message)                         #envelope recipient
    connection.quit()
    s = '' if len(toaddrs) == 1 else 's'
    print("Message sent to {} recipient{}".format(len(toaddrs), s))

if __name__ == '__main__':
    main()
{% endhighlight %}

这里可以看到Envelop recipient和message header是不一样的。

### 2.1 Error Handling and Conversation Debugging ###

{% highlight python linenos %}
import sys, smtplib, socket
message_template = """To: {}
From: {}
Subject: Test Message from simple.py
Hello,
This is a test message sent to you from the debug.py program
in Foundations of Python Network Programming.
"""
def main():
    if len(sys.argv) < 4:
        name = sys.argv[0]
        print("usage: {} server fromaddr toaddr [toaddr...]".format(name))
        sys.exit(2)
    server, fromaddr, toaddrs = sys.argv[1], sys.argv[2], sys.argv[3:]
    message = message_template.format(', '.join(toaddrs), fromaddr)
    try:
        connection = smtplib.SMTP(server)
        connection.set_debuglevel(1)                            # set debug level to get detailed sending information
        connection.sendmail(fromaddr, toaddrs, message)
    except (socket.gaierror, socket.error, socket.herror,
            smtplib.SMTPException) as e:
        print("Your message may not have been sent!")
        print(e)
        sys.exit(1)
    else:
        s = '' if len(toaddrs) == 1 else 's'
        print("Message sent to {} recipient{}".format(len(toaddrs), s))
        connection.quit()
if __name__ == '__main__':
    main()
{% endhighlight %}

### 2.2 Getting Information from EHLO ###

略。

### 2.3 Using SSL & TLS ###

略。

### 2.4 Authenticated SMTP ###

略

### 2.5 SMTP Tips ###

略

## 3 总结 ##

{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-14-网络实战(十三)：SMTP/SMTP Summary.png)

## 4 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





