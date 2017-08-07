---
layout: post
title: Algorithm(一)：基础 Part I：Union-Find 并查集
categories: [-01 Algorithm]
tags: [Union-Find]
number: [-1.1]
fullview: false
shortinfo: Union-Find并查集用于检查元素是否属于同一个集合。本文介绍其实现并借此讨论什么是好的算法。最后用其解决一个具体的Percolation问题。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. Union-Find(并查集)建模 ##
我们先来看下面一个问题:

{: .img_middle_mid}
![Percolation](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/percolation.png)
![Percolation](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/thresh.png)

上图是一个n*n的网格，起初每个网格都是关闭的（显示黑色）。我们开启一个个网格(白色)。求开启多少个网格后，上面才会和下面联通,即想象网格上面是水流，网格是一个管道，开启的格子可以流水，得平均开启多少个网格后水流才能通过网格流下来。求这个开启网格占总网格的比例。

这里先列出答案，可能会让你惊讶。当n足够大时开启的比例大于0.593时，上下联通; 小于0.593时，上下不联通。这个问题叫渗透(Percolation)问题，可以用于导体和绝缘体的混合比例，社交网络的节点等现实问题。

这类问题称为**Dynamic Connectivity(动态连接)**。解决这类问题我们得通过Union-Find(并查集)数据结构。可以将这类问题的共性提取出来，用下列API表示。因此解决**Dynamic Connectivity(动态连接)**问题也就退化成实现下列API。

{: .img_middle_hg}
![0_Union Find问题建模](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/0_Union Find问题建模.png)




## 2. 并查集 三种实现 ##

那么什么是Union-Find 数据结构呢，它的定义如下:

{% highlight java linenos %}
public abstract class UF {
    protected int[] id;
    protected int count;
    
    public UF(int n){                           //初始化，用数组存储每个节点
        id = new int[n];
        count = n;
        for(int i = 0; i < n; i++){
            id[i] = i;
        }
    }
    public int count(){return count;}           //返回set个数


    public abstract int find(int p);            //返回每个节点的ID
    public boolean connected(int p, int q){     //返回两个节点是否属于一个set，即ID是否相同
        return find(p) == find(q);
    }
    public abstract void union(int p, int q);   //合并两个节点，即两个节点的ID相同
}

{% endhighlight %}

UF有两个实例变量,int[]数组用来存储每个节点(数组的index)和其ID信息(数组的值); count 表示当前有多少个独立的set。
UF主要有三个public方法，`int find(int p)`返回p节点的ID;`boolean connected(int p, int q)`判断两个节点的ID是否一样,一样表示属于同一个set;`void union(int p, int q)`用来合并两个节点，令他们属于一个set。

我们来看看下面三种具体实现。

### 2.1 快查 ###

{% highlight java linenos %}

public class UF_QuickFind extends UF{

    public UF_QuickFind(int n) {
        super(n);
    }

    public int find(int p){                     //p的ID是id[p]
        return id[p];
    }

    public void union(int p, int q){            //将数组里每一个id和pID相同的节点改成qID。
        if(this.connected(p, q)) return;
        int pID = find(p);
        int qID = find(q);
        for(int i = 0; i < id.length; i++){
            if(id[i] == pID) id[i] = qID; 
        }
        count--;
    }
}
{% endhighlight %}

UF_QuickFind的`find`时间复杂度是o(1),`union`也是o(N)。如果我们要合并N个节点使其成为一个set，时间复杂度是o(N<sup>2</sup>)。其空间复杂度是o(N)。

### 2.2 快并 ###
快查的缺点是每一次`union`都要改每一个id和pID相同的节点，我们完全可以用一个tree data structure来代替它，

{% highlight java linenos %}
public class UF_QuickUnion extends UF{

    public UF_QuickUnion(int n) {
        super(n);
    }

    public int find(int p)              //p的id是其根节点，id[p]存储器父节点
    {  
         while (p != id[p]) p = id[p];
         return p; 
    }
    
