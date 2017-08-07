---
layout: post
title: FP in Scala(二)：OOP和λ-演算的结合 Part I：万物皆类 && 函数式的体现
categories: [06 Functional Program]
tags: [OOP&&FP]
number: [-2.2]
fullview: false
shortinfo: 什么是λ-Calculus，函数式编程的理论基础是什么，什么是高阶函数, 为什么有Currying, 函数式编程为什么需要单参数函数？本文将为您抽丝剥茧，一一梳理。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. OOP ##




{: .img_middle_mid}
![compound data](/assets/images/posts/2015-10-03/compound data.png)



## 2. 万物皆类 ##


### 2.1 Primitive Data is Class ###

如果说让你定义一个布尔基础类**Boolean**，你会怎么做呢？Scala里的实现如下，先上代码：

{% highlight scala linenos %}
package week4

object Main{
   
 abstract class Boolean{
   
    def ifThenElse[T](e1: => T, e2: => T): T
    
    def && (b2: => Boolean): Boolean = ifThenElse(b2,False)
    def || (b2: => Boolean): Boolean = ifThenElse(True,b2)
    def unary_! : Boolean            = ifThenElse(False,True)
    
    def == (b2: Boolean): Boolean = ifThenElse(b2, b2.unary_!)
    def != (b2: Boolean): Boolean = ifThenElse(b2.unary_!, b2)
  }
  
  object True extends Boolean{
    def ifThenElse[T](e1: => T, e2: => T): T = e1
  }
  
  object False extends Boolean{
    def ifThenElse[T](e1: => T, e2: => T): T = e2
  }
  
   def main(args: Array[String]) {
     
    def p(b:Boolean) = b.ifThenElse(println("True"),println("False")) 
     
    p(True)               //True
    p(False)              //False
    
    p(True && False)      //False
    p(True && True)       //True
    p(False && True)      //False
    p(False && False)     //False
    
    p(True || False)      //True
    p(True || True)       //True
    p(False || True)      //True
    p(False || False)     //False
    
    p(!True)              //False
    p(!False)             //True

    p(True == False)      //False
    p(True == True)       //True
    p(False == False)     //True
    p(False == True)      //Flase
    
    p(True != False)      //True
    p(True != True)       //False
    p(False != False)     //False
    p(False != True)      //True
  }
}
{% endhighlight %}

课程里只是给了如上一个完成了的Boolean的实现，令人感叹于它的优雅。可是如果是让你重头开始自己设计Boolean，怎样才能获得这样的结果呢？对于编程学习者，我们渴求的不是Boolean的实现，而是Boolean实现的思路；正如物理学家渴求的不是相对论，而是发现相对论的那个头脑。我们来试着思考下：

1. 首先，`Boolean`有两个子类`True`和`False`，这是毋庸置疑的；
2. 这两个子类实现的最基本的功能应该就是单元操作，而不是多元操作；
3. 那么我们先看看单元操作，有取反`!`，和条件判断 `if (b:Boolean) e1 else e2`；
4. 我们暂且将这个函数定义为`def ifThenElse[T](e1: => T, e2: => T): T`，即假定Scala已经自动实现了上述将`if (b:Boolean) e1 else e2`转换成`b.ifThenElse(e1,e2)`；
5. 那么两个子类`True`和`False`对该函数的实现分别为返回`e1`和`e2`也就理所应当了；
6. 在这个基础上，我们尝试用`ifThenElse[T](e1: => T, e2: => T): T`实现取反的单元操作和其他二元操作，结果证明是可行的。

我觉得上述思路应该是一个正常的思路，如果一开始就尝试用一个general的公式取实现Boolean所有的单元和多元操作，比较困难。我们留一个小问题，请同学自己实现Boolean的 **< method**.


最后Boolean的实现用以下这张图表示。

{: .img_middle_mid}
![Boolean](/assets/images/posts/2015-10-04/Boolean.png)



