---
layout: post
title: 网络实战(二)：应用层 Part VI：POP
categories: [-02 Networks]
tags: [Networks, HTTP]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第14篇笔记《POP》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 POP ##

> **POP(Post Office Protocol，邮局协议)**：用于电子邮件的接收。本协议主要用于支持使用客户端远程管理在服务器上的电子邮件。POP最大的优点是简单(执行邮件fetch，delete)，这同时也是它最大的缺点(无法辨别已下载的邮件，即客户端和服务器的同步；IMPA可以解决这个问题)。

IMAP在POP基础上提供更多的功能，本文简单介绍POP。对于IMAP的介绍，请见[这里]({{site.baseurl}}/-02%20networks/2015/04/16/网络实战-Part-II-应用层(七)-IMAP.html)

如今最流行的POP版本是3，因此我们常默认POP就是POP3。

Python的[21.14 poplib](https://docs.python.org/3/library/poplib.html)的是对POP3协议的client实现，主要提供连接POP server，mailbox信息收集，下载messages，删除server原始message等4个功能。

### 1.1 Pop Server Compatibility ###

POP服务端的实现在POP标准里并没有指明，因此对于如何同步非常模糊，有些POP服务端只有当你下载才认为是已读，有些即使下载也不认为是已读。因此在使用POP3的时候要格外小心。

{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-15-网络实战(十四)：POP/POP conversation.png)

### 1.2 Connecting and Authenticating ###

{% highlight python linenos %}
import getpass, poplib, sys

def main():
    if len(sys.argv) !=3:
        print("usage: {} hostname username".format(sys.argv[0]))
        sys.exit(2)

    hostname, username = sys.argv[1:]
    p = poplib.POP3(hostname)
    try:
        p.user(username)
        p.pass_(getpass.getpass())

    except poplib.error_proto as e:
        print("Login fail: {}".format(e))

    else:
        print("Your message: {}".format(p.stat()))
    finally:
        p.quit()
        
if __name__=="__main__":
    main()
{% endhighlight %}

### 1.3 Obtaining Mailbox Information ###

{% highlight python linenos %}
import getpass, poplib, sys

def main():
    if len(sys.argv) != 3:
        print('usage: %s hostname username' % sys.argv[0])
        exit(2)
    hostname, username = sys.argv[1:]
    passwd = getpass.getpass()
    p = poplib.POP3_SSL(hostname)
    try:
        p.user(username)
        p.pass_(passwd)
    except poplib.error_proto as e:
        print("Login failed:", e)
    else:
        response, listings, octet_count = p.list()
        if not listings:
            print("No messages")
        for listing in listings:
            number, size = listing.decode('ascii').split()
            print("Message %s has %s bytes" % (number, size))
    finally:
        p.quit()
if __name__ == '__main__':
    main()

{% endhighlight %}

### 1.4 Downloading and Deleting Messages ###

{% highlight python linenos %}
import email, getpass, poplib, sys

def main():
    if len(sys.argv) != 3:
        print('usage: %s hostname username' % sys.argv[0])
        exit(2)

  	hostname, username = sys.argv[1:]
    passwd = getpass.getpass()
    p = poplib.POP3_SSL(hostname)
    try:
        p.user(username)
        p.pass_(passwd)
    except poplib.error_proto as e:
        print("Login failed:", e)
    else:
        visit_all_listings(p)
    finally:
p.quit()
def visit_all_listings(p):
    response, listings, octets = p.list()
    for listing in listings:
        visit_listing(p, listing)
def visit_listing(p, listing):
    number, size = listing.decode('ascii').split()
    print('Message', number, '(size is', size, 'bytes):')
    print()
    response, lines, octets = p.top(number, 0)
    document = '\n'.join( line.decode('ascii') for line in lines )
    message = email.message_from_string(document)
    for header in 'From', 'To', 'Subject', 'Date':
        if header in message:
            print(header + ':', message[header])
    print()
    print('Read this message [ny]?')
    answer = input()
    if answer.lower().startswith('y'):
        response, lines, octets = p.retr(number)
        document = '\n'.join( line.decode('ascii') for line in lines )
        message = email.message_from_string(document)
        print('-' * 72)
        for part in message.walk():
            if part.get_content_type() == 'text/plain':
                print(part.get_payload())
                print('-' * 72)
    print()
    print('Delete this message [ny]?')
    answer = input()
    if answer.lower().startswith('y'):
        p.dele(number)
        print('Deleted.')
if __name__ == '__main__':
    main()
{% endhighlight %}

## 2 总结 ##

{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-15-网络实战(十四)：POP/POP Summary.png)

## 4 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