    public void union(int p, int q)     //合并p和q节点，只需将p的根节点设为q的根节点的子节点
    {
         int i = find(p);
         int j = find(q);
         if (i == j) return;
         id[i] = j;
         count--; 
    }
}
{% endhighlight %}

利用tree，数组id[]存储的信息量更大了。只有自己的父节点是自己的节点才是跟节点。
UF_QuickUnion的`find`时间复杂度是o(N),`union`也是o(N)。如果我们要合并N个节点使其成为一个set，时间复杂度是o(N<sup>2</sup>)。其空间复杂度是o(N)。UF_QuickUnion只在有些情况下`union`比UF_QuickFind快；在最坏的情况下，还是o(N)。合并N个节点使其成为一个set，时间复杂度是o(N<sup>2</sup>)。

### 2.3 快并+权重+路径压缩 ###
虽然UF_QuickUnion并没有比UF_QuickFind在最坏的情况下提高，但是也是一个improvement。在此基础上，我们加入权重，即当union发生时，Descendant少的tree作为Descendant个数多的tree的Child。这样可以保证tree的height(有些地方也叫rank)总是小于log<sub>2</sub>N。我们首先来看下tree的定义：

1. node分三种: root node, inner node, leaf node;
2. depth of node: number of edges from the node to the tree's root node. A root node will have a depth of 0；
3. height of node: number of edges on the longest path from the node to a leaf. A leaf node will have a height of 0；
4. Root: The top node in a tree.
5. Child: A node directly connected to another node when moving away from the Root.
6. Parent: The converse notion of a child.
7. Siblings: Nodes with the same parent.
8. Descendant: A node reachable by repeated proceeding from parent to child.
9. Ancestor: A node reachable by repeated proceeding from child to parent.

{: .img_middle_mid}
![Percolation](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/tree.png)

然后我们来看下`UF_WieghtedQuickUnion`的实现:

{% highlight objc linenos %}

public class UF_WieghtedQuickUnion extends UF{

    private int[] sz;                           //extra int[] sz存储每个节点的子孙个数(包括自己)

    public UF_WieghtedQuickUnion(int N){
        super(N);
        sz = new int[N];
        for (int i = 0; i < N; i++) sz[i] = 1;
    }

    public int find(int p){                     //和快查一样，查跟节点.
       while (p != id[p]) p = id[p];
       return p; 
    }
    
    public void union(int p, int q){            //让sz小的tree作为sz大的tree的children，而不是反过来
       int i = find(p);
       int j = find(q);
       if (i == j) return;
       // Make smaller root point to larger one.
       if   (sz[i] < sz[j]) { id[i] = j; sz[j] += sz[i]; }
       else {id[j] = i; sz[i] += sz[j];}
       count--; 
       }
}

{% endhighlight %}

权重的好处是包含n个节点的树的高度 < log<sub>2</sub>N。使得`find`的时间复杂度从o(N)降到o(log<sub>2</sub>N); `union`也从o(N)降到o(log<sub>2</sub>N);合并N个节点使其成为一个set，时间复杂度是o(log<sub>2</sub>N)。需要注意的是，权重指的是某tree包括的node，包括自身。

可以再improve加上路径压缩: 每一次`find`,就将其设为Root的Child。

{% highlight objc linenos %}
...
    //路径压缩version 1：找到root后增加一次循环，将path上每个node指向root
    public int find(int p){
       int s = p                     
       while (p != id[p]) {
        p = id[p];
       }
       //already find root as p
       while (s != id[s])
        id[s] = p;    //go through the tree again to set each node point to the root
      }
       return p; 
    }

    //路径压缩version 2：每次将path上的node指向grandparents,使得tree的深度-1。
    public int find(int p){                     
       while (p != id[p]) {
        id[p] = id[id[p]];              //将pnode的Parent's Parent不断往上移，直到移到root。
        p = id[p];
       }
       return p; 
    }
...
{% endhighlight %}

### 2.4 三种时间空间复杂度比较 ###


{: .img_middle_mid}
![Percolation](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/Union_Find_complexity.png)

## 3 算法分析初步 ##

### 3.1 八卦 ###

