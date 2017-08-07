---
layout: post
title: FP in Scala(一)：λ-演算 Part II：高阶函数和柯里化
categories: [06 Functional Program]
tags: [High Order Function, Currying]
number: [-2.1]
fullview: false
shortinfo: 我们在讨论Currying的时候，究竟在讨论什么？
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. 问题的由来 ##

我们在上一节中提到，其实函数和变量在更高一级的抽象上是一样的，它们的声明和定义都有**关键词**，**名称**，**类型**，**字面值**。那么你或许会问，那什么是区别于函数和数据的本质呢？答案是**有无输入**。如果仔细看下图(函数有输入不是常识么？别急，我们一步一步剖析函数和数据的异同)，你会发现函数有**输入**，这体现在函数的**类型**和**完整字面值(Block)**上。

{: .img_middle_mid}
![function and variable](/assets/images/posts/2015-10-02/function and variable.png)

这一不同将打开一扇崭新的大门，引出一系列深刻的问题：

+ **函数**作用在**数据**上可以**输入数据**和**输出数据**，**函数**能否作用在**函数**上？
+ **函数**有**输入**，那么**输入**的**参数个数**之间有没有关系呢？

这就是我们下面要讨论的**高阶函数(函数输入输出与函数有关)**和**柯里化(函数输入个数的关系)**。



## 2. 高阶函数和柯里化 ##

### 2.1 高阶函数 ###

在scala里，函数作为一等公民，享有和其他数据(也是一等公民)一样的待遇，包括作为函数的输入和输出。这样使得函数的输入输出除了可以是数据外，还可以是函数。这就引出一个函数阶级的问题：

>**一阶函数**：输入输出都为数据；<br/>
**高阶函数**：输入输出至少有一个为函数。

我们先从一个简单的求和问题来，下面是两个求和函数，上下界分别是`a`和`b`：

+ sumInts     = &sum;<sub>k=a</sub><sup>b</sup> k；
+ sumCube = &sum;<sub>k=a</sub><sup>b</sup> k<sup>3</sup>；
它们的实现如下：

{% highlight scala linenos %}

def sumInts(a:Int,b:Int):Int = {
  if (a > b) 0
  else a + sumInts(a+1,b)
}                                               
  
sumInts(1,3)                         // 1+2+3 = 6          

def sumCube(a:Int,b:Int):Int = {
  if (a > b) 0
  else a*a*a + sumCube(a+1,b)
}                                          
    
sumCube(1,3)                        // 1*1*1 + 2*2*2 + 3*3*3 = 35

{% endhighlight %}

这两个函数可以看到都是一阶函数。但是这两个函数有共性：都是求和，且上下界都为a和b。不同的是求和的单项不一样。我们能不能把它的共性抽象出来，使得任何满足这种共性的函数不用每次都写递归求和呢？请看下面代码：

{% highlight scala linenos %}

def sum(f:Int=>Int, a:Int,b:Int):Int = {            //高阶函数，输入是一个函数，a，b
    if(a>b) 0 else f(a)+sum(f,a+1,b)
}

def sumInts(a:Int,b:Int) = sum((x:Int) => x, a, b)  //一阶函数，输入两个Int，输出一个Int
                                                  
sumInts(1,3)                                        //6
  
def sumCube(a:Int,b:Int) = sum((x:Int)=>x*x*x,a,b)  //一阶函数，输入两个Int，输出一个Int
                                                  
sumCube(1,3)                                        //35

{% endhighlight %}

这里我们写了一个高阶函数`sum(f:Int=>Int, a:Int,b:Int):Int`，输入是一个函数(Int=>Int)，a，b，输出是Int。相比于上例的`sumCube`和`sumInt`，我们将他们的不同点用函数f抽象出来，剩下的共同点的实现还是一样。因此本例中**一阶函数**`sumInt`和**一阶函数**`sumCube`可以用**高阶函数**`sum`表示，输入分别是block`(x:Int)=>x`和`(x:Int)=>x*x*x`。

这样高阶函数就很好地抽象了低阶函数，使得低阶函数的实现变的简单而不重复。

### 2.2 柯里化 ###

在上面的例子中，我们可以看到高阶函数`sum(f:Int=>Int, a:Int,b:Int):Int`有**三个输入参数**，如何将三个输入函数变成一个输入函数而不影响其实现呢？这个时候，我们不要忘了**函数**的返回也可以是**函数**。我们能否使其输入是f，输出变成`(a:Int,b:Int)=>Int`呢？请看下面代码。

{% highlight scala linenos %}

def sum(f:Int=>Int):(Int,Int)=>Int = {
  def sumF(a:Int,b:Int):Int = {
    if (a>b) 0 else f(a)+sumF(a+1,b)
  }
  sumF
}                                         //sum: (Int => Int)=>(Int, Int) => Int
  
def sumInts = sum((x:Int)=>x)             //sumInts: (Int, Int) => Int
sumInts(1,3)                              //6
sum((x:Int)=>x)(1,3)                      //6
  
def sumCube = sum((x:Int)=> x*x*x)        //sumCube: (Int, Int) => Int
sumCube(1,3)                              //36
sum((x:Int)=> x*x*x)(1,3)                 //36

{% endhighlight %}

