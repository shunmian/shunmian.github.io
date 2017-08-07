---
layout: post
title: 网络实战(二)：应用层 Part IV：Building and Parsing E-mail
categories: [-02 Networks]
tags: [Networks, HTTP]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第12篇笔记《Building and Parsing E-mail》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

本文是接下来4篇e-mail笔记中的第1篇，重点介绍如何在邮件中正确包括多媒体和国际化，这为后面下面3个e-mail协议打下基础。

1. **SMTP(Simple Mail Transport Protocol，简单邮件传输协议)**，传送e-mail从原地址到目的地址服务器，然后准备被某个收件人读取。

2. **POP(Post Office Protocol，邮局协议)**，采用Client/Server工作模式，Client被称为客户端，一般我们日常使用电脑都是作为客户端，而Server（服务器）则是网管人员进行管理的。举个形象的例子，Server（服务器）是许多小信箱的集合，就像我们所居住楼房的信箱结构，而客户端就好比是一个人拿着钥匙去信箱开锁取信一样的道理。

3. **IMAP(Internet Message Access Protocol，交互邮件访问协议)**，它的主要作用是邮件客户端（例如MS Outlook Express)可以通过这种协议从邮件服务器上获取邮件的信息，下载邮件等。

3个协议的关系可以简单描述成：编写邮件，传送邮件到目的地服务器(SMTP)，client从服务器读取邮件(POP或IMAP)。

## 1 E-Mail Protocol Introduction ##

### 1.1 E-Mail Message Format ###

{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-13-网络实战(十二)：Building and Parsing E-mail/E-mail Formating.png)

 
### 1.2 Building an E-Mail Message ###

#### 1.2.1 Basic Building ####

{% highlight python linenos %}
import email.message, email.policy, email.utils

text = '''Hello,
This is a basic message from Chapter 12 - Anonymouse'''

def main():
    message = email.message.EmailMessage(email.policy.SMTP)
    message['To'] = 'shunmian@gmail.com'
    message['From'] = 'Test Sender<sender@example.com>'
    message['Subject'] = 'Text Message, Chapter 12'
    message['Date'] = email.utils.formatdate(localtime=True)
    message['Message-ID'] = email.utils.make_msgid()
    message.set_content(text)
    print(message.as_string())

if __name__=='__main__':
    main()
{% endhighlight %}

``EmailMessage``是表示email message的类，其中包含了message格式里的上图中的各个field。

#### 1.2.2 MIME Building ####

> MIME(Multipurpose Internet Mail Extensions, 多用途互联网邮件扩展): MIME出台之前，使用RFC 822只能发送基本的ASCII码文本信息，邮件内容如果要包括二进制文件、声音和动画等，实现起来非常困难。MIME提供了一种可以在邮件中附加多种不同编码文件的方法，弥补了原来的信息格式的不足。实际上不仅仅是邮件编码，现在MIME经成为HTTP协议标准的一个部分。

下面代码实现了plain，html，附件，inline image等多种功能的邮件。

{% highlight python linenos %}

import email.message, email.policy, email.utils, mimetypes, argparse, sys

plain = """Hello,
This is a MiME Message from Chapter 12.
= Anonymous """

html = """<p>Hello,</p>
<p>This is a <b> test message</b> from Chapter 12.</p>
<p>- <i>Anonymous</i></p>"""

img = """<p> This is the smallest possible blue GIF:</p>
<img src ="cid:{}" height = "80" width = "80">"""

blue_dot = (b'GIF89a1010\x900000\xff000,000010100\x02\x02\x0410;'
            .replace(b'0', b'\x00').replace(b'1', b'\x01'))