关于算法分析的起源，还得从**巴贝奇(Charles Babbage)**说起，在他的生活时代里，蒸汽机大规模使用，人类自信心膨胀，相信机械能够完成一切，于是巴贝奇想，能不能设计一个**多项式求值机器**，全自动化运行，避免人为计算的错误。于是，他轰轰烈烈地开展了他的计划，构建**差分机引擎(Difference Engine)**一号，这台机子的最牛的地方在于只需要加减法就能计算一系列多项式的值。于是在英国政府的资助下，他和大工匠**Joseph Clement**一起打造这个宏伟的机器。但是由于过程艰辛，设计图纸一改再改，后来克里木不干了，然后计划流产，最后英国政府一统计才发现，这小子竟然烧了17500英镑，相当于22个火车头。

但是巴贝奇没有气馁，吸取了差分机一号的失败的经验，他先后设计了更加通用化的计算模型**分析引擎(Analytical Engine)**和差分机引擎二号。据说分析引擎还有存储单元和计算单元，后人复现的机器大概长这个样子。


{: .img_middle_mid}
![Percolation](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/Analytical Engine.png)

那么，问题来了：

>究竟需要摇多少下摇杆才能算完呢？

先撇开这个问题不管，咱们继续八卦巴贝奇，话说在他有生之年虽然没能制造出他理想中的机器，但是后人却根据他留下来的大量图纸造出了**可用的差分机**，而费袭击貌似只是做了模型，就是上图那个。此外，也正是这两台机器，诞生了世界上第一个程序员，而且还是女程序员--**Ada Lovelace**。他的父亲就是大名鼎鼎的诗人**拜伦**，她对巴贝奇的机器非常着迷，据说还写出来了第一个伯努利数列求值的算法。后来的**Ada语言**名称就是来源于她。

那么，回到刚刚的问题，变形版本就是：**究竟要算多少次才能得出结果？**

下面先举两个栗子说明研究这个问题的重要性：

1. FFT，由O(N<sup>2</sup>)变到了O(NlogN)的复杂度，给现代多媒体信息压缩并快速传输提供了可能。
2. N-body simulation，常在物理新系统仿真中使用，同样由O(N<sup>2</sup>)变到了O(NlogN)的复杂度，使得许多问题求解虚度大幅度提高。

由上可以看出，算法优化带来的好处是非常直接的，省时省力，无形中节省了整个人类攀登科技树巅峰的时间。那么下面的问题是，给定一个算法的流程，该如何计算这个算法究竟**耗时耗内存**多少呢？

### 3.2 实验###
首先一个算法究竟耗时多少，直观上想，取决于两个因素：

1. Cost，机器执行每一条指令所用的时间，取决于*机器本身**及**编译器**；
2. Frequency，一个算法究竟需要执行多少条指令，取决于**算法本身**以及**输入规模**。

只要知道了上述连个值，那么直接用
<p style="text-align: center;">
T<sub>N</sub> = Cost<sub>1</sub> × Frequency<sub>1</sub> + ... + Cost<sub>N</sub> × Frequency<sub>N</sub>
</p>
就能算出最后的耗时了。

那么接下来一个棘手的问题是，真的要一行一行的去数吗？并且**Discrete Sum**有时候还需要蛮高的数学技巧才能求出和值，比如转化为**连续积分**等技巧。因此，我们暂且撇开数学，看看能不能通过做实验的方法求出一个算法的耗时T关于规模N的增长函数T(N)呢？

先看我们取到的数据。

{: .img_middle_mid}
![problem size](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/problem size.png)

首先假设T(N)的形式，不多解释(设为指数形式T(N) = ca<sup>N</sup>可以直接去撞墙)，默认为幂指数函数T(N) = aN<sup>b</sup>，然后通过常规的待定系数的方法，首先对T(N)两边取log，得到表达式：
<p style="text-align: center;">
log(T<sub>N</sub>) = blogN + loga
</p>

利用以上数据绘制log-log图，得到：

{: .img_middle_mid}
![log-log](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/log.png)

