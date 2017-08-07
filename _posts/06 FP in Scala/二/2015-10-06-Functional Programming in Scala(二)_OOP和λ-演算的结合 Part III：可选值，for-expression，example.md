---
layout: post
title: FP in Scala(二)：OOP和λ-演算的结合 Part III：可选值，for-exp
categories: [06 Functional Program]
tags: [OOP&&FP]
number: [-2.2]
fullview: false
shortinfo: List是函数式Data Structure的一个典型，结合函数式的pattern matching和递归，我们可以实现许多一阶和高阶的methods。这些methods高度抽象了list需要做什么，而不是怎么做。这也正是函数式编程和命令式编程的区别的一个集中体现。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. 介绍 ##


## 2. collection ##

### 2.1 可选值 ### 

>**Option(可选值)**：有两个子类，一个是None，一个是Some。这是Scala表示有和无的一种处理。我们再回顾下上文中`Map`的`get`方法。

{% highlight scala linenos %}
  val capitalOfCountry = Map("US"->"Washington","Switzerland"->"Bern")
                                                  //Map(US -> Washington, Switzerland -> Bern)
  capitalOfCountry("US")                          //String = Washington
  capitalOfCountry.get("China")                   //Option[String] = None
  capitalOfCountry.get("US")                      //Option[String] = Some(Washington)

{% endhighlight %}

有和无的处理一直是programming中的一个无法避免的问题，有些时候，函数返回非正常值来代替无，比如`List`的方法`indexOf(x)`返回下标若有x，返回-1若没有x。这会带来一些额外的问题，比如编程者需要把结果和-1判断才能知道是不是有x，这就提前假定了编程者知道-1在`List`类里的含义。如果我们用`Option`就不需要有这种担忧，直接和`None`比较即可。`Swift`中对于`Option`有大量的应用，只要随便翻开`Swift`的代码，里面一定充满了`?`和`!`，这就是对于`Option`的拆包的操作。

### 2.2 for-expression ### 

Scala的函数式也有自己的for循环的语法，相比较于命令式的for循环，scala的for循环更简洁，而且是基于对数据的转换而不是更改。下面举一个栗子。

>**质数问题**：给定一个自然数n，求所有的组合(i,j)，满足i+j为质数，且1<=j<i<N。

我们用之前的高阶函数的实现如下：

{% highlight scala linenos %}

  def isPrime(n:Int):Boolean = (2 until n).forall(x=>n%x!= 0)
  
  def primeCombs1(n:Int)= ((1 until n).flatMap(i=>(1 until i).map(j=>(i,j)))).filter(pair=>isPrime(pair._1 + pair._2))

{% endhighlight %}

这种语法看起来不是非常容易理解。那么如果我们用Scala的for循环语句的实现是什么样子呢？

{% highlight scala linenos %}

  def isPrime(n:Int):Boolean = (2 until n).forall(x=>n%x!= 0)
  
  def primeCombs2(n:Int) =
  	for{
  		i<- 1 until n      //generator，遍历范围
  		j<- 1 until i
  		if isPrime(i+j)    //filter，遍历过滤
  	} yield(i,j)  		   //yield，遍历业务逻辑。

{% endhighlight %}

for语句使得我们的代码看起来更简洁易懂。

#### 2.2.1 for-expression语法 #### 

`for-expression`的语法有三部分：

1. `generator`，也就是for循环的单个元素。“左边<-右边” 表示左边是右边(Collection)的元素。`genertor`后面可以再跟`generator`表示嵌套；
2. `filter`跟在`generator`后面，用`if`表示；
3. `yield`表示对于`for{}`里的每个符合filter的来自generator的元素，我们的处理。这里是生成一个`tuple(i,j)`。最后的结果这里是`IndexedSeq[(Int,Int)]`。我们可以将结果`toList`转成List。

总结起来`generator`和`filter`分别是遍历范围和条件，而`yield`是遍历业务逻辑。

#### 2.2.2 for-expression as Query for Database #### 
由于Scala **for-expression**的简洁强大，它可以用于**数据库查询**。我们看下面这个例子。

{% highlight scala linenos %}

  case class Book(title:String, authors:List[String])
  
  val books:List[Book] = List(new Book(title = "Structure and Interpretation of Computer Programs", authors = List("Abelson, Harald", "Sussman, Gerald J.")),
                	 new Book(title = "Introduction to Functional Programming", authors = List("Bird, Richard", "Wadler, Phil")),
                	 new Book(title = "Effective Java", authors = List("Bloch, Joshua")),
                	 new Book(title = "Effective Java2", authors = List("Bloch, Joshua")),
                	 new Book(title = "Java Puzzlers", authors = List("Bloch, Joshua", "Gafter, Neal")),
                	 new Book(title = "Programming in Scala", authors = List("Odersky, Martin","Spoon, Lex", "Venners, Bill"))
  )

  for(b<-books; a<-b.authors if a.startsWith("Bloch")) yield b.title
                                                  //List(Effective Java, Effective Java2, Java Puzzlers)
  
  (for{b1<-books
  		b2<-books
  		if b1.title < b2.title
  		a1<-b1.authors
  		a2<-b2.authors
  		if a1==a2
  		} yield a1
 	).distinct                                //or toSet，结果是List(Bloch, Joshua)
 

