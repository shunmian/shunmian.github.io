---
layout: post
title: Web Scraping A1：Selenium
categories: [-06 Web Scraping]
tags: [Web Scraping, BeaufitulSoup]
number: [-4.1.15]
fullview: false
shortinfo: 本文是对Selenium的Python库做一个入门介绍。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}


## 1 介绍 ##

对于大量使用Ajax或者HTML的动态网页，数据的更新并不依赖网页重载，HTML源码和当前显示的内容可能并不一致。由于没能正确执行JavaScript，这个时候你的Web Scraper可能就会失效。解决这两种情况只有两种方法：

1. 直接scrape JavaScript；

2. 使用能**执行JavaScript**的Python module然后从网页中scrape就像你正常浏览一个网站一样。

而后者基于强大的第三方库，可以使得我们的工作更加有效率。在这些执行JavaScript的第三方库中，以**Selenium**最为著名。

> **Selenium**：a powerful **web scraping tool** developed originally for website testing by **automating browsers** to load the website, retrieve the required data, and even take screenshots or assert that certain actions happen on the website。

### 1.1 Selenium 安装和示例###

[Selenium](http://www.seleniumhq.org/)是一个Python Module，可以用``pip3 install selenium``下载安装。**Selenium**没有自己的浏览器，必须和第三方浏览器(同时得下载相应浏览器的驱动)结合使用。你在使用的时候会发现有一个浏览器弹出，按照你写好的程序运行。但是你可以用一个**PhantomJS**的**headless browser**使你的程序安静运行在后台而不会弹出浏览器。

如果你要用Selenium运行**Chrome**，则需要到[ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/)下载其驱动(不是一个Python Module，所以需要你手动下载到程序)。然后将其二进制执行文件的路径在实例化Chrome的时候作为``executable_path``传入。下面是一个简单的例子，用Chrome来打开google，输入cheese！并搜索，最后关闭Chrome浏览器。

{% highlight python linenos %}

from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

#1. 实例化一个WebDriver类，基于Chrome浏览器
driver = webdriver.Chrome(executable_path="/Users/LAL/PycharmProjects/Python/WebScrapingWithPython/Chapter10_Scraping JavaScript/Selenium Tutorial/chromedriver")										

#2. 在浏览器地址栏中输入http://www.google.com并回车
driver.get("http://www.google.com")   			

#3. 将当前页的标题打印出来
print(driver.title)

#4. 找到搜索框(元素名字是"q")
inputElement = driver.find_element_by_name("q")

#5. 搜索框输入"cheese!"并回车
inputElement.send_keys("cheese!")
inputElement.send_keys(Keys.RETURN)

#6. 等到0-10秒，直到当前页面的标题改为"cheese!"，意味着搜索结果页面已加载完成，并打印标题
try:
    WebDriverWait(driver,10).until(EC.title_contains("cheese!"))
    print(driver.title)

finally:
    print("chrome task finished")

{% endhighlight %}

如果你要用Selenium运行**PhantomJs**，同理需要到[PhantomJS](http://phantomjs.org/)下载其驱动然后将其二进制执行文件的路径在实例化PhantomJS的时候作为``executable_path``传入。


### 1.2 类图 ###

前面我们简单介绍了下Selenium的用法，现在我们来看看**selenium module**里的类关系。里面**WebDriver**是主类，其他几个类**WebElement**，**WebDriverWait**(和exptected)，**Keys**，**Select**，**ActionChains**，都围绕着**WebDriver**：

1. 通过**WebElement**的``find_element(s)_by_name``等定位方法，可以返回**WebElemnt实例**。

2. Tag为select的**WebElemnt**可以用作**Select**的构造函数的参数来选择子项，提交表单。

3. **Keys**自定义了一些键盘上的特殊符号。

4. **ActionChains**可以形成一系列的操作。

5. **WebDriverWait**用来做显示等待Ajax页面载入。

{: .img_middle_lg}
![web scraping](/assets/images/posts/2015-12-15/Selenium Class relationship.png)

## 2 WebDriver API ##

{: .img_middle_lg}
![web scraping](/assets/images/posts/2015-12-15/Selenium WebDriver Summary.jpg)


## 3 参考资料 ##

- [《Selenium WebDriver》](http://www.seleniumhq.org/docs/03_webdriver.jsp#introducing-webdriver);
- [《Selenium WebDriver Python Documentation》](https://seleniumhq.github.io/selenium/docs/api/py/api.html);
- [《Selenium with Python》](http://selenium-python.readthedocs.io/);