然后就可以用**linear regression**的方法待定求出a和b啦，最后结果是这个
<p style="text-align: center;">
T<sub>N</sub> = 1.006 × 10<sup>-10</sup> × N<sup>2.999</sup>
</p>

大功告成，可以洗洗睡了？不，大家有没有发现这样做就麻烦了，有没有更加快速的求解方法，当然有--**Doubling Approach**

我们研究当输入规模翻倍的时候T(N)会发生什么有趣的变化：
<p style="text-align: center;">
Ratio = T<sub>2N</sub>/T<sub>N</sub> = a(2N)<sup>b</sup> / a(N)<sup>b</sup> = 2<sup>b</sup>
</p>

进一步

<p style="text-align: center;">
log(Ratio) = b
</p>

也就是说，当N不停翻倍并且比较大的时候，log(Ratio)是趋向于b的，这样好办了，我们于是算算log(Ratio)的变化：

{: .img_middle_mid}
![log ratio](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/log ratio.png)

太棒了，一眼就看出了b = 3，然后接下来的只需要用任意一个点待定出剩下的a就OK了。但是程序员是不会局限于用做实验的low方法给出算法复杂度的，他们一般都追求数学上的完备，于是接下来介绍如何用数学的方法应对算法复杂度。

### 3.3 符号系统###

数学分析不可少，其实正是**高爷爷**的**TAOCP**将算法复杂度数学分析方法发扬光大的，下面先介绍一系列的符号Notation:

1. Tilde notation，~，也就是波浪号，表示的是高阶项，即**leading term**，只计算算法中最耗时的部分，这么做的原因很容易证明：当输入规模N非常大时，低阶项直接忽略；当输入规模N不大的时候，低阶项非常小，也能忽略。
2. Big Theta，Θ(N)，表示的是渐近复杂度，常用来区分算法的本质。
3. Big Oh，O(N)，是表示的Θ(N)或者更低复杂度，也就是上界(Upper bound)。
4. Big Omega， Ω(N)，表示的是Θ(N)或者更高复杂度，也就是下界(Lower bound)。

由上不难看出，在Upper bound和Lower bound的层层紧逼下，算法的最优化版本就会逐渐浮出水面，最理想的情况是当上下界相等的时候，那么类似于极限证明里面的**夹逼法**，Optimal Algorithm就显而易见了。例如**1-Sum**问题的求解，上下界分别为O(N)和Ω(N)，那么最优算法复杂度自然是Θ(N)。

关于**x-Sum**问题的定义，说的其实就是从一个数组a[N]中同时出x个数，这x个数加起来的和为0的组合有哪些？暴力遍历？复杂度显而易见是O(C<sub>N</sub><sup>x</sup>)。

那么，下面研究**3-Sum**，探索一下是否能将O(N<sup>3</sup>)的Upper bound进一步降低呢？

### 3.4 二分搜索###

实际上，据说写二分搜索的无bug版本还是蛮难的，第一版的bug-free版本还是1962年才找到，另外哪怕是进入21世纪的2006年，java的库函数`Array.binarySearch()`还发现有bug。

简单来说，二分搜索的前提是**数组有序**，这样的话，只需要定义`lo`和`hi`，然后计算`mid = lo + (hi-lo)/2`，不断用mid替换lo和hi，直到`lo>hi`。千言万语不如撸代码:

{% highlight java linenos %}
public static int binarySearch(int[] a, int key){
    int lo = 0, hi = a.length-1;
    while(lo<=hi){
        int mid = lo + (hi - lo)/2;
        if     (key < a[mid]) hi = mid - 1;
        else if(key = a[mid]) return mid;
        else                  lo = mid + 1;
    }
    return -1;
}

{% endhighlight %}

while中的循环主题就是**3-way compare**,分析上述算法的复杂度，不难得到一下递推关系：
<p style="text-align: center;">
T(N) ≤ T(N/2) + 1
</p>

不断迭代就能得到：
        
<p style="text-align: center;">
T(N) ≤ 1 + lgN
</p>
至此，我们已经可以在一**排好序**的数组中以lgN找出某一特定元素，注意前提是排号序。

### 3.5 3-Sum问题###