{% endhighlight %}

第一个查询是列出books中作者名字以"Block"开头的书本名字；第二个查询稍微复杂点，是列出books中写出两本以上书的作者的名字，`.distinct`或者`toSet`是将结果中重复的剔除掉。

我们可以看到Scala的**for-expression**用于数据库查询也十分紧凑明了。数据库查询框架**ScalaQuery**，**Slick**和**Microsoft's LINQ**都是基于Scala的**for-expression**来实现的。

Scala的**for-expression**的实现是基于**filter**，**map**和**flatMao**这三种高阶函数。本质上来说**for-expression**是语法糖，让开发者代码写起来简单易懂又不失其强大。




### 2.3 例子 ###

我们将所学的Collection数据结构以及可选值和for表达式结合在一起，来解决一个具体问题。

>**号码字母问题**：给定一个数字，返回所有可能的单词。数字和字母的对应关系如下图所示。即"2"->"ABC"，...,"9"->"wxyz"。如果我们输入是"7225247386"，其中一个输出是"scala is fun"。

{: .img_middle_mid}
![Nokia N70](/assets/images/posts/2015-10-06/Nokia N70.png)

我们用**top-down**一步步思考该如何实现：


1. 首先，我们有一个`encode(number:String):Set[String]`，输入一个数字，返回最后答案的`Set`。因为答案不能有重复，所以`Set`比较合适。这个方法的实现主要用到**分而治之**的思想：

	1.1 将数字拆解成两部分，第一部分求解；然后第二部分求解；<br/>
	1.2 将结果merge。

2. 求解过程中有一个函数`numberToWords(number:String):Set[String]`，即给定了一个数字(不可拆分)，输出所有可能的单词。那么如何实现呢，我们设想有一个`val numberToWordsMap: Map[String，List[String]]`，任何数字的对应单词都已经存在里面了。那么我们只需要调用这个`Map`即可。

3. `numberToWordsMap`需要我们将已有的单词字典里的每个单词都写成数字`def encodeWord(word:String):Int`，然后按照数字归类。

4. `def encodeWord(word:String):Int`需要实现将字母写成单个数字`def encodeChar(char:Char): Int`。

5. `def encodeChar(char:Char): Int` 需要将上图Nokia键盘的数字转换字母规则转换成字母转换数字规则。

6. 课程提供了一个简易字典文件，最后我们将上述所有连接起来，就是最终实现了。见下面代码。




{% highlight scala linenos %}

import scala.io.Source
import scala.collection.immutable.List
object lecture7 {

  import scala.io.Source
import scala.collection.immutable.List
object lecture7 {
          
  val in = Source.fromFile("/Users/LAL/Scala/hello-project/src/week6/dict/linux.words")
                                                
  val words = in.getLines.toVector.filter(word=>word.forall(lt=>lt.isLetter))
                                                
  val numbToLettsMap:Map[Char,String]= Map('2'->"ABC",'3'->"DEF",'4'->"GHI",'5'->"JKL",
  								'6'->"MNO",'7'->"PQRS",'8'->"TUV",'9'->"WXYZ")
                                                  //> numbToLettsMap  : Map[Char,String] = Map(8 -> TUV, 4 -> GHI, 9 -> WXYZ, 5 -
                                                  //| > JKL, 6 -> MNO, 2 -> ABC, 7 -> PQRS, 3 -> DEF)
  /**Invert the mnem map to give a map from chars 'A'...'Z' to '2'...'9'*/
  val charCode:Map[Char,Char] = for ((numb,letts)<-numbToLettsMap;lett<-letts) yield lett->numb
                                                  //> charCode  : Map[Char,Char] = Map(E -> 3, X -> 9, N -> 6, T -> 8, Y -> 9, J 
                                                  //| -> 5, U -> 8, F -> 3, A -> 2, M -> 6, I -> 4, G -> 4, V -> 8, Q -> 7, L -> 
                                                  //| 5, B -> 2, P -> 7, C -> 2, H -> 4, W -> 9, K -> 5, R -> 7, O -> 6, D -> 3, 
                                                  //| Z -> 9, S -> 7)
  /**Maps a word to the digit string it can represent, e.g. "Java"->"5282"*/
  def wordCode(word:String):String = word.toUpperCase.map(charCode)
                                                  //> wordCode: (word: String)String
  /**
  *A map from digit strings to the words that represent them
  *e.g. "5282" -> List("Java","Kata","Lava",...)
  *Note: A missing nuber should map tot he empty set,e.g. "1111" -> List()
  */
  def numbToWordMap:Map[String,Vector[String]] = words.groupBy(wordCode).withDefaultValue(Vector())
                                                  //> numbToWordMap: => Map[String,Vector[String]]
  /**Return all ways to encode a number as a list of words*/
  def encode(numb: String):Set[List[String]] =
  if (numb.isEmpty) Set(List())
  else{
  	for{
  		part<-1 to numb.length
  		result<-numbToWordMap(numb.take(part))
  		rest<-encode(numb.drop(part))
  	} yield result::rest
  }.toSet                                         //> encode: (numb: String)Set[List[String]]

