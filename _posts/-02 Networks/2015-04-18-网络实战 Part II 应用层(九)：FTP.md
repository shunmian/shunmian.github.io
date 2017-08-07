---
layout: post
title: 网络实战(二)：应用层 Part IX：FTP
categories: [-02 Networks]
tags: [Networks, FTP]
number: [-8.1]
fullview: false
shortinfo: 本文是《Foundations of Python Networking Programming》系列第17篇笔记《FTP》。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}



## 1 FTP：介绍 ##

FTP通常支持4种操作：

1. 下载。

2. 匿名上传。

3. 文件同步在不同主机。

4. 文件系统管理。

### 1.1 替代品 ###

FTP有如下缺点：

1. 数据传输安全性低。所有内容，包括文件，用户名，密码都是用明文传输。

2. FTP Client将所有操作通过1个connection传输。不像HTTP，每一个request是1个独立的connection。

3. 文件系统安全性低。FTP Client可以cd到任何目录(虽然可以通过设置权限来限制)。

因此FTP的替代者有什么呢？

1. 下载文件用HTTPS。

2. 上传文件HTTP POST。

3. 文件同步。rsync或者rdist命令；非编程用户更偏向用云存储，例如用Python写的dropbox等。

4. 文件系统获取。SFTP来增加安全性。

虽然FTP有诸多缺点，是一个比较过时的文件传输协议，但是如果你想通过Python来学习文件传输系统协议，FTP是个不错的起点。


### 1.2 传输通道 ###

ftp有两个channel：

1. **Control Channel**，传输命令，ack或者error；

2. **Data Channel**，传输数据或者块信息。


## 2 FTP：Python使用 ##

{% highlight python linenos %}
from ftplib import FTP

def main():

    ftp = FTP('ftp.ibiblio.org')
    print("Welcome: {}".format(ftp.getwelcome()))
    ftp.login()                                     # login anonymously
    print("Current working directory:",ftp.pwd())   #just like pwd in terminal
    ftp.quit()

if __name__=='__main__':
    main()
{% endhighlight %}

### 2.1 ASCII and Binary Files ###

可以通过FTP的``retrlines()``或者``retrbinary()``来获取ASCII或者byte文件。

{% highlight python linenos %}
from ftplib import FTP
import os

def main_ASCII():
    if os.path.exists('README'):
        raise IOError('refusing to overwrite your README file')

    ftp = FTP('ftp.kernel.org')
    ftp.login()
    ftp.cwd('/pub/linux/kernel')

    with open('README','w') as f:
        def writeline(data):
            f.write(data)
            f.write(os.linesep)
        ftp.retrlines('RETR README',writeline) #Retrieve data in line mode

    ftp.quit()


def main_byte():

    if os.path.exists("patch8.gz"):
        raise IOError('refusing to overwrite your patch8.gz file')

    ftp = FTP('ftp.kernel.org')
    ftp.login()
    ftp.cwd('/pub/linux/kernel/v1.0')

    with open('patch8.gz', 'wb') as f:
        ftp.retrbinary('RETR patch8.gz',f.write) #Retrieve data in binary mode

    ftp.quit()

if __name__ == '__main__':
    main_byte()
{% endhighlight %}

### 2.2 Advanced Binary Downloading ###

{% highlight python linenos %}
from ftplib import FTP
import os,sys

def main():

    fname = 'linux-1.0.tar.gz'
    if os.path.exists(fname):
        raise IOError('refusing to overwrite your {} file'.format(fname))

    ftp = FTP('ftp.kernel.org')
    ftp.login()
    ftp.cwd('/pub/linux/kernel/v1.0')
    ftp.voidcmd('TYPE I')

    socket, size = ftp.ntransfercmd('RETR linux-1.0.tar.gz')
    nbytes = 0

    f = open(fname,'wb')

    while True:
        data = socket.recv(4096)
        if not data:
            break
        f.write(data)
        nbytes +=len(data)
        print('\rdownloading: {:.2%}'.format(nbytes/size), end='')
        sys.stdout.flush()

    f.close()
    socket.close()
    ftp.voidresp()
    ftp.quit()

if __name__=='__main__':
    main()

{% endhighlight %}

### 2.3 Uploading Data ###

略

### 2.4 Advanced Binary Uploading ###

略

### 2.5 Handling Errors ###

``socket.error``, ``IOError``, ``ftplib.all_errors``。

### 2.6 Scanning Directories ###

{% highlight python linenos %}
from ftplib import FTP
import os,sys

def main_list1():
    ftp = FTP('ftp.ibiblio.org')
    ftp.login()
    ftp.cwd('/pub/academic/astronomy/')
    entries = ftp.nlst()        #nlst just list the file name, no other information
    ftp.quit()

    print(len(entries),"entries:")
    for entry in entries:
        print(entry)

def main_list2():
    ftp = FTP('ftp.ibiblio.org')
    ftp.login()
    ftp.cwd('/pub/academic/astronomy/')
    entries = []
    ftp.dir(entries.append)     #dir has more information 
    ftp.quit()

    print(len(entries),"entries:")
    for entry in entries:
        print(entry)

if __name__=='__main__':
    main_list2()
{% endhighlight %}

### 2.7 Detecting Directories and Recursive Download ###

``nlst()``和``dir()``区分不出目录和文件，因此无法区别来下载文件。下面代码显示如何用``cwd()``来递归打印目录(区分文件)。

{% highlight python linenos %}
from ftplib import FTP

def walk_dir(ftp, target_dir):
    source_dir = ftp.pwd()
    try:
        ftp.cwd(target_dir)
    except Exception as e:
        return

    print("{}".format(target_dir))
    names = ftp.nlst()
    for name in names:
        walk_dir(ftp,'{}/{}'.format(target_dir,name))
    ftp.cwd(source_dir)

def main():
    ftp = FTP('ftp.kernel.org')
    ftp.login()
    walk_dir(ftp,'/pub/linux/kernel/Historic/old-versions')
    ftp.quit()

if __name__=='__main__':
    main()
{% endhighlight %}

### 2.8 Creating Directories, Deleting Things ###

其他ftblib文件：

1. ``delete(filename)``，删除文件。

2. ``mkd(dirname)``，创建目录。

3. ``rmd(dirname)``，删除目录。

4. ``rename(oldname,newname)``，重命名。

### 2.9 Doing FTP Securely ###

虽然可以有更好的加密协议来替代FTP，如用SSH的SFTP。但是ftplib是支持TLS的：

1. 建立一个TLS保护的FTP和command channel，可以用``FTP_TLS``来实例化FTP class。之后的登录用的用户名和密码就会被加密。

2. TLS保护的data channel和非保护的data channel，可以用``prot_p()``和``prot_c()``切换。

## 3 总结 ##


{: .img_middle_lg}
![Network Data & Error Summary](/assets/images/posts/2015-04-18-网络实战(十七)：FTP/FTP Summary.png)


## 4 参考资料 ##

- [《Foundations of Python Network Programming》](https://www.amazon.com/Foundations-Python-Network-Programming-Brandon/dp/1430258543/ref=sr_1_1/159-7715257-2675343?s=books&ie=UTF8&qid=1474899055&sr=1-1&keywords=foundations+of+python+network+programming);