### 2.2 Compound Data is Class ###

这个同学们应该都很熟悉了，就是最常见的类的对data和function的封装(当function被封装在类里，我们称之为method，具体区别见[2.4]({{ site.baseurl}}/functional%20programming/2015/10/04/Functional-Programming-in-Scala(二)_OOP和λ-演算的结合.html#compound-procedure-is-class)。我们重新改写[λ-演算 Part III：抽象数据]({{ site.baseurl}}/functional%20programming/2015/10/03/Functional-Programming-in-Scala(一)_λ-演算-Part-III-抽象数据.html)的有理数的例子，将**Compound Data**改写成**类**，只需将函数封装到数据里面即可。

{% highlight scala linenos %}
object Main {

  class Rational(x: Int, y: Int) {
    require(y != 0, "demoninator must be non-zero")
    val numer = x / gcd(x, y)
    val denomi = y / gcd(x, y)
    private def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)

    // compared to the previous compound data example, here function are defined inside Rational
    override def toString() = this.numer + "/" + this.denomi

    def makeRationl(x: Int) = new Rational(x, 1)
    def neg(r: Rational) = new Rational(-r.numer, r.denomi)

    def add(r2: Rational): Rational = {
      new Rational(this.numer * r2.denomi + this.denomi * r2.numer, this.denomi * r2.denomi)
    }

    def sub(r2: Rational): Rational = add(neg(r2))

    def less(r2: Rational): Boolean = this.numer * r2.denomi - this.denomi * r2.numer < 0
    def max(r2: Rational) = if (this.less(r2)) r2 else this

  }

  def main(args: Array[String]) {

    val r1: Rational = new Rational(4, 5)
    val r2: Rational = new Rational(3, 4)
    
    val r3 = r1.add(r2)
    println(r3)

    val r4 = r1.sub(r2)
    println(r4)

    val r5 = r1.max(r2)
    println(r5)
  }
}
{% endhighlight %}

### 2.3 Primitive Procedure is Class ###

那么如何将诸如+,-,*,\的**Primitive Procedure**表示成类呢，其实我们在[2.1]({{ site.baseurl}}/functional%20programming/2015/10/04/Functional-Programming-in-Scala(二)_OOP和λ-演算的结合.html#compound-procedure-is-class)已经看到了如何将。。------------------------------------------





### 2.4 Compound Procedure is Class ###

**Compound Procedure**即**函数**，在scala里，函数如何才能使一个类呢？

>**function variable** are **class**，and **function value** are **object**。<br/>
**function value** is from instantiation of function literals(anonymous functions)。

如果一个函数的类型是 `A=>B`，那么在scala里，这会转成`scala.Function1[A,B]`，其中A和B是泛型，1表示输入参数是1个，Function1[A,B]表示1个输入类型A和1个输出类型B的函数类。那么这个类的定义是什么呢？是一个如下的接口，

{% highlight scala linenos %}
trait Function1[A,B]{
  def apply(x:A):B
}

//变量f1是Int=>Int的函数，函数体是(x:Int)=>x*x；这个定义的完整版如下f2(f1,和f2等价)
val f1: Int=>Int = (x:Int)=>x*x

//变量f2，实例化一个Function1[A,B]的子类Function1[Int,Int],其中的apply method的实现就是f1中的函数体。
val f2 = new Function1[Int,Int]{
  def apply(x:Int)=x*x
}

//f1调用
f1(3)			//9

//f1会被转换成
f1.apply(3)		//9


{% endhighlight %}

上面f2实例化一个Function1[A,B]的子类Function1[Int,Int],其中的apply method的实现就是f1中的函数体。

也许有人会问，apply 会再实例化一个函数类吗，如果会，不就陷入死循环了吗？

这里我们要着重讲解两个概念**Function vs Method**。

>A **function** in Scala is a complete object，which **contains methods**， One of these methods is the apply method， which contains the code that implements the body of the function。 A method(an OOP concept) is defined within class and without class, there is no method。

在了解function和method的关系下，有几个method和function相互替代的地方需要注意：

<ol>

<li> method 只能是def，function可以是val(即一个变量指向一个函数object)。</li>
<li> method可以被隐式转换成function，如：

{% highlight scala linenos %}

def m(x:Int) = x*x
val f  = m    			  //error
val f1 = m _  			  //return a new Function1[Int,Int]{def apply(x:Int)=x*x}
val f2 = m(_) 			  //the same as f1
val f3 = new Function1[Int,Int]{	  //f1,f2 are actually converted as f3
  def apply(x:Int)= m(x)
}

{% endhighlight %}

</li>


<li> function通常比实现同样功能的method占用更多内存，但是function可以有OOP带来的其它好处，比如Scala默认实现了toString()。所以有些时候用function用内存换来的性能是值得的。

{% highlight scala linenos %}

def m(x:Int) = x*x
m.toString()	//error
f1.toString()	//res5: String = &lt;function1>&gt;

{% endhighlight %}

</li>

<li>def evaluates every time it gets called while val evaluates only once。

{% highlight scala linenos %}

def isDivisibleBy(k: Int): Int => Boolean = i => i % k == 0
def isEven1 = isDivisibleBy(2)		
isEven1(2)(5)
isEven1(2)(5)			// isDivisibleBy(k: Int): Int => Boolean is called every time

val isEven2 = isDivisibleBy(2)	
isEven2(2)(5)
isEven2(2)(5)			// isDivisibleBy(k: Int): Int => Boolean is called only once during val isEven2's definition.


{% endhighlight %}

</li>


</ol>



{% highlight scala linenos %}

{% endhighlight %}




## 3. Scala中的类 ##

既然**Scala**中万物皆类，我们来看下类需要特别注意的地方，包括语法和类等级(Class Hierachy)。

### 3.1 类的关键词 ###

Scala里类的语法还有以下关键词。

<ol>

<li><b>this</b>，和Java里的this一样，在类里指自己这个实例。但是Scala里的<b>this</b>还可以是一个构造器。

{% highlight scala linenos %}

class Rational(x: Int, y: Int) {		//这是一个默认构造器
...
  def this(x: Int) = this(x,1) 		//如果只输入一个参数x，那么自动生成分子为x，分母为1的Rational实例
...
}

{% endhighlight %}
</li>

<li><b>require</b>，used to enforce a precondition on the caller of a function。

{% highlight scala linenos %}

class Rational(x: Int, y: Int) {
  require(y != 0, "demoninator must be non-zero")	//用来规定一个precondition，当对Rational构造器调用时。
...
}

{% endhighlight %}
</li>

<li><b>assert</b>，used to check the code of the function itself。

{% highlight scala linenos %}

class Rational(x: Int, y: Int) {	
...
  assert(y != 0)		// up to here，check whether y!=0
...
}

{% endhighlight %}

<b>assert</b>和<b>assert</b>的区别在他们各自的定义里基本说清楚了，需要补充的是<b>assert</b>其实更多的是类的implementation的时候的先决条件，而<b>assert</b>更多的是一种对程序当前状态是否已达预期的一种判断。

</li>

<li><b>trait</b>，类似Java里的interface，但是比它强大，因为<b>trait</b>可以有<b>method</b>的具体实现和field(但是不能有value)。<b>Swift的protocal</b>和<b>Scala的trait</b>非常相似。

{% highlight scala linenos %}

trait Movable{	
...
  val speed:Int；			//field cannot have value
  def trace(x:Int) = x*x 	//method can have implementation，can be override
...
}

{% endhighlight %}
</li>
</ol>




### 3.2 多态和泛型 ###

<blockquote><b>Polymorphism(多态)</b>，在OOP中有两种形式存在于类中：
<ol>
<li> 类的<b>subTyping</b>，即当需要一个父类的时候，子类可以代替父类；</li>
<li> 类的<b>generic</b>，即定义类和定义类中的method的时候，input argument type can be parameterized。</li>
</ol>
</blockquote> 


### 3.3 类等级 ###

{: .img_middle_lg}
![scala class hierarchy](/assets/images/posts/2015-10-04/scala class hierarchy.png)

在Scala里，类等级(Class Hierachy)如上图所示：

1. 所有类的祖先是`Any`。`Any`又包括`AnyVal`和`AnyRef`，即分别是**Primitive Data**和**Compound Data**；

2. `AnyRef`里的底部的子类是`Null`，`AnyVal`里的底部的子类是`Nothing`。同时`Nothing`又是`Null`的子类。

3. 关于`AnyRef`里的`Iterable`，`Seq`，`List`等接口我们暂且不表。


### 3.4 运行时和dynamic method dispatch ###

这里暂且不表，具体请先参见[Objective-C的runtime系列]({{ site.baseurl}}/objective-c/2016/03/14/OC-Runtime(一)_Sending-Message.html)。


## 4. 函数式特性的体现:Pattern Matching ##

**pattern match**是Scala为了避免为同一类添加一个新方法导致大量修改子类。由于**OOP和λ-Calculus**结合的函数式(递归)特性催生了这一**pattern match**模式。下面我们以一个算术表达式为例子(有Number，表示数字；Sum，有左Operand和右Operand)，逐步理解**pattern match**的来历，优点及不足。

#### 4.1 OOP Decomposition ####

>**Object-oriented decomposition**： breaks a large system down into progressively smaller classes or objects that are responsible for some part of the problem domain。

我们将`Number`和`Sum`同时抽象成`Expr`接口。`Expr`接口里，我们分别实现了classification和accessor为了同时满足Number和Sum的操作(其实有冗余，`Sum`不需要`numberValue`，同时`Number`不需要`leftOp`，`rightOp`)。我们让`Number`和`Sum`同时实现`Expr`接口(即实现这5个method)。最后我们定义了一个`eval(e: Expr): Int` method 来实现最终的赋值。

{% highlight scala linenos %}

trait Expr {
  //classification
  def isNumber: Boolean          
  def isSum: Boolean
  
  //accessor
  def numValue: Int
  def leftOp:Expr
  def rightOp:Expr
}

class Number(n:Int) extends Expr{    //the Number subclass
  
  def isNumber: Boolean = true
  def isSum: Boolean = false
  
  def numValue: Int = n
  def leftOp:Expr = throw new Error("Number.leftOp")
  def rightOp:Expr = throw new Error("Number.rigthOp")
}

class Sum(left:Expr, right:Expr) extends Expr{
  
  def isNumber: Boolean = false
  def isSum: Boolean = true
  
  def numValue: Int = throw new Error("Sum.numValue")
  def leftOp:Expr = left
  def rightOp:Expr = right
}


object ExprTest extends App{
 
  def eval(e: Expr): Int = {
    if (e.isNumber) e.numValue
    else if (e.isSum) eval(e.leftOp) + eval(e.rightOp)
    else throw new Error("Unknown expression" + e)
  }
  
  val x = new Sum(new Number(1), new Number(2))
    println(x.toString())
}

{% endhighlight %}

上面的代码对于实现`Number`和`Sum`是没有问题的，但是它的扩展性不好，比如我们需要增加一个子类`Prod`用来计算乘积和一个子类`Val`来表示变量，我们需要如何改动呢：

+ Expr的接口里需要再增加3个方法，`isProd`，`isVal`，`val`。前两个是classification，第3个是accessor；
+ `Number`和`Sum`需要实现这额外的3个方法。
+ 新增的`Prod`和`Val`需要各自实现8个方法。

所以增加2个新的类一共需要实现25个额外的方法，额外方法的个数与新增子类的个数呈平方关系，有点不好接受。那么如何解决这个问题呢，我们看下面几种解决方法



<ol>
<li><b>Non-Solution：Type Tests and Type Casts</b><br/>

我们在`Expr`里减少这5个方法，在<code>Number</code>，<code>Sum</code>，<code>Prod</code>和<code>Val</code>各自实现各自所需的方法。这样做就不需要<code>Expr</code>的子类实现多余的方法。在<code>def eval(e:Expr):Int</code>里我们只需要用<b>Type Tests</b>和<b>Type Casts</b>来进行实现。<br/>

<p><img src="/assets/images/posts/2015-10-04/type checking.png" align = "middle"></p>

<br/>


{% highlight scala linenos %}

 def eval(e:Expr):Int = {
   if(e.isInstanceOf[Number]) e.asInstanceOf[Number].numValue
   else if(e.isInstanceOf[Sum]) eval(e.asInstanceOf[Sum].leftOp) + eval(e.asInstanceOf[Sum].rightOp)
   else throw new Error("Unknown expression" + e) 
 }

{% endhighlight %}

这个方法的优点是不需要classficiation和accesor方法在<code>Expr</code>里，只需要将access方法在只需要的类里实现即可；缺点是<b>low-level</b> and <b>potentially unsafe</b>。因此我们把这个方法称为<b>Non-Solution</b>。
<br/>
</li>

<li><b>1st-Solution：Object-oriented decomposition</b>

<blockquote><b>Object-oriented decomposition</b>： breaks a large system down into progressively smaller classes or objects that are responsible for some part of the problem domain。</blockquote>


{% highlight scala linenos %}

//OOP decomposition
trait Expr{
  def eval: Int
}

class Number(n:Int) extends Expr{
  def eval: Int = n
}

class Sum(e1:Expr, e2:Expr) extends Expr{
  def eval:Int = e1.eval + e2.eval
}

object ExprTest extends App{
    val x = new Sum(new Number(1), new Number(2))
    println(x.toString())
}	
{% endhighlight %}

<b>OOP decomposition</b>看起来简单多了，如果再需要扩展<code>Prod</code>和<code>Val</code>子类，只需要各自实现如上<code>eval</code> method即可。但是如果我们需要一个方法来展示<b>Expr</b>呢，比如说<code>Show</code> method，我们需要在<code>Expr</code>所有的子类里定义这个新方法。另外如果我们需要实现 a * b + a * c -> a * (b + c)这个提取公因式的method，我们该如何做呢。由于这个方法涉及不同的子类，因此无法通过在一个子类里的method实现，我们必须又得回到该开始的平方增加的例子(增加access method在所有子类里)。

</li>

</ol>



#### 4.2 Pattern Match 语法 ####

{: .img_middle_lg}
![why pattern matching](/assets/images/posts/2015-10-04/why pattern matching.png)

我们来对前面的问题做一个小结，我们的任务是想要同时在**扩展子类**和**扩展接口方法**(接口方法指的是适合所有子类，定义在接口里，而具体实现需要access subclass's own method implementation，即**子类方法**)上获取一个普适的方便的方法：

1. 第一种方法，是将子类**classificatoin**和**子类方法**全部声明在接口中，这样做的弊端是产生了很多不必要的方法，**原有方法的实现**和**扩展类的个数**呈**平方**关系，这还不包括**扩展新的接口方法**，因此这种方法效率极其低下，耦合太紧；
2. 第二种non-solution，在**接口方法**里用**type tests**和**type casts**分别调用**子类方法**。这样如果**扩展子类**只需要在新子类中自己实现相应的**接口方法**调用的**子类方法**，然后在**接口**方法里增加对于这个新的子类的**type tests**和**type casts**。这样做相比于第一个方法貌似灵活了许多，但是却是**low-level**和**unsafe**的；
3. 第三种方法是OOP的decomposition，这其实是我们现在对**接口**的最常用的方法。现在看起来理所当然的方法，但是当我们一步步从这之前走到decomposition所要比较和改进的思路不是那么容易的。OOP的decomposition即在接口里定义**接口方法**，然后在**子类**中直接实现。如果需要**扩展子类**，只需要在子类中实现已有的**接口方法**；如果需要**扩展接口方法**，只需要在接口里**定义新接口方法**，然后分别在**子类里增加实现**。这样我们将**扩展子类**和**扩展接口方法**解耦，使得OOP的Class有着灵活的扩展性。但是对于**扩展接口方法**我们能否做的更好呢？下面我们看看**Pattern Match**。

先上代码。



{% highlight scala linenos %}

object ExprTest extends App{
  trait Expr
  case class Number(n:Int) extends Expr
  case class Sum(e1:Expr,e2:Expr) extends Expr
  
  /* implicit define the companion factory method,Number(2) would be Number.apply(2). No need to use new Number(2) anymore
    object Number{
    def apply(n:Int) = new Number(n)
  }
  
  object Sum{
    def apply(e1:Expr,e2:Expr) = new Sum(e1,e2)
  }
  */
  
  //pattern match 
  def eval(e:Expr):Int = e match{
    case Number(n) => n
    case Sum(e1,e2) => eval(e1) + eval(e2)
  }  
  
  def show(e:Expr):String = e match{
    case Number(n) => n.toString()
    case Sum(e1,e2) => "( " + show(e1) + " + " + show(e2) + " )"
  }
  
  val x = Sum(Number(1), Number(2))
  println(show(x))
}

{% endhighlight %}

我们来一一解释：

1. 首先定义了一个接口 Expr，然后显示表示这个接口的子类是一个所有需要的子类的**枚举**，关键词是**case**。在Scala里这就隐式声明了各个需要子类的工厂方法，当你调用`Number(2)`的时候，会被转换成`Number.apply(2)`(apply的实现就是`new Number(n)`，即new了一个新的Number实例)。

2.然后在接口里直接实现了接口方法，用的就是**Pattern Match**。枚举分流接口方法的子类类型，这里可以看到有在`Sum`里有递归调用，这是**函数式的体现**。

3. 如果我们需要**扩展接口方法**，比如增加一个`Show`的接口方法，同样的使用**Pattern Match**。这样对比与OOP的decomposiont的**扩展接口方法**，可以看到更加简洁方便(不需要在每一个子类中分别实现**接口方法**)。

>因此**Pattern Matching**可以说是函数式OOP的核心(递归调用)，是对命令式OOP接口decompostion的一个提升。**Pattern Matching**建立在scala的**trait**可以有method的实现，和函数式的递归基础上。

**Pattern Matching**的语法如下图

{: .img_middle_lg}
![pattern matching](/assets/images/posts/2015-10-04/pattern matching.png)

#### 4.3 Pattern Match 例子 ####

现在我们将上面这个例子重新改写，将pattern matching 封装在`Expr`里。

{% highlight scala linenos %}
object ExprTest extends App {
  //将pattern matching 封装在trait里
  trait Expr {
    //pattern match 
    def eval(): Int = this match {
      case Number(n)   => n
      case Sum(e1, e2) => e1.eval + e2.eval
    }

    def show(e: Expr): String = e match {
      case Number(n)   => n.toString()
      case Sum(e1, e2) => "( " + show(e1) + " + " + show(e2) + " )"
    }
  }
  case class Number(n: Int) extends Expr
  case class Sum(e1: Expr, e2: Expr) extends Expr

  /* implicit define the companion factory method,Number(2) would be Number.apply(2). No need to use new Number(2) anymore
    object Number{
    def apply(n:Int) = new Number(n)
  }
  
  object Sum{
    def apply(e1:Expr,e2:Expr) = new Sum(e1,e2)
  }
  */

  val x = Sum(Number(1), Number(2)).eval()
  println(x)

}

{% endhighlight %}

这就是`Expr`的最终的实现形式，如果需要**扩展子类**和**扩展接口方法**，聪明的你，肯定知道怎么做了^_^。

### 4.4 Pattern Matching vs OOP decomposition ###

{: .img_middle_lg}
![pattern matching vs oop decompostion](/assets/images/posts/2015-10-04/pattern matching vs oop decompostion.png)

**Pattern Maching**和**OOP Decompostion**各有优劣，那么在我们实际编程中，该如何选择呢：

+ 如果更多的是**扩展子类**，则用**OOP Decompostion**；
+ 如果更多的是**扩展接口方法**，则用**Pattern Matching**


## 5 Assignment ##

本周作业是霍夫曼编码，其中的难点是如何用tuple来做pattern matching来简化实现，举个具体的例子，`def decode(tree: CodeTree, bits: List[Bit]): List[Char]`将整数链表转换成字母链表。用`tuple`只需要4个pattern。但是需要格外小心的是4个pattern的先后顺序，一个实用的经验是从一次完整的函数调用来排列case的顺序。比如这里，对于一个正常的`tree:CodeTree`，如果`bits0`是0开头，则进入左边的`subtree`，1开头则进入右边的`subtree`；如果碰到叶子节点，则将`char`加入到`acc`，然后从头开始调用`tree`和剩下的`bits0`；最后当`bits0`满足`Nil`时，返回`acc`。

这里请同学们思考一个问题，如果将第三个case的`a`改成`x::xs`，结果还是正确的吗？答案是不正确的，因为最后一个字母没有被加入到`acc`里(`x::xs`将`Nil`的情况给剔除了)。

{% highlight scala linenos %}
  /**
   * This function decodes the bit sequence `bits` using the code tree `tree` and returns
   * the resulting list of characters.
   */

  def decode(tree: CodeTree, bits: List[Bit]): List[Char] = {

    def decode0(tree0: CodeTree, bits0: List[Bit], acc: List[Char]): List[Char] = (tree0, bits0) match {
      case (Fork(left, right, chars, weight), 0 :: xs) => decode0(left, xs, acc)
      case (Fork(left, right, chars, weight), 1 :: xs) => decode0(right, xs, acc)
      case (Leaf(char, weight), a)                     => decode0(tree, a, acc :+ char)
      case (x, Nil)                                    => acc
    }

    decode0(tree, bits, List())
  }

  {% endhighlight %}

  具体代码见[这里](https://github.com/shunmian/-2_Functional-Programming-in-Scala)。

## 6 总结 ##

当**OOP**遇上**λ-Calculus**，Scala给出了自己的实现方案：

1. 万物皆类。**Primitive Data**，**Compound Data**，**Primitive Procedure**，**Compound Procedture(Function)**都是类。
2. 函数式的体现不仅在于将函数也表示成类，而且在于实现**Data**的时候优先考虑递归和用**Pattern Matching**更方便的将接口实现用于**扩展接口方法**。当然函数式体现还体现在**Data**实现里的immutable特性。

最后将本节内容总结成一张图。

{: .img_middle_lg}
![OOP with λ-Calculus summary](/assets/images/posts/2015-10-04/OOP with λ-Calculus summary.png)



## 7 参考资料 ##
- [《Structure and Interpretation of Computer Programs》](https://mitpress.mit.edu/sicp/full-text/book/book.html);
- [Martin Odersky: Scala with Style](https://www.youtube.com/watch?v=kkTFx3-duc8);
- [SF Scala: Martin Odersky, Scala -- the Simple Parts](https://www.youtube.com/watch?v=ecekSCX3B4Q);
- [Programming Languages: Lambda Calculus](https://www.youtube.com/watch?v=v1IlyzxP6Sg);
- [Functional Programming For The Rest of Us](http://www.defmacro.org/ramblings/fp.html);
- [Scala Functions vs Methods](http://jim-mcbeath.blogspot.hk/2009/05/scala-functions-vs-methods.html);