def main(args):
    message = email.message.EmailMessage(email.policy.SMTP)
    message['To'] = 'Test Recipient <recipient@example.com>'
    message['From'] = 'Test Sender <sender@example.com>'
    message['Subject'] = 'Foundations of Python Network Programming'
    message['Date'] =  email.utils.formatdate(localtime=True)
    message['Message-ID'] = email.utils.make_msgid()

    if not args.i:
        message.set_content(html,subtype='html')
        message.add_alternative(plain)
    else:
        cid = email.utils.make_msgid()
        message.set_content(html + img.format(cid.strip('<>')), subtype='html')
        message.add_related(blue_dot,'image','gif',cid=cid,filename='blue-dot.gif')
        message.add_alternative(plain)

    for filename in args.filename:
        mime_type, encoding = mimetypes.guess_type(filename)
        if encoding or (mime_type is None):
            mime_type = 'application/octet-stream'
        main, sub = mime_type.split('/')
        if main == 'text':
            with open(filename, encoding = 'utf-8') as f:
                text = f.read()
            message.add_attachment(text,sub,filename=filename)

        else:
            with open(filename,'rb') as f:
                data = f.read()
            message.add_attachment(data,main,sub,filename=filename)

    sys.stdout.buffer.write(message.as_bytes())

if __name__=='__main__':
    parser = argparse.ArgumentParser(description='Build, print a MIME email')
    parser.add_argument('-i', action = 'store_true', help='Include GIF image')
    parser.add_argument('filename', nargs='*',help='Attachment filename')
    main(parser.parse_args())

{% endhighlight %}

4种添加content方法总结：

1. ``set_content``：设置主要内容；

2. ``add_related``...：主要内容的相关内容，例如inline image；

3. ``add_attachment``...：添加附件；

4. ``add_alternative``...：添加可选格式，例如``set_contet(html, subtype='html')``后，``add_alternative(plain)``添加了文本。


### 1.3 Parsing E-Mail Message ###

为了解析邮件，python的email模块提供了两种方法：

1. **EmailMessage有便利的门面模式提供message的各个field**，前提是接收来的message严格按照MIME格式。

2. **人工访问message**，来决定如何处理每个field。

#### 1.3.1 Basic Building ####

{% highlight python linenos %}
import argparse, email.policy, sys

def main(binary_file):
    policy = email.policy.SMTP
    message = email.message_from_binary_file(binary_file,policy=policy)
    for header in ['From', 'To', 'Date', 'Subject']:                    #header
        print(header + ':', message.get(header,'(none)'))
    print()

    try:                                                                #body
        body = message.get_body(preferencelist=('plain','html'))
    except KeyError:
        print('<This message lacks a printable text or HTML body>')
    else:
        print(body.get_content())


    for part in message.walk():                                         #attachment
        cd = part['Content-Disposition']
        is_attachment = cd and cd.split(';')[0].lower() == 'attachment'
        if not is_attachment:
            continue

        content = part.get_content()
        print('* {} attachment named {!r}: {} object of length {}'.format(part.get_contetn_type().part.get_filename(),type(content).__name__,len(content)))


if __name__=='__main__':
    parser = argparse.ArgumentParser(description='Build, print a MIME email')
    parser.add_argument('filename', nargs='?', help='File containing an email')
    args = parser.parse_args()

    if args.filename is None:
        main(sys.sydin.buffer)
    else:
        with open(args.filename,'rb') as f:
            main(f)

{% endhighlight %}

terminal中运行下列命令，即可看到基本解析输出


{% highlight python linenos %}
$ python3 build_basic_email.py > email.txt   #stdout输出到email.txt而不是默认的print。
$ python3 display_email.py email.txt
{% endhighlight %}

#### 1.3.2 Walking MIME Parts ####

略。

#### 1.3.3 Header Encodings ####

特殊字符编码。略。

#### 1.3.4 Parsing Dates ####

解析邮件日期有两种方法，都在email.utils模块里：

1. parserdate()和parsedate_tz()，返回旧式C语言的date表示。

2. parsedate_to_datetime()，返回1个datetime object。

现在有第3个选择，第三方库[pytz](http://pytz.sourceforge.net/)，用来处理date。

## 2 总结 ##

{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-13-网络实战(十二)：Building and Parsing E-mail/E-mail Summary.png)

## 3 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