  println(encode("7225247386"))                   //> Set(List(rack, ah, re, to), List(sack, ah, re, to), List(Scala, ire, to), L
                                                  //| ist(sack, air, fun), List(rack, air, fun), List(rack, bird, to), List(pack,
                                                  //|  air, fun), List(pack, ah, re, to), List(pack, bird, to), List(Scala, is, f
                                                  //| un), List(sack, bird, to))
  def transform(numb: String):Set[String] = encode(numb).map(_.mkString(" "))
                                                  //> transform: (numb: String)Set[String]
  println(transform("7225247386"))                //> Set(sack air fun, pack ah re to, pack bird to, Scala ire to, Scala is fun, 
                                                  //| rack ah re to, pack air fun, sack bird to, rack bird to, sack ah re to, rac
                                                  //| k air fun)
}

{% endhighlight %}

关于这个例子，有一篇文章[An empirical comparison of C, C++, Java, Perl, Python, Rexx, and Tcl for a search/string-processing program](http://page.mi.fu-berlin.de/prechelt/Biblio/jccpprtTR.pdf)专门比较了7种语言实现的不同。其中：

1. 脚本语言用了大约100loc(line of code)；
2. 非脚本语言用了大约200-300loc。

对比一下scala,**20loc**以内解决问题，而且不用担心遍历Collection时的边界条件，实在令我们印象深刻。**Scala**函数式语言的强大可见一斑，这主要依赖基于函数式数据结构实现的Collection的强大且简捷的高阶函数。

## 3 Assignment ##

本周的作业是写Sentence的Anagram，其中`foldLeft`的紧凑在这里得到了良好的体现，本来需要好十几行的代码，用`foldLeft`几行就能解决，其中巧妙，需要细细体会。`foldLeft`其实是对`monoids`的一种应用。`monoids`将在后续文章中讨论。

{% highlight scala linenos %}

  /**
   * Returns the list of all subsets of the occurrence list.
   *  This includes the occurrence itself, i.e. `List(('k', 1), ('o', 1))`
   *  is a subset of `List(('k', 1), ('o', 1))`.
   *  It also include the empty subset `List()`.
   *
   *  Example: the subsets of the occurrence list `List(('a', 2), ('b', 2))` are:
   *
   *    List(
   *      List(),
   *      List(('a', 1)),
   *      List(('a', 2)),
   *      List(('b', 1)),
   *      List(('a', 1), ('b', 1)),
   *      List(('a', 2), ('b', 1)),
   *      List(('b', 2)),
   *      List(('a', 1), ('b', 2)),
   *      List(('a', 2), ('b', 2))
   *    )
   *
   *  Note that the order of the occurrence list subsets does not matter -- the subsets
   *  in the example above could have been displayed in some other order.
   */
      
  def combinations(occurrences: Occurrences): List[Occurrences] = {
    occurrences.foldLeft(List[Occurrences](Nil))((acc, item) => {
      for {
        oc <- acc //oc: List[(Char,Int)]
        x <- (for (i <- 0 to item._2) yield (item._1, i)).toList //x: (Char,Int)
      } yield if (x._2 == 0) oc else oc ++ List(x)
    })
  }

{% endhighlight %}

具体代码见[这里](https://github.com/shunmian/-2_Functional-Programming-in-Scala)。


## 4 总结 ##




{: .img_middle_lg}
![for-expression && Option](/assets/images/posts/2015-10-06/for-expression && Option.png)

## 5 参考资料 ##
- [《Structure and Interpretation of Computer Programs》](https://mitpress.mit.edu/sicp/full-text/book/book.html);
- [Martin Odersky: Scala with Style](https://www.youtube.com/watch?v=kkTFx3-duc8);
- [SF Scala: Martin Odersky, Scala -- the Simple Parts](https://www.youtube.com/watch?v=ecekSCX3B4Q);
- [Programming Languages: Lambda Calculus](https://www.youtube.com/watch?v=v1IlyzxP6Sg);
- [Functional Programming For The Rest of Us](http://www.defmacro.org/ramblings/fp.html);



