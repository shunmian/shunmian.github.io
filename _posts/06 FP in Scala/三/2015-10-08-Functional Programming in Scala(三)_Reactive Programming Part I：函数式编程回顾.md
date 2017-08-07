---
layout: post
title: FP in Scala(三)：Reactive Programming Part I：函数式编程回顾
categories: [06 Functional Program]
tags: [OOP&&FP]
number: [-2.2]
fullview: false
shortinfo: 本文将对函数式编程做一个总结，回顾函数式数据类型(JSON实例)的实现(基于Pattern matching)，for语法本质，如何子类化函数(既然函数和数据一样，当然可以继承)，Collection Hierachy，以及如何实现随机数产生器(既然函数式编程是只和输入有关，如何才能产生随机数呢，即调用同一个函数，每次产生的结果不同)，最后介绍一下函数式编程的重要概念Monad(如何将数据上下文，包括failure(Maybe)，不确定性([])，IO等引入函数式编程)。本文内容属于响应式编程的一个基础，需要同学们牢牢掌握。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. 介绍 ##


## 2. 纯函数式编程回顾 ##

### 2.1 函数式数据类型(Pattern Matching) ###

函数式数据类型的实现来源于函数式函数的实现，而后者的实现就是基于**Pattern Matching**的。**Pattern Matching**其实是对函数**分流**本质的体现，特别是在输入Pattern对应不同的输出表达式的函数，比如递归在退出条件里和在一般条件里的输出表达式是不一样的。而**Pattern Mathing**简化了这一**分流**的实现。关于**Pattern Match vs Decomposition** 用来扩展类的方法的好处，我们在[这篇文章]({{ site.baseurl}}/functional%20programming/2015/10/04/Functional-Programming-in-Scala(二)_OOP和λ-演算的结合-Part-I-万物皆类-&&-函数式的体现.html#pattern-matching)中讨论过。如果对于子类种类固定的父类来扩展方法，**Pattern Matching** 更方便；如果对于子类种类个数会增加的父类来扩展方法，则用Decompostion更灵活。下面是**Scala**用**Pattern Matching**对**JSON**的一种实现。

{% highlight scala linenos %}

object MyJSON extends App {

  abstract class JSON

  case class JObj(bindings: Map[String, JSON]) extends JSON
  case class JSeq(elems: List[JSON]) extends JSON
  case class JNum(num: Double) extends JSON
  case class JBool(bool: Boolean) extends JSON
  case class JStr(str: String) extends JSON
  case object JNull extends JSON

  def show(json: JSON): String = json match {
    case JObj(bindings) =>
      val assocs = bindings map {
        case (key, value) => "[ " + "\"" + key +"\"" + ": " + show(value) + " ]"  
      }
     "{ " + (assocs mkString ",\n") +  " }"

    case JSeq(elems) => "\n" + "[" + (elems map show mkString ",") + "]"

    case JNum(num)   => num.toString()
    case JBool(bool) => bool.toString()
    case JStr(str)   => "\"" + str + "\""
    case JNull       => "null"
  }

  override def main(args: Array[String]) {

    val data = JObj(Map(
      "firstName" -> JStr("Johnson"),
      "lastName" -> JStr("Smith"),
      "address" -> JObj(Map(
        "streetAddress" -> JStr("21 2nd Street"),
        "state" -> JStr("NY"),
        "postalCode" -> JNum(10021))),
      "phoneNumbers" -> JSeq(List(
        JObj(Map(
          "type" -> JStr("home"),
          "number" -> JStr("212 555-1234"))),
        JObj(Map(
          "type" -> JStr("fax"),
          "number" -> JStr("646 555-4567")))))))
    println(show(data))
    
  }
}
{% endhighlight %}

程序本身并不困难，但其中有一些细节需要我们去解释。第15行代码的type是什么呢？显然这是高阶函数map的一个输入，也就是函数，因此有**"_=>_"**的形式。

{% highlight scala linenos %}
 { case (key, value) => "[ " + "\"" + key +"\"" + ": " + show(value) + " ]" } 
{% endhighlight %}

这行代码表示的是如果输入是一个pair的形式，则返回如下String，因此它的type是`(String，JSON) => String`.

这是一个匿名函数，在Scala里匿名函数和函数一样，都是一个实现了`def apply(args1:type1,...argsn:typen):typem`方法的类`Functionn+1(type1,...typen,typem)`。因此第15行代码将会被转换成如下代码：

{% highlight scala linenos %}

new Function1[(String, JSON), String]{
  def apply(x:(String,JSON)) =  x match {
    case (key, value) =>  "[ " + "\"" + key +"\"" + ": " + show(value) + " ]"
  }
}

{% endhighlight %}

可以看到匿名函数就像java里new出来的一个data实例，只不过没有变量来point指向这个函数。

因此函数式数据结构的实现在JSON这里例子里得到了很好的体现。

### 2.2 子类化函数 ###

那么既然函数就是类，那么可否子类化函数呢？答案是可以的。

我们知道**Map**和**函数**本质上是一样的，**Map**对于给定的**key**，返回一个**value**，**函数**也是一样。在**命令式编程**里，数据是一等公民，而函数不是，**Map**是数据，因此**函数**也是**数据**建立在data比function优先的基础上。这里我们要讲的是另外一面，在**函数式编程**里，函数提升到一等公民(数据当然也是)，甚至可以认为比数据更优先。因此**Map**，即数据，是可以通过继承**函数**来实现的。这就是我们所说的**子类化函数**。

{% highlight scala linenos %}

trait Map[Key, Value] extends (Key => Value)...
trait Seq[Elem] extends Int => Elem ...
{% endhighlight %}

除了上述的数据类型`Map`继承`(Key=>Value)`，`Seq`继承`Int=>Elem`。还有一个特殊的**函数**继承函数，那就是**Partial Function**。

{% highlight scala linenos %}

  val pf: PartialFunction[String, String] = { case "Ping" => "Pong" }

  pf.isDefinedAt("Ping")  //return true
  pf.isDefinedAt("Hello") //return false
  pf("Ping")              //return Pong
  pf("hello")             //exception

  if (pf.isDefinedAt("Hello") pf("hello") //no exception
  
  trait PartialFunction[-A,+R] extends Function1[-A,+R]{
    def apply(x:A):R
    def isDefinedAt(x:A):Boolean
  }

{% endhighlight %}

**Partial Function**继承了函数，同时声明了`def isDefinedAt(x:A):Boolean`。因此为了避免调用函数时抛出未定义的异常，我们先用`isDefinedAt`判断再取值。

### 2.3 Collection Hierachy ###

在Scala里，Collection作为**Algebraic Data Type(ADT)**，它的Class Hierachy 如下图所示。

{: .img_middle_hg}
![collection hierachy](/assets/images/posts/2015-10-08/collection hierachy.png)

这些Collection子类都有一些重要的高阶函数，如`map`，`flatMap`，`filter`和`foldLeft`，`foldRight`等。List对其中一些方法的实现如下。

{% highlight scala linenos %}
def map[U](f:T=>U):List[U] = this match {
  case x::xs => f(x):: xs.map(f)
  case Nil => Nil
}
  
def flatMap[U](f: T=>List[U]):List[U] this match {
  case x::xs => f(x) ++ xs.map(f)
  case Nil => Nil
}
  
def filter(f: T=>Boolean):[T] this match {
  case x::xs => if(f(x)) x :: xs.filter(f) else xs.filter(f)
  case Nil => Nil
}

{% endhighlight %}

这些高阶函数其实是**Category Thoery**里对于**Monoid(foldRight,foldLeft)**，**Functor(map)**，**Monad(flatMap)**的实现。这些我们在第三部分讨论。

### 2.4 for表达式本质 ###

for表达式其实是对`filter`，`flatMap`，`map`应用的一种语法糖。

{% highlight scala linenos %}

(1 until n) flatMap (i=>
   (1 until i) filter (j => 
        isPrime(i+j)) map (j=>
           (i,j))
//简化成
for{
  i <- 1 until n
  j <- 1 until i
  if isPrime(i+j)
} yield (i,j)

{% endhighlight %}

需要注意的是`<-`的左边也可以是pattern，比如

{% highlight scala linenos %}
for {
  JObj(bindings) <- data
  JSeq(phones)    = bindings("phoneNumbers")
  JObj(phone)    <- phones
  JStr(digits)    = phone("number")
  if digits startsWith "212"
} yield (bindings("firstName"), bindings("lastName"))

{% endhighlight %}

这里是找到所有电话号码以212开头的JObj，然后将firstName和lastName返回。

### 2.5 随机数产生器 ###





## 3. 范畴轮(Category Thoery) ##

### 3.1 幺半群(Monoid) ###

### 3.2 函子(Functor) ###

### 3.3 应用函子(Applicative Functor) ###

### 3.4 单子(Monad) ###

 
{% highlight scala linenos %}


{% endhighlight %}




## 4. 响应式编程(Reactive Programming) ##


4. 还有一点需要注意的是性能问题，对于之前探索过的`Set[State]`，`Path`extend后要过滤掉回到之前`State`的Path。

{: .img_middle_mid}
![water pouring](/assets/images/posts/2015-10-07/water pouring.png)



## 5. assignment ##

## 6. 总结 ##





## 7 参考资料 ##
- [《Structure and Interpretation of Computer Programs》](https://mitpress.mit.edu/sicp/full-text/book/book.html);
- [Martin Odersky: Scala with Style](https://www.youtube.com/watch?v=kkTFx3-duc8);
- [SF Scala: Martin Odersky, Scala -- the Simple Parts](https://www.youtube.com/watch?v=ecekSCX3B4Q);
- [Programming Languages: Lambda Calculus](https://www.youtube.com/watch?v=v1IlyzxP6Sg);
- [Functional Programming For The Rest of Us](http://www.defmacro.org/ramblings/fp.html);



