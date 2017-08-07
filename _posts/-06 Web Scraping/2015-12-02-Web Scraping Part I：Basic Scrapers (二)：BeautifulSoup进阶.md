---
layout: post
title: Web Scraping Part I：Basic Scrapers (二)：BeautifulSoup进阶
categories: [-06 Web Scraping]
tags: [Web Scraping, BeaufitulSoup]
number: [-4.1.2]
fullview: false
shortinfo: 本文是基于Ryan Mitchell的《Web Scraping With Pyhton》书本的第2篇笔记，将带您全面深入了解BeautifulSoup框架。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}


上篇[文章](https://www.shunmian.me/scraping/2015/12/01/Web-Scraping-Part-I-Basic-Scrapers-(一)-BeautifulSoup入门.html)我们已经介绍过**BeautifulSoup**是一个提供一些简单的、python式的函数用来处理**HTML**和**XML**导航、搜索、修改分析树等功能的解释器。本文我们来详细介绍Advanced HTML Parsing。

## 1 BeautifulSoup的4大类 ##

### 1.1 BeautifulSoup ###

**BeautifulSoup**：该对象表示的是一个文档的全部内容(**文档树**).大部分时候,可以把它当作**Tag**对象，是一个特殊的**Tag**，，我们可以分别获取它的名称，属性，文本以及类型来感受一下。

{% highlight python linenos %}

bsObj = BeautifulSoup(htmlHandler.read(), "html.parser")

print("name:", bsObj.name)                             #name: [document]
print("attrs:", bsObj.attrs)                           #attrs: {}
print("string:", bsObj.string)                         #string: None
print("type:", type(bsObj))                            #type: <class 'bs4.BeautifulSoup'>

{% endhighlight %}

### 1.2 Tag ###
**Tag**：就是HTML中的一个个标签。

{% highlight python linenos %}

people = bsObj.find("span",{"class":"green"})
print(people)                                          #<span class="green">Anna Pavlovna Scherer</span>
print("name:", people.name,type(people.name))          #name: span <class 'str'>
print("attrs:", people.attrs,type(people.attrs))       #attrs: {'class': ['green']} <class 'dict'>
print("string:", people.string,type(people.string))    #string: Anna Pavlovna Scherer <class 'bs4.element.NavigableString'>
print("type:", type(people))                           #type: <class 'bs4.element.Tag'>

{% endhighlight %}

### 1.3 NavigableString ###

**NavigableString**：Tag的string属性的类型，上面的``type(people.string)``即是**NavigableString**。

### 1.4 Comment ###

**Comment**：是一个特殊类型的**NavigableString**对象，表示注释。

## 2 搜索文档树##

### 2.1 findAll() ###

> **findAll(tag, attributes, recursive, text, limit, keywords)**：是BeautifulSoup类型里最重要的方法，它搜索当前tag的所有tag子节点，并判断是否符合过滤器的条件。返回值是bs4.element.ResultSet，ResultSet是List的子类。

在95%的情况下，你只需用到前两个参数。但是我们还是全面的深入了解各参数的意义。

**tag**：可以输入a string或者 a list(tuple也可以，因为list是mutable，tuple是immutable list在Python里) of string as tag names， 例如:

{% highlight python linenos %}

htmlHandler = urlopen("http://www.pythonscraping.com/pages/warandpeace.html")
bsObj = BeautifulSoup(htmlHandler.read(), "html.parser")
names = bsObj.findAll(("h1","h2","h3","h4","h5"))       #输出所有的h1,h2,h3,h4,h5
print(names)

{% endhighlight %}


**attributes**：用attributes dictionary 作为filter条件，返回任何满足attribute的tag，可以用于帅选CSS的Class，例如:

{% highlight python linenos %}
...
names = bsObj.findAll("span",{"class":"green"})         #输出class=green的span
...
{% endhighlight %}


**recursive**：是否搜索所有子孙节点，默认是。如果只想搜索tag的直接子节点,可以使用参数 recursive=False

{% highlight python linenos %}
...
names = bsObj.findAll("span",{"class":"green"}，False)   #输出bsObj的满足class=green的直接子节点span
...
{% endhighlight %}

**text**：搜索满足相应条件的text的Tag。比如要text是"the princce"，即tag的text完全等同于the princce，并不是包括，代码如下：

{% highlight python linenos %}
...
names = bsObj.findAll(text="the prince")                #输出bsObj的满足text="the prince"的所有tag
...
{% endhighlight %}


**limit**：limit是结果的个数。find() 是findAll(limit=1)的情况。

**keyword**：在上述参数不能满足的情况下，keyword可以用来搜索一个特定的attribute(当然也可以用attributes参数)例如搜索所有id="text"的tag：

{% highlight python linenos %}
...
names = bsObj.findAll(id="the prince")                   #输出bsObj的满足id="text"的所有tag
...
{% endhighlight %}


### 2.2 find() ###

> **find(tag, attributes, recursive, text, limit, keywords)**：是BeautifulSoup类型里findAll()方法limit=1的特殊情况，返回当前Tag的子孙节点里第一个满足筛选条件的Tag。
{% highlight python linenos %}
...
people = bsObj.find("span",{"class":"green"})           #返回第一个clss=grenn的span
...

{% endhighlight %}


## 3 遍历文档树 ##

基于目标Tag的name和attribute，我们可以用``find()``和``findAll()``来搜索满足条件的Tag。考虑另一种情况，如果需要基于一个Tag的当前位置来搜索相应的Tag(如兄弟，孩子，子孙，父母Tag)，那么我们就要通过遍历文档树来访问。

### 3.1 Children and Descendants ###

> **Children**：指直接子节点，类比于人类的孩子。通过tag的``.children`` 生成器，可以对tag的孩子子节点进行循环。

{% highlight python linenos %}
...
for child in bsObj.find("table",{"id":"giftList"}).children:      #输出第一个id="giftList"的table Tag的所有孩子Tag
    print(child)                
...
{% endhighlight %}


> **Descendants**：指所有子节点，类比于人类的子孙。**Descendant**包括**Children**。通过tag的``.descendants`` 生成器，可以对tag的子孙节点进行循环。

{% highlight python linenos %}
...
for child in bsObj.find("table",{"id":"giftList"}).descendants:      #输出第一个id="giftList"的table Tag的所有子孙Tag，因此包括孩子，然后一层层剥离。
    print(child)                
...
{% endhighlight %}




### 3.2 Siblings ###

> **Siblings**：指兄妹。通过tag的``.next_sibling(s)`` 生成器，可以对tag的下一兄妹(们)循环；通过tag的``.previous_sibling(s)`` 生成器，可以对tag的上一兄妹(们)循环。


{% highlight python linenos %}
...
for child in bsObj.find("table",{"id":"giftList"}).tr.next_siblings: #输出第一个id="giftList"的table Tag的第一个tr Tag的下一兄妹们。
    print(child)               
...
{% endhighlight %}

{% highlight python linenos %}
...
for child in bsObj.find("table",{"id":"giftList"}).tr.next_siblings: #输出第一个id="giftList"的table Tag的第一个tr Tag的所有上一兄妹们。
    print(child)               
...
{% endhighlight %}

### 3.3 Parents ###

> **Parents**：指父辈。通过tag的``.parent`` 生成器，可以定位到父亲；通过tag的``.parents`` 生成器，可以定位到父亲，爷爷，一直往上到根Tag。


{% highlight python linenos %}
...
print(bsObj.find("img",{"src":"../img/gifts/img1.jpg"}).parent.previous_sibling.get_text())       //找到符合条件的img Tag的父亲的哥哥的文本。       
...
{% endhighlight %}



## 4 Regular Expression ##

### 4.1 Regular Expression Basics ###

Regular Expression的基础请见[这里](https://shunmian.me/string/2015/08/01/Regular-Expression.html)。

### 4.2 Regular Expression With BeautifulSoup ###

在``find()``和``findAll()``函数里，可以应用正则表达式表示函数参数的pattern，进行更高级的Tag搜索。例如

{% highlight python linenos %}
...
import re
...
print(bsObj.findAll("img", src=re.compile("\.\./img/gifts/img.*\.jpg")))       //用Regular Expression对src的pattern进行match。       
...
{% endhighlight %}

## 5 Accessing Attributes ##

Tag的``.attrs``是一个字典，可以对其访问，例如：

{% highlight python linenos %}
...
images= bsObj.findAll("img", src=re.compile("\.\./img/gifts/img.*\.jpg"))
image = images[0]
print(image.attrs['src'])								//输出image这个Tag的src属性。
...
{% endhighlight %}


## 6 Lambda Expression ##

关于**Lambda Expression**的介绍见[这里]({{ site.baseurl}}/functional%20programming/2015/10/01/Functional-Programming-in-Scala(一)_λ-演算-Part-I-表达式-函数和赋值.html#calculus)。简单来说，λ表达式是接受一个变量的匿名函数，可以作为高阶函数的输入或者输出参数。

BeautifulSoup的``find()``和``findAll()``函数也支持λ表达式，如下：

{% highlight python linenos %}
...
tagWith2Attrs = bsObj.findAll(lambda tag: len(tag.attrs)==2) //找到所有属性数为2的Tag
for tag in tagWith2Attrs:
    print(tag.attrs)									
...
{% endhighlight %}

## 7 Summary ##
**Beautiful Soup**框架是一个强大的解析HTML的框架，包括**BeautifulSoup**， **Tag**， **NavigableString**， **Comment**4个类。 其中**BeautifulSoup**对象表示的是一个html后者xml文档的全部内容(**文档树**)。它的``find()``和``findAll()``(通过tag的name， attribute， keywords等参数， 以及正则表达式， λ表达式等)可以快速便捷地搜索Tag， 一个Tag有``.name``， ``.attrs``， ``.string``(NavigableString类)组成，且可以通过``.children``， ``.descendants``， ``.next_sibling(s)``， ``.previous_sibling(s)``， ``.parents(s)``等属性遍历文档树。

最后本文总结成下图以供参考。




{: .img_middle_hg}
![web scraping](/assets/images/posts/2015-12-02/BeautifulSoup进阶.png)



## 8 参考资料 ##

- [《BeautifulSoup Documentation》](https://www.crummy.com/software/BeautifulSoup/bs4/doc/);
- [《Python 3 Documentation》](https://docs.python.org/3/);