由上例可以看出`sum`输入一个函数(类型为(Int=>Int))，返回一个函数(类型为(Int,Int)=>Int)，即输入为单参数，返回的一阶函数输入也是单参数。因此，对于最终的执行有两种形式：

1. `sumCube = sum((x:Int)=> x*x*x)`，用一个函数名`sumCube`获取这个返回函数，然后再调用它`sumCube(1,3)`；
2. 不需要这个中间变量，直接用`sum((x:Int)=> x*x*x)(1,3)`调用。

第二种情况实现了我们上面提到的高阶函数也只需输入1个参数而不影响其最终实现。这样，所有阶级的函数(包括低阶和高阶)都可以完美的统一起来用单参数表示。

>**柯里化**：将**n参数函数**转成**单参数函数**的过程。方法是通过不断转换成一个包含**n个单参数的函数**的嵌套**树形结构**。每一个树形结构的**上层节点**的输入参数对所有**下层节点**的函数**可见**(参数之间先后关系更立体)。

{% highlight scala linenos %}
def sum(f:Int=>Int, a:Int,b:Int):Int = {            //多参数
    if(a>b) 0 else f(a)+sum(f,a+1,b)
}

def sum(f:Int=>Int):(Int,Int)=>Int = {              //单参数
  def sumF(a:Int,b:Int):Int = {
    if (a>b) 0 else f(a)+sumF(a+1,b)
  }
  sumF
} 
//在Scala里，第二种单参数形式可以写成以下形式的语法糖，使单参数调用的形式更直观。
def sum(f:Int=>Int)(a:Int,b:Int): Int = {           //单参数
  if (a>b) 0 else f(a)+sum(f)(a+1,b)
}              

def sumInts = sum((x:Int)=>x)(_,_)                  //中间函数的语法稍微不同，用(_,_)表示第二个单参数没有
sumInts(1,3)                                        //6
sum((x:Int)=>x)(1,3)                                //6

def sumCube = sum((x:Int)=>x*x*x)(_,_)   
sumCube(1,3)                                        //36
sum((x:Int)=> x*x*x)(1,3)                           //36

{% endhighlight %}

对于为什么有Currying的存在，结合[上篇文章]({{ site.baseurl}}/functional%20programming/2015/10/01/Functional-Programming-in-Scala(一)_λ-演算-Part-I-表达式-函数和赋值.html#calculus)对**λ-Calculus**的介绍，我们来整理下思路：

1. λ calculus可以完成所有的计算，它是建立在函数基础上；
2. λ calculus的重要形式之一是函数只能是单参数；
3. 那么既然λ calculus可以完成所有的计算，它如何表达多参数函数呢？
4. 答案是用Currying(多次返回高阶函数)。

这也就是我们为什么要用Currying。

## 3 Assignment ##

这次的作业是用函数式编程完成数据结构**Set(集合)**。`Type Set: Int=>Boolean`，Set被定义成一个输入Int返回Boolean的函数。若包含输入Int，返回true；反之false。(是不是有点脑洞大开的感觉？)

它有一系列操作：

1. `def singleton(elem:Int):Set`，单例的构造方法，输入一个elem，返回一个Set；
2. `def contains(elem:Int, s:Set):Boolean`；
3. `def union(s:Set,p:Set):Set`；
4. ...
5. `def forall(s:Set,p:Int=>Boolean):Boolean`，是否所有的s中的元素都满足p。
6. `def exist(s:Set,p:Int=>Boolean):Boolean`，是否在s中有元素不满足p。 
7. `def map(s:Set,f:Int=>Int):Set`，输入一个set，返回一个经过f变换后的set。

具体代码见[这里](https://github.com/shunmian/-2_Functional-Programming-in-Scala)。

`map`函数需要遍历s，由于Set的定义，不能直接完成这个操作，因此作业中设定了我们要处理的数字的上下界。
这要借助`exist`函数，而`exist`函数本身又建立在`forall`函数上。这真是有种**虚无生两极，两极生四象，四象生八卦**的感觉，**上帝**创造了**Alonzo Church**，**Alanzo Church**创造了**λ-Calculus**，**λ-Calculus**催生了现在的**函数式编程**，**函数式编程**又用**函数**实现了数据结构**Set**，**Set**中的复杂函数又建立在一个个基础函数上。任何事物都有一个源头，顺着时间的长河回溯，上帝之前又是什么呢？




## 4 总结 ##


Currying是伴随高阶函数而存在的。最后，我们将Currying和高阶函数总结成一句话和一幅图：

>高阶函数是纯函数式编程的核心。<br/>
Currying(即嵌套高阶函数)：纯函数式编程中对多参数函数的实现。

{: .img_middle_mid}
![Currying](/assets/images/posts/2015-10-02/Currying.png)

## 5 参考资料 ##
- [《Structure and Interpretation of Computer Programs》](https://mitpress.mit.edu/sicp/full-text/book/book.html);
- [Martin Odersky: Scala with Style](https://www.youtube.com/watch?v=kkTFx3-duc8);
- [SF Scala: Martin Odersky, Scala -- the Simple Parts](https://www.youtube.com/watch?v=ecekSCX3B4Q);
- [Programming Languages: Lambda Calculus](https://www.youtube.com/watch?v=v1IlyzxP6Sg);
- [Functional Programming For The Rest of Us](http://www.defmacro.org/ramblings/fp.html);



