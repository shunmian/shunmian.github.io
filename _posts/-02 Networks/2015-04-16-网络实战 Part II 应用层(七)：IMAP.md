---
layout: post
title: 网络实战(二)：应用层 Part VII：IMAP
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


待续

## 1 IMAP ##

> **IMAP(Internet Mail Access Protocol，Internet邮件访问协议)**：和POP一样，用于客户端从服务器管理邮件，但是提供更多服务，比如同步，加标签，搜索，上传message，文件夹管理，部分下载等。

### 1.1 IMAP Basics ###

{% highlight python linenos %}
import getpass, imaplib, sys

def main():
    if len(sys.argv) !=3:
        print("usage: {} hostname, username".format(sys.argv[0]))
        sys.exit(2)

    SERVER, USERNAME = sys.argv[1:]
    m = imaplib.IMAP4_SSL(SERVER)
    m.login(USERNAME,getpass.getpass())

    try:
        print('Capabilities {}'.format(m.capability()))
        print('Listing mailboxes ')
        status, data = m.list()
        print('Status: {}'.format(status))
        print('Data:')
        for datum in data:
            print(repr(datum))
    finally:
        m.logout()

if __name__=='__main__':
    main()

"""
输出：
Capabilities ('OK', [b'IMAP4rev1 UNSELECT IDLE NAMESPACE QUOTA ID XLIST CHILDREN X-GM-EXT-1 UIDPLUS XXXXXXXXXXXXXXXX ENABLE MOVE XXXXXXXXX ESEARCH UTF8=ACCEPT LIST-EXTENDED LIST-STATUS LITERAL- APPENDLIMIT=35651584', b'IMAP4rev1 UNSELECT IDLE NAMESPACE QUOTA ID XLIST CHILDREN X-GM-EXT-1 UIDPLUS XXXXXXXXXXXXXXXX ENABLE MOVE XXXXXXXXX ESEARCH UTF8=ACCEPT LIST-EXTENDED LIST-STATUS LITERAL- APPENDLIMIT=35651584'])
Listing mailboxes 
Status: OK
Data:
b'(\\HasNoChildren) "/" "Deleted Items"'
b'(\\HasNoChildren) "/" "Google Calendar"'
b'(\\HasNoChildren) "/" "HKU"'
b'(\\HasNoChildren) "/" "HKU_EEE"'
b'(\\HasNoChildren) "/" "INBOX"'
b'(\\HasNoChildren) "/" "Junk E-mail"'
b'(\\HasNoChildren) "/" "Mar_1"'
b'(\\HasNoChildren) "/" "Travel"'
b'(\\HasNoChildren) "/" "Work"'
b'(\\Noselect \\HasChildren) "/" "[Gmail]"'
b'(\\HasChildren \\HasNoChildren) "/" "[Gmail]/All Mail"'
b'(\\HasNoChildren) "/" "[Gmail]/Drafts"'
b'(\\HasChildren \\HasNoChildren) "/" "[Gmail]/Sent Mail"'
b'(\\HasNoChildren) "/" "[Gmail]/Spam"'
b'(\\HasNoChildren) "/" "[Gmail]/Starred"'
b'(\\HasChildren \\HasNoChildren) "/" "[Gmail]/Trash"'
"""

{% endhighlight %}

imaplib没有提供对data的解析(只显示raw data，例如上面的``b'(\\HasNoChildren) "/" "INBOX"'``)，这个时候就需要1个更强大的IMAP client library了。




#### 1.1.1 IMAP Client ####