首先将问题明确定义一下：
>在一个包含N个元素的数组a[N]中，找出所有三元组使得其加起来的和为0。

显而易见，直接三次循环遍历，一定能把问题解决，但是时间复杂度是O(N<sup>3</sup>)，我们思考能不能借用上述的**二分查找**来把这个Upper bound给进一步降低。

既然是三个数，不妨设为a[i]，a[j]，a[k]的和为0，即：
<p style="text-align: center;">
a[i] + a[j] + a[k] = 0
</p>
那么进行简单的数学运算，可以得到：
<p style="text-align: center;">
a[k] = -(a[i] + a[j])
</p>

脑洞一开，左边不就是一个待搜索的值吗，于是我们想到另外一种解决方案：

1. 首先将a[N]进行从小到大的排序；
2. 然后利用两层循环确定a[i]和a[j]，计算-(a[i] + a[j])；
3. 接着调用**二分搜索**的方法确定a[k] = -(a[i] + a[j])的位置，如果没有则返回-1。

分析一下上述新算法的复杂度：

1. 排序部分，哪怕用最naive的**Insertion Sort**的方法，复杂度也就是O(N<sup>2</sup>)；
2. 主体部分，容易知道复杂度是O(N<sup>2</sup>logN)；
3. 因此总的算法复杂度是是O(N<sup>2</sup>logN)；

对比**Brute Force Approach**的O(N<sup>3</sup>)，Upper Bound进一步降低；另一方面，3-Sum问题的一个Lower Bound非常容易想到，就是至少遍历一遍所有的元素，复杂度是Ω(N)。于是我们可以得到3-Sum问题的渐进复杂度已定价与这两个界限之间，即：
<p style="text-align: center;">
Ω(N) ≤ Optimal Algorithm ≤ O(N<sup>2</sup>logN)
</p>

界限之间存在Gap，那么究竟3-Sum问题的渐进复杂度是多少呢，留下一个Open Question...

### 3.6 怎样的算法才叫好 ###

一般来说，算法的渐近复杂度之间存在相对的好坏，排序如下:
<p style="text-align: center;">
Θ(1) ≤ Θ(logN) ≤ Θ(N) ≤ Θ(NlogN) ≤ Θ(N<sup>2</sup>) ≤ Θ(N<sup>3</sup>) ≤ Θ(2<sup>N</sup>)
</p>

一个有意思的问题是

> 怎么样的算法复杂度才叫好？

一个容易想到的答案是，必须得要Θ(N)才行，再给出结论之前，先看下面一张图：

{: .img_middle_mid}
![problem size](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/problem size.png)

分析一下上图，首先人类设计的通用计算机已经早在1970s就能解决所有复杂度为Θ(1)和Θ(logN)的算法，这是一个值得骄傲的事情。那么，对于稍微复杂的Θ(N)呢，到了2000s也能在1s内计算10亿次，键值吓尿了，虽说不上any，但是也是妥妥的搞定了。这样，必须得让算法复杂度降到Θ(N)的要求显得有点高，不妨放宽一点，Θ(NlogN)怎么样。

大家都知道**摩尔定理**，每隔一定的时间，计算机的存储和计算速度都翻一番。如果这样的话，存储翻一番，问题的规模就×2，假设算法复杂度为Θ(N<sup>2</sup>)，那么总复杂度×4，虽然运算速度×2，最后求解时间还是×2。于是我们就得出结论Θ(N<sup>2</sup>)的算法是**不Scalable**的，跟不上摩尔定律，随着计算机的发展，求解的时间实际上是**翻倍上涨**的，简直无法忍受。那么Θ(NlogN)呢？

用同样的方法尽心计算，可以得到
<p style="text-align: center;">
2Nlog(2N) / NlogN * 1/2 = log2N/logN = 1 + log2/logN
</p>

显然这个比值是趋向于1的，因此我们恍然大悟，啊，难怪**FFT**和**N-Body Simulation**的问题被降到了Θ(NlogN)后被世人大加赞赏，原来真的是意义重大啊，因为Θ(NlogN)的算法是**Scalable**的，让计算机发展来得更猛烈些吧。

