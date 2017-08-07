---
layout: post
title: Algorithm(一)：基础 Part II：Stack 栈，Queue 队列 
categories: [-01 Algorithm]
tags: [Queue, Bag, Stack, LinkedList, Array, Generics, Iterable]
number: [-1.2]
fullview: false
shortinfo: 包，队列，栈是基本的数据结构，在计算机科学里有着广泛的应用。本文用链表和数组来实现包，队列和栈。借着这三个数据结构我们还介绍泛型(Generics)和可迭代(Iterable)接口。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. 介绍 ##
包(Bag)，队列(Queue)，栈(Stack) 在计算机科学里作为最重要的三个基本数据结构之一有着广泛的应用。对于他们的实现,有两种方案，分别是基于链表(Linked-List)和数组(Array)。链表和数组对于查找(前者是O(N),后者是O(1))和增删(前者是O(1),后者是O(N))各自有优势。下面我们先介绍链表和数组，再用他们实现包，队列和栈。最后我们引入泛型的概念和可迭代的接口来使得包，队列，栈能够优雅地实现存储各个类型的数据以及快速枚举。

## 2. 链表(Linked-List)，数组(Array) ##
链表便于增删(O(1))，查找效率低(O(N)); 数组便于查找(用下标O(1)),增删效率低(由于固定长度，增加减少元素时要copy到新的Array。最优算法：items = capacity时，capactiy double; items = 1/4 *capacity时,capacity half。时间复杂度是(O(N))。链表和数组的空间复杂度都是O(N)。

{: .img_middle_mid}
![LinkedList&Array](/assets/images/posts/01_Algorithm/2015-09-02_Algorithm(Part I)： 基础(二)：Stack 栈，Queue 队列/LinkedList.png)

## 3. 队列(Queue)，栈(Stack) ##





### 3.1 栈 ###
队列是LIFO(后进先出)。需要1个操作位，指向栈的尾部。
{% highlight java linenos %}
class StackOfStrings{
    StackOfStrings()            //create an empty stack
    void push(String item)      //insert a new string onto stack
    String pop()                //remove and return the string
    most recently added
    boolean isEmpty()           //is the stack empty?
    int size()                  //number of strings on the stack
}
{% endhighlight %}

#### 3.1.1 List实现 ####
实现栈只需要实现上述四个API接口即可，维护链表(List)相对简单，处理好指针的赋值即可，内从空间管理比较高效，并且各个操作所小号的时间固定，方差小，因此链表适合快速响应`pop()`和`push(Item item)`的操作场合，比如**实时的通信系统**。

{: .img_middle_lg}
![Stack](/assets/images/posts/01_Algorithm/2015-09-02_Algorithm(Part I)： 基础(二)：Stack 栈，Queue 队列/stack.png)


#### 3.1.2 Array实现 ####
数组的优点是**随机访问(Random Access)**，访问时只需要索引下标即可，复杂度是O(1)，但是由于数组大小固定，内存维护起来稍微复杂。例如在`push(Item item)`的时候，如果数组容量不够了需要申请新的更大容量的数组，并且将原有的数据**一一复制**到新的数组中。这样`push(Item item)`的时间消耗方差比较大，最好的情况是直接插入，O(1)复杂度；最坏的情况是遍历一遍原有的数组，O(N)复杂度。因此，这对增大**数组容量**的策略提出了考验。

一种直接能想到的策略是每次增加1的容量，但是这样的最坏的情况是连续`push(Item item)`操作，那么时间复杂度为:

<p style="text-align: center;">
 1 + 2 + 3 + ... + N ~ O(N<sup>2</sup>)
</p>
简直无法忍受，因此我们想到能否增大新数组的增量来降低复杂度，每次翻倍如何？计算一下**均摊复杂度(Amortized Cost)**：
<p style="text-align: center;">
 N + (2 + 4 + 8 + ... + N) ~ O(3N)
</p>
这里需要注意一下实现的细节，就是运用第二种策略的时候，初始的时候数组大小为2，那么每次除了要添加新的元素外(公式里的N)，每次数组容量到达上限后需要重新复制的时候也需要额外的操作，并且操作步数是以2为公比的等比数列，因此求和后的复杂度为O(3N)。
`push(Item item)`的复杂度成功的得到了降低，继续思考，这样数组的维护就足够了吗？如果执行了`pop()`操作呢？容量为100的数组只存1个元素习大大会不高兴的，本着**厉行节约，反对浪费**的主旨，我们需要制定内存回收的机制。

直接能想到的方案就是如法炮制，当数组元素个数降到数组容量的一半的时候，申请容量减半的心数组并且一一复制。但是，仔细分析发现如果恰好一组`push(Item item)`和`pop()`连续交替操作发生在数组容量为2<sup>m</su>的时候，会发生不断申请和释放内存的情况，并且每次操作都要遍历原有数组。

改进的办法就是当数组元素个数降到原有数组容量的1/4的时候执行减半操作，也就是说，数组的大小在数组容量的25% ~ 100%直接波动，这样能避免频繁的震荡。

#### 3.1.3 Java库的List ####

在Java中自带了List数据结构，并且分别用联保和数组实现了两个版本，如下所示：

1. `java.util.LinkedList`，利用了linked list；
2. `java.util.ArrayList`，利用了resizing array。


#### 3.2.4 Stack application #### 

利用栈的LIFO性质，可以用来解决**算术表达式求值**问题，同样也基于实现简化版的解释器。具体来说，例如求以下表达式的代数值：

<p style="text-align: center;">
 (1 + ((2 + 3) * (4 * 5)))
</p>

解决方案是利用**Dijkstra**的**双栈算法(2-stackalgorithm)**，简单来说，就是维护两个栈，**操作数栈(Operand Stack)**和**操作符栈(Operator Stack)**，前者就是被运算的数字，后者就是具体运算。算法如下：

1. Push Operands onto the operand stack；
2. Push operator onto the operator stack；
3. Ignore left parentheses；
4. On encountering a right parenthesis, pop and Operator, pop the requisite number of operands, and push onto the operand stack the result of applying that operator to thoses operands。

### 3.2 队列###
队列是FIFO(先进先出)。需要两个操作位，分别指向队列的头和尾。因此实现上只需要在Stack的基础上做些许调整即可。队列需要的API如下。

{% highlight java linenos %}
class QueueOfStrings{
    QueueOfStrings{}            //create an empty queue
    void enqueue(String item)   //insert a new string onto queue
    String dequeue()            //remove and return the string least recently added
    boolean isEmpty()           //is the queue empty?
    int size()                  //number of strings on the queue
}
{% endhighlight %}

下面介绍Queue的两个比较有意思的衍生数据结构，**Dequeue**和**Randomized Queue**

#### 3.2.1 Dequeue ####

Deque(Double-ended Queue)是双端队列，即允许在队列的两端进行插入和取出，其实相当于两个反向的栈拼接在一起，如下图所示。

{: .img_middle_mid}
![Deque](/assets/images/posts/01_Algorithm/2015-09-02_Algorithm(Part I)： 基础(二)：Stack 栈，Queue 队列/deque.png)

deque的API如下：
{% highlight java linenos %}
class Deque{
    boolean isEmpty();
    int size();
    void addFirst(Item item);
    void addLast(Item item);
    Item removeFirst()
    Item removeLast()
}
{% endhighlight %}

和Stack一样，可以用List和Array两种方式实现Deque,相对而言List的实现简单，可以利用双向链表，即每个节点包含`pre`和`next`指针，这样维护起来非常的方便高效。这其实是week2的programming assignemnt，具体代码见[这里](https://github.com/shunmian/Algorithm)。

#### 3.2.2 Randomized Queue ####
**随机化队列**也就是一个增加了删除元素操作的**包(Bag)**(包：只增加元素不能删除元素的无序集合)本质上就是一堆无序的元素，如下所示：

{: .img_middle_mid}
![Bag](/assets/images/posts/01_Algorithm/2015-09-02_Algorithm(Part I)： 基础(二)：Stack 栈，Queue 队列/bag.png)

随机化队列是无序的，需要的API如下：
{% highlight java linenos %}
class BagOfStrings{
    BagOfStrings()              //create an empty bag
    boolean isEmpty()           //empty or not
    void enqueue(String x)      //insert a new String item onto bag
    int size()                  //number of items in bag
    String sample()             //随机返回一个String item
    String dequeue()            //随机返回一个String item 并删除
}
{% endhighlight %}

其中最核心的是后面的`Item dequeue()`和`Item sample()`，前者要求随机取出一个元素，并且删除；后者则要求返回一个元素，并不需要删除。

囿于笔者的水平，考虑到如果用List实现RandomizedQueue随机化不好处理，于是考虑利用Array实现，大致思想是生成随机序列作为索引的下标，然后依次将合法的元素取出即可，具体的实现参见[这里](https://github.com/shunmian/Algorithm)。

## 4 泛型(Generic) ##
如果我们存储的不只是String，还有Integer，Double,或者我们自己的Object在包，队列，或者栈里。我们如果再实现那些类型的包，队列，栈，就有大量重复的代码。泛型优雅地解决了这个问题，将类型参数化。

> 泛型<Item>：将类型参数化，相当于给"类型"可选，在类内部可以对Item进行引用。

{% highlight java linenos %}
public class FixedCapacityStack<Item>
{
 private Item[] s;
 private int N = 0;

 public FixedCapacityStack(int capacity)
 { s = (Item[]) new Object[capacity]; }

 public boolean isEmpty()
 { return N == 0; }

 public void push(Item item)
 { s[N++] = item; }

 public Item pop()
 { return s[--N]; }
}
{% endhighlight %}


## 5 可迭代(Iterable) ##
泛型大大加强了基本数据结构的泛化能力，但是对于程序员而言，适当简介的代码不仅能增强程序的可读性，并且能减少代码的长度，就像一颗糖一样甜蜜蜜，含在嘴中爱不释口，这就是语法糖。
在Java中允许使用foreach statement，即支持以下用法:
{% highlight java linenos %}
for (String s : stack)
 StdOut.println(s);
{% endhighlight %}

但是前提是stack的类型必须实现Iterable的接口:

1. Iterable是Iterator(实现了hasNext和next方法)的工厂方法，实现了Iterable就能用foreach快速遍历。
2. Iterator 需要实现 `hasNext()`和`next()`方法。

Iterable：
{% highlight java linenos %}
public interface Iterable<Item>
{
 Iterator<Item> iterator();
}
{% endhighlight %}


Iterator：
{% highlight java linenos %}
public interface Iterator<Item>
{
 boolean hasNext();
 Item next();
 void remove();
}
{% endhighlight %}

## 6 Programming Assingment: Deque and RandomizedQueue##

本周作业如下:

Deque(可以从头尾入队出队)，RandomizedQueue(随机出队)。

Deque的思路是用一个双向的`node(Item item, Node previous, Node next)`来用链表实现。

RandomizedQueue的实现用数组很容易想到，用stdRandom.uniform(N)来取item，关键的问题是如何维护这个数组:

1. Step 1, 容量不足double数组，容量1/4,half 数组。这样才能满足均摊时间复杂度为O(1)的要求；
2. Step 2, 但是仔细一想dequeue的实现，随机出队一个a[i]，则在resize条件未满足前(Step 1)，数组a的中间有空的元素，那么下次dequeue，在同样的数组a下，如何剔除掉这个空的元素？方法是一直取随机数i，直到取到下一个不为null的a[i]；
3. Step 3, enqueue策略如何选择？可以用一个iVar int tail来保存数组最后一个有效item的index，每次enqueue先判断Step1，再加到tail++。那么enqueue的是时候如何判断step1呢？是用N(item 个数)还是tail呢？答案是用tail，当tail<a.length/4的时候，resize(a.length/2)。

主要思路如上，其中有一些细节需要读者仔细斟酌。具体code见[这里](https://github.com/shunmian/-01-Algorithm-Princeton)。下面贴一张跑分图。

{: .img_middle_lg}
![Assessment](/assets/images/posts/01_Algorithm/2015-09-02_Algorithm(Part I)： 基础(二)：Stack 栈，Queue 队列/assessment.png)



## 7 总结 ##


{: .img_middle_hg}
![Assessment](/assets/images/posts/01_Algorithm/2015-09-02_Algorithm(Part I)： 基础(二)：Stack 栈，Queue 队列/Chapter 1 Foundamental Summary.png)

## 8 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);
- [Visualize Algorithm](http://visualgo.net/);
- [[ALGORITHMS] STACKS AND QUEUES](http://aaronxic.com/algorithms-week2-stacks-and-queues/#.Vx3WL6N941g);