第三方库[IMAPClient](https://imapclient.readthedocs.io/en/stable/)提供了更强大的IMAP client功能。

{% highlight python linenos %}
import getpass, sys
from imapclient import IMAPClient


def main():
    if len(sys.argv) !=3:
        print("usage: {} hostname, username".format(sys.argv[0]))
        sys.exit(2)

    SERVER, USERNAME = sys.argv[1:]
    c = IMAPClient(SERVER,ssl=False)

    try:
        c.login(USERNAME, getpass.getpass())
    except c.Error as e:
        print('Could not log in:', e)
    else:
        print('Capabilities {}'.format(c.capabilities()))
        print('Listing mailboxes ')
        data = c.xlist_folders()
        for flags, delimiter, folder_name in data:
            print('{:30s} {} {}'.format(''.join(str(flags)), delimiter, folder_name))
    finally:
        c.logout()

if __name__=='__main__':
    main()

"""
输出
Capabilities: ('IMAP4REV1', 'UNSELECT', 'IDLE', 'NAMESPACE', 'QUOTA', 'XLIST', 'CHILDREN', 'XYZZY',
'SASL-IR', 'AUTH=XOAUTH')
Listing mailboxes:
\HasNoChildren                  / INBOX
\HasNoChildren                  / Personal
\HasNoChildren                  / Receipts
\HasNoChildren                  / Travel
\HasNoChildren                  / Work
\Noselect \HasNoChildren        / [Gmail]
\HasChildren \HasNoChildren     / [Gmail]/All Mail
\HasNoChildren                  / [Gmail]/Drafts
\HasChildren \HasNoChildren     / [Gmail]/Sent Mail
\HasNoChildren                  / [Gmail]/Spam
\HasNoChildren                  / [Gmail]/Starred
\HasChildren \HasNoChildren     / [Gmail]/Trash
"""
{% endhighlight %}

#### 1.1.2 Examining Folders ####

IMAP protocol is stateful，即当你进行下载，搜索或者修改任何message等操作时，你需要选择要操作的文件夹。当你选择了某个文件夹作为当前文件夹后，你的操作就在该文件夹下。

#### 1.1.3 Message Numbers vs. UIDs ####

IMAP协议提供了两个数字来追踪message：

1. **Message Number**：某个连接下的特定文件夹内的message的编号，1，2，3等。如果关闭该次连接，下次在连接式，同样的message，其message nnumber可能会不一样。

2. **UIDs**：每次连接都一样的编号。对于同步特别有用，也是IMAP优于POP的1个特点。

#### 1.1.4 Message Ranges ####

IMAP提供了1次操作1个或者多个message的机制，比如下面命令：

{% highlight python linenos %}
2，4：6，20：*
#表示message2，4到6，20及20以后所有的message。
{% endhighlight %}

#### 1.1.5 Summary Information ####

IMAPClient的``select_folder(foldername)``返回foldername的元信息，存储在dict里。不同key代表不同含义：

1. **EXISTS**：该folder里的message个数。

2. **READ-WRITE**：是否可读写。

3. **UIDNEXT**：server对下个新来的message预备的UID。

4. **RECENT**：上次登录之后新来的message个数。

{% highlight python linenos %}
import getpass, sys
from imapclient import IMAPClient

def main():
    if len(sys.argv) !=4:
        print("usage: {} hostname, username foldername".format(sys.argv[0]))
        sys.exit(2)

    SERVER, USERNAME, FOLDERNAME = sys.argv[1:]
    c = IMAPClient(SERVER,ssl=False)

    try:
        c.login(USERNAME, getpass.getpass())
    except c.Error as e:
        print('Could not log in:', e)
    else:
        select_dict = c.select_folder(FOLDERNAME,readonly=True)
        for k,v in sorted(select_dict.items()):
            print('%s: %r' % (k,v))
    finally:
        c.logout()

if __name__=='__main__':
    main()

"""
输出
EXISTS: 3
PERMANENTFLAGS: ('\\Answered', '\\Flagged', '\\Draft', '\\Deleted',
                 '\\Seen', '\\*')
READ-WRITE: True
UIDNEXT: 2626
FLAGS: ('\\Answered', '\\Flagged', '\\Draft', '\\Deleted', '\\Seen')
UIDVALIDITY: 1
RECENT: 0
"""
{% endhighlight %}

### 1.2 IMAP Tasks ###

#### 1.2.1 Downloading ####

##### 1.2.1.1 Downloading an Entire Maibox #####

{% highlight python linenos %}
import getpass, sys, email
from imapclient import IMAPClient

def main():
    if len(sys.argv) !=4:
        print("usage: {} hostname, username foldername".format(sys.argv[0]))
        sys.exit(2)

    SERVER, USERNAME, FOLDERNAME = sys.argv[1:]
    c = IMAPClient(SERVER,ssl=False)

    try:
        c.login(USERNAME, getpass.getpass())
    except c.Error as e:
        print('Could not log in:', e)
    else:
        print('Login succesfully')
        print_information(c, FOLDERNAME)
    finally:
        c.logout()

def print_information(c,foldername):

    c.select_folder(foldername,readonly=True)
    msgdict = c.fetch('1:*',['BODY.PEEK[]'])

    for message_ID, message in list(msgdict.items()):
        e = email.message_from_bytes(message[b'BODY[]'])
        print(e['From'])
        payload = e.get_payload()
        if isinstance(payload,list):
            part_content_types = [part.get_content_type() for part in payload]
            print(' Parts:', ' '.join(part_content_types), '...')
        else:
            print(' ', ' '.join(payload[:60].split()), '...')

if __name__=='__main__':
    main()
{% endhighlight %}

这是下载邮件的最简单形式，当然如果文件夹很大而且附件很大的话，就需要很长的下载时间。

##### 1.2.1.2 Downloading Messages Individually #####

上面下载整个文件夹里的message这种使用情境在实际情况中比较少见，通常是下载某一个message。

略。


#### 1.2.2 Flagging Messages ####

前面对每一个Message都有会mark 1个attribute称为flag，它们在\之后显示，通常有下面几种：

1. **\Answered**: The user has replied to the message.

2. **\Draft**: The user has not finished composing the message.

3. **\Flagged**: The message has somehow been singled out specially; the purpose  and meaning of this flag vary between e-mail readers.

4. **\recent**: No IMAP client has seen this message before. This flag is unique int hat the flag cannot be added or removed by normal commands; it is automatically removed after the mailbox is selected.

5. **\Seen**: The message has been read.

你可以通过下面命令来读写message的flag。

{% highlight python linenos %}
c.get_flags(2703)
c.remove_flags(2703, ['\\Seen'])
c.add_flags(2703, ['\\Answered'])
{% endhighlight %}

#### 1.2.3 Deleting Messages ####

删除message有两个步骤：

1. mark message as **"\delete"**;

2. call ``expunge()``。

{% highlight python linenos %}
c.delete_messages([2703, 2704])
c.expunge()
{% endhighlight %}

#### 1.2.4 Searching ####

{% highlight python linenos %}
>>> c.select_folder('INBOX')
>>> c.search('SINCE 13-Jul-2013 TEXT Apress')
[2590L, 2652L, 2653L, 2654L, 2655L, 2699L] #UID
{% endhighlight %}

#### 1.2.5 Manipulating Folders and Messages ####

通过IMAPClient创建和删除文件夹比较简单。

{% highlight python linenos %}
c.create_folder('Personal')
c.delete_folder('Work')
{% endhighlight %}

#### 1.2.6 Asynchrony ####

IMAPClient本身没提供Asynchrony，但是在其他包含IMAPClient的库里有提供，例如[Twisted Python](https://twistedmatrix.com/trac/)。

## 2 总结 ##

{: .img_middle_hg}
![Network Data & Error Summary](/assets/images/posts/2015-04-15-网络实战(十五)：IMAP/Email Summary.png)


## 3 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