### 3.7 空间复杂度 ###

相比于时间复杂度的计算，空间复杂度相对容易一些，只需要把所有闸弄的内存加起来即可。下图给出了JVM中一些基本类型的占用内存情况。

{: .img_middle_mid}
![memory_0](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/memory_0.png)

注意在JVM的实现中，堆得占用情况是**按8bytes对齐**的，因此会看到boolean等类型向上取到了8bytes，不够的部分JVM会自动填补。

要注意的是Object有16 bytes的overhead用于存储Object的信息; 指针大小为8 bytes; padding是确保每一个Object的大小都是8的倍数; 如果有内部类，则有额外的reference 8 byte指向所在的类。

{: .img_middle_mid}
![memory_1](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/memory_1.png)
![memory_2](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/memory_2.png)

在内存已经如此廉价的今天，已经没有太大必要深究内存的占用情况，一种性价比高的方法是开启`-XX:+UseCompressedOops`选项，这个选项会把Object Reference有16bytes降到8bytes，其他的就多使用原始类型吧，详细的优化原理请参见[这里](http://btoddb-java-sizing.blogspot.hk/)

## 4 Programming Assingment: Percolation##

下面我们回过头来看Percolation的问题,它给定公共API如下：

{% highlight java linenos %}
public class Percolation {
   public Percolation(int N)               // create N-by-N grid, with all sites blocked
   public void open(int i, int j)          // open site (row i, column j) if it is not open already
   public boolean isOpen(int i, int j)     // is site (row i, column j) open?
   public boolean isFull(int i, int j)     // is site (row i, column j) full?
   public boolean percolates()             // does the system percolate?

   public static void main(String[] args)  // test client (optional)
}
{% endhighlight %}

解题思路: 

1. 我们将int[N][N]2维数组映射到一个一维int[N<sup>2</sup>+2]数组(WQUUF(N<sup>2</sup>+2)里的数组表示)。一维多出来的两个分别是virtual top和 virtual bottom。
2. open函数里，如果该网格在第一排，则union(0,xyTo1D(i,j))；第N排，则union(N<sup>2</sup>+1,xyTo1D(i,j))；其他情况若neighbour开启，则union。
3. 这里有一个倒流的情况，因此得额外加一个WQUUF(N<sup>2</sup>+1)，少了virtual bottom，来实现isFull。还得用一个二维布尔数组boolean[][]来存储开启关闭情况。具体code见[这里](https://github.com/shunmian/-01-Algorithm-Princeton)。



最后上一张跑分图。

{: .img_middle_lg}
![score](/assets/images/posts/01_Algorithm/2015-09-01_Algorithm(Part I)： 基础(一)：Union-Find 并查集/score.png)



## 5 总结 ##


本文用我们用Union-Find 并查集实现了Percolation threshhold p = 0.593的计算。Good Code = Algorithm + Data Structure 这句话我们有了更深入的认识。什么是好的算法，并查集如何降低时间复杂度，percolation问题如何解决总结如下:

1. Union-Find利用quick union + weighted + path compression 可以将Union和Find的时间复杂度从O(N)降为log<sub>2</sub>N；
2. 用0 to N-1的整数表示N个site简化了并查集的模型。可以用Symbol Table来建立整数和其他object的联系。
3. 好的算法(**Scalable**)的时间复杂度 ≤ O(NlogN)；
4. 我们用这个WQUUF数据结构处理了Percolation问题，并得到了概率阈值 p = 0.593；
5. "A good algorithms must be seen to be believed" -Donald Ervin Knuth. 这句话的意思是理解好的算法的通用的可实践的方法不是阅读它，而是用笔和纸一步步画出算法的演算，来visualize它的过程。这是[Visualize Algorithm](http://visualgo.net/)和[Algorithm](http://algs4.cs.princeton.edu/home/)教授算法的主要方法。



## 6 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);

- [[ALGORITHMS] UNION-FIND](http://aaronxic.com/algorithm-week1-union-find/#.Vx2QY6N94cg)




