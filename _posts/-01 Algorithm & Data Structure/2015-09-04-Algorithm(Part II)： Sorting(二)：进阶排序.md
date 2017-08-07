---
layout: post
title: Algorithm(二)：Sorting Part II：进阶排序 
categories: [-01 Algorithm]
tags: [Sorting]
number: [-1.3]
fullview: false
shortinfo: 排序是computer science 一个基础而有富有挑战的课题。排序算法是人类智慧的体现，不同的排序算法时间复杂度可以从N<sup>2</sup>>到NlogN。本文介绍6种排序算法，从基础到复杂，让我们体会一下排序算法的魅力。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 2. 排序算法 ##

紧接part I，下面介绍进阶排序算法。

### 2.2 进阶排序 ###

#### 2.2.1 归并排序 ####

##### 2.2.1.1 自顶向下法 #####

> 归并排序(Mergesort): 利用了**分而治之(Divide and Conquer)**方法, 将原始问题分成两个子问题，当两个子问题解决后，再将两个子问题的结果进行归并(Merge).这个正是其名称的由来。归并排序与后文介绍的**快速排序(Quick Sort)**一道，是工业界最为广泛使用的两种排序算法。

{: .img_middle_lg}
![Merge sort](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/merge sort1.png)



{% highlight java linenos %}
public class MergeSort {    
    private static Comparable[] aux;
    
    public static void sort(Comparable[] a){
        aux = new Comparable[a.length];
        sort(a,0,a.length-1);
    }
    
    private static void sort(Comparable[] a, int lo, int hi){
        
        if(hi <= lo) return;
        int mid = lo + (hi-lo)/2;
        sort(a, lo, mid);
        sort(a, mid+1, hi);
        merge(a,lo,mid, hi);
    }
    
    private static void merge(Comparable[] a, int lo,int mid, int hi){
        for(int k = lo; k <= hi; k++){  //copy
            aux[k] = a[k];
        }
        int i = lo;
        int j = mid+1;
        for(int k = lo; k <= hi; k++){
            if      (i > mid)               a[k] = aux[j++];
            else if (j > hi)                a[k] = aux[i++];
            else if (less(aux[j],aux[i]))   a[k] = aux[j++];
            else                            a[k] = aux[i++];
        }
    }
}
{% endhighlight %}

这是一种自顶向下递归求解的方法，容易知道比较次数(compares)满足一下的递推关系：
<p style="text-align: center;">
C(N) = 2C(2N) + N
</p>

求解后可知即使在最坏的情况下，归并排序也能保证经过O(NlgN)次比较就能完成排序，但是，值得注意的是，归并排序需要额外的**线性空间**才能完成，因此不是inplace的。此外在发现相同元素的时候，优先选用左侧的(即靠前)的元素，因此归并排序是stable的。

在工业界的使用中，为了提升归并算法的效率，往往会采用以下两种方案：

+ 在数据规模比较小的时候，直接使用插入排序；
+ 如果两个子序列已经可以完全分开，即第一个子序列的hi不大于第二个子序列的lo，那么无需merge，函数直接返回即可。

此外，Java实现过程中为了保证算法的正确性，可以对invariants采用assert(boolean statement)的方法进行检验。在JVM中可以通过：

{% highlight java linenos %}
java -ea MyProgram //enable assertions
java -da MyProgram //disable assertions(default)
{% endhighlight %}

来控制是否执行assert(boolean statement)语句

##### 2.2.1.2 自底向上法 #####

归并排序的递归实现实际上是DFS，实际上，还有另外一种类似于**BFS的非递归版本**，想法也非常简单，就是手动控制次数反复对2n各元素进行排序，这里的n随着循环的增加而不断加1。从二叉树的角度看，就是从树的底部逐渐往上移动，计算完一层后，向上移动计算上一层，元素个数翻倍，直到到达根节点对所有元素排序。这样只需要经过lgN次的大循环即可完成排序。

{: .img_middle_lg}
![Merge sort2](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/merge sort2.png)

Java代码如下：
{% highlight java linenos %}
private static void merge(Comparable[] a, int lo, int mid, int hi){
    /* as before */
}
public static void sort(Comparable[] a){
    int N = a.length;
    aux = new Comparable[N];
    for(int sz=1; sz<N; sz = sz+sz){
        for(int i=0; i<N-sz; i += sz+sz){
            merge(a, i, i+sz-1, Math.min(N-1, i+sz+sz-1));
        }
    }
}
{% endhighlight %}


由前面的分析可以知道，无论是最好，最坏还是平均的情况下，归并排序的时间复杂度都都是O(NlogN)，这个上界比我们目前介绍的算法都要小，我们就非常纳闷了：

+ **Comparison sort**的下界是什么呢？
+ 进一步来说，**Comparison sort**的渐进复杂度是多少呢？

##### 2.2.1.3 基于比较的排序复杂度分析 #####

一般来说，**Comparison sort**可以用**决策树(Decision Tree)**作为抽象模型，如下图所示：

{: .img_middle_mid}
![Decision tree](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/decision tree.png)

每次的排序结果可以分为**大于**和**不大于**，分别进入决策树的两支，接着继续往下判断，直到最后一次判断能完全决定所有元素的顺序，这样算法终止。也就是说，**排序算法在叶节点终止**。有了这个重要的观察规律，我们可以进行复杂度推导。

考虑从两个方向计算一棵高度为h的决策树的叶子节点数量(#leaves)：

+ 对于N个不同的元素，所有的排序方案数是N!种，因此决策树**至少**有N!个叶节点，之所以说“至少”是因为有可能存在多个节点对应同一只排序方案。因此 


<p style="text-align: center;">
N! ≤ #leave
</p>

+ 对于高度为h的决策树，叶节点的数量最多为2h，因此

<p style="text-align: center;">
#leave ≤ 2h
</p>

由上可知
<p style="text-align: center;">
h ≥ lg(N!)
</p>

又由Stirling公式可知

<p style="text-align: center;">
h ≥ lg(N!) ∼ NlgN
</p>

而h正好就是比较的次数，因此Comparison Sort的下界是Ω(NlgN)，而又由归并排序知上界是O(NlgN)，故Comparison Sort的渐近复杂度就是Θ(NlgN)。

由此可见，归并排序作为复杂性最优的算法，具有非常稳定的效率。然而，归并算法在空间复杂度上并不是最优的，O(N)的空间复杂度让人难以接受，尤其是在现今大数据泛滥的时代。

为此，我们思考，能否找到一种更加理想的排序算法呢？


#### 2.2.2 快速排序 ###

##### 2.2.2.1 2-way 快排 ####

**快速排序(Quick Sort)**最早由**Antony Hoare**(1980年图灵奖获得者)提出，算法的思想和归并排序一样，也是分而治之的方法。具体来说，对于选定的元素pivot(中心点)，通过不停地交换，使得pivot落到最终顺序的对应位置上，并且保证这个时候**左边的所有元素不大于pivot，右边的所有元素不小于pivot**。至此，就把原来的问题划分为两个子问题，再递归地左右两个子问题进行排序。


{: .img_middle_lg}
![2-way Quick Sort](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/2-way Quick Sort2.png)

为了上述的步骤，首先需要确定pivot的选取方法，一种常用的方法是直接选取待排数组的第一个元素，即`pivot = a[lo]`。如上图所示，第一次迭代选取8作为pivot，然后不停交换元素使得8落到对应的正确的位置上，同时把数组划分为两半，分别递归地对两个子问题进行求解，得到最终结果。

完整的Java代码如下所示：
{% highlight java linenos %}
    
public class QuickSort {    
    public static void sort(Comparable[] a){
        StdRandom.shuffle(a);
        sort(a, 0, a.length-1);
    }
    
    private static void sort(Comparable[] a, int lo, int hi){
        if(hi <= lo) return;
        int j = partition(a,lo,hi);
        sort(a, lo,j-1);
        sort(a, j+1,hi);
    }
    
    private static int partition(Comparable[] a, int lo, int hi){
        int i = lo, j = hi+1;
        Comparable v = a[lo];    //刚开始从数组第一个元素开始partition
        while(true){
            while(less(a[++i],v)) if (i == hi) break;
            while(less(v,a[--j])) if (j == lo) break;
            if(i >= j) break;
            exchange(a,i,j);
        }
        exchange(a,lo,j);
        return j;
    }
}
{% endhighlight %}

细心的朋友应该会发现在上述Java的实现中，排序前进行了Shuffle，**为什么要这么做呢？**首先让我们分析一下最坏情况下快速排序的时间复杂度。







{: .img_middle_lg}
![2-way Quick Sort cases](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/2-way Quick Sort cases.png)

如上图所示，（a）为最好情况，(c)为最差情况，(b)为平均情况。在最差情况下，每次选取的pivot在划分后非常不幸都会出现一个**空的子问题**，数组长度缓慢缩短，因此这个情况下的时间复杂度为：

<p style="text-align: center;">
N + (N-1) + ... + 1 ~ 1/2 * N<sup>2</sup>
</p>

当N非常大的时候，这是无法忍受的，因此为了避免上述情况，我们采取洗牌的策略，无论输入的序列如何，我们都能在O(N)的时间内完成洗牌操作，然后再近些快速排序。但是完美主义者不高兴了，即便洗牌也会出现最坏情况呀，打麻将一摸牌就胡的也有呀...Calm Down，当N很大的时候，这个概率比此时此刻坐在电脑前的你被雷劈的概率还要小，所以，大可放心...

下面分析一下快速排在在平均情况下的时间复杂度。

我们采取递推的方法，由快速排序的算法过程我们可以得出以下递推关系：

<p style="text-align: center;">
C<sub>N</sub> = (N + 1) + 1/N * [(C<sub>0</sub> + C<sub>N-1</sub>) + ... + (C<sub>N-1</sub> + C<sub>0</sub>)]
</p>

其中N+1是在partition操作中的比较次数，分式1/N是每种情况发生的概率。求解上述递推，左右乘以N，并且进行逐差，可以得到

<p style="text-align: center;">
C<sub>N</sub>/N+1 -  C<sub>N-1</sub>/N = 2/N+1
</p>

结合出项C<sub>0</sub> = C<sub>1</sub> = 0， N ≥ 2，错位相消后得到：

<p style="text-align: center;">
C<sub>N</sub> = 2(N + 1)(1/3 + 1/4 + ... + 1/N+1) ~ 2(N + 1)lnN ≈ 2NlnN
</p>

又因为
<p style="text-align: center;">
2NlnN = 2N(lgN/lge) = 2N * ln2 * lgN = 1.39NlgN
</p>

因此可见快速排序实际上比归并排序要多39%的比较次数，但是在实际中却快于归并排序，因为快速排序涉及的元素比归并排序少。

##### 2.2.2.2 3-way 快排 ####

上面介绍的是2-way的方法，当数组存在大量的**重复键值(Duplicate Keys)**的时候，这些重复的键值会不管的出现在子问题中，使得数组长度的减少类似worst-case一样缓慢，这个时候的复杂度为O(N<sup>2</sup>)。这里八卦一些野史，就是说自从c语言的`qsort()`出来蛮长的一段时间后，来自AT&T Bell Lab(1991)的**Allan Wilks**和**Rick Recker**发现本来一个瞬间排序结束的`qsort()`却需要好几分钟，后来人们才意识到了qsort()对于重复键值处理得非常糟糕，后来就诞生了额下面将要介绍的3-way方法，如下图所示。

{: .img_middle_lg}
![3-way Quick Sort](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/3-way Quick Sort.png)

Java 代码如下所示：

{% highlight java linenos %}
public static void sort(Comparable[] a, int lo, int hi){  
    if(lo>=hi) return;
    int lt=lo,gt=hi;
    Comparable v = a[lo];
    int i = lo;
    while(i<=gt){
        int cmp = a[i].compareTo(v);
        if (cmp<0) exch(a, lt++, i++);
        else if (cmp>0) exch(a, i, gt--);
        else i++; 
    }
    sort(a, lo, lt-1);
    sort(a, gt+1, hi);
} 
{% endhighlight %}

##### 2.2.2.3 工业级别快排 #####


作为目前最重要的一种算法之一，快速排序的性能已经得到了极大的优化，一般来说，工业级别的快速排序遵循以下几个原则：

+ 对于规模较小的数组(长度不大于8)，使用插入排序取代；
+ Partition的时候用3-way Partition；
+ 对于Pivot的选择：小型规模数组，中间项；中型规模数组：取任意3个元素的中位数；大型规模数组：**Tukey's ninther**(Median of median of 3 samples, each of 3 entries)。

还记得文章一开始提到的信息论角度理解Comparison Sort吗？不同规模数组下的Pivot选择的策略不一样，目的就是尽可能的让Pivot大概率选取到中位数，因为如果Pivot恰好是中位数，就正好*等分**数组，使得获取的信息量最大。


##### 2.2.2.4 选择问题 ####

在实际应用中，往往需要找出排在第k位的元素是多少，这类问题统一归类为**Selection Problem选择问题**。这类问题有显而易见的上界和下界：

+ Upper bound，如果先对数组进行排序，然后直接访问数组中对应的位置，就能解决Selection问题，这样复杂度为O(NlogN);

+ Lower bound，易知无论用何种算法，至少需要对所有的数据扫描一遍，因此下界就很容易确定下来，为Ω(N)。

下面介绍一种介于快速排序思想的方法，**Quick Selection**，这种算法能在平均情况下达到线性，也就是说，把Upper bound进一步降低至O(N)，这样就得到Selection的渐进复杂度Θ(N)。代码如下


{% highlight java linenos %}
public static Comparable select(Comparable[] a, int k){  
    StdRandom.shuffle(a);
    int lo=0, hi=a.length-1;
    while(hi>lo){
        int j = partition(a, lo, hi);
        if (j < k) lo = j+1;
        else if (j > k) hi = j-1;
        else return a[k];
    }
    return a[k];
}
{% endhighlight %}


##### 2.2.2.5 Java实现 ####

前文提到了Java中在调用`Arrays.sort()`函数时，Java会更具传入的数据类型使用归并排序或者快速排序。

+ 如果传入的是Object类型，那么Java会使用归并排序；
+ 如果传入的原始类型(int，double等)，Java会调用快速排序。

这么做的逻辑是若用户传入的是Object，说明用户对于内存空间并不在乎(神逻辑？！)，因此可以放心的申请O(N)的额外空间进行Stable且O(NlogN)guarantee的归并排序；如果用户使用了原始类型，说明用户非常谨慎地考虑了内存使用问题，故Java退而求其次使用快速排序。但无论如何，快速排序和归并排序已经成为工业界的中流砥柱，分别在不同的情况下为计算机事业奉献自己的所有青春和力量，在此表示衷心的敬意！

#### 2.2.3 Heap Sort ####

由前面的介绍知道，不同的数据结构实际上是有着不同的特性，这节首先以**Priority Queue**为起点，引出实现这一数据结构的的等效方法**Binary Heap**，接着思考如何利用这个数据结构实现排序的功能。下面先总结回顾一下目前已经介绍的数据结构的特性：

+ **Stack**，remove the item most recently added；
+ **Queue**，remove the item least recently added；
+ **Randmozied Queue**，remove a random item。

而在实际应用中，往往需要给不同的元素标记不同的优先级，然后根据优先级进行排序，每次选取出优先级最大(或最小)的元素，这就是**Priority Queue**。一般来说，Max Priority Queue需要支持以下的APIs:

+ void insert(Key v);
+ key delMax();
+ boolean isEmpth();

首先上一张Roadmap：

{: .img_middle_mid}
![Priority Queue](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/Priority Queue.png)

##### 2.2.3.1 Elementary Implementation ######

前两种属于基本的实现方案，底层使用的是数组，不同的是一个保持数组有序(ordered)，另一个不需要保持数组有序(Unordered)。下面介绍无序的Java实现代码：


{% highlight java linenos %}

public class UnorderedMaxPQ<Key extends Comparable<Key>>{  
    private Key[] pq;
    private int N;
    public UnorderedMaxPQ(int capacity){
        pq = (Key[]) new Comparable[capacity];
    }
    public boolean isEmtpy(){
        return N==0;
    }
    public void insert(Key x){
        pq[N++] = x;
    }
    public Key delMax(){
        int max = 0;
        for(int i=0; i<N; i++)
            if(less(max,i)) max=i;
        exch(max, N-1); //放到数组末端
        return pq[--N];
    }
}

{% endhighlight %}

由上不难看出，在无序的版本中，insert操作非常迅速，O(1)即可，但是每次执行delMax都需要重新扫描数组，因此是O(N)的复杂度。

有序的版本不同的地方在于插入需要扫描一遍数组找到合适的插入的地方，然后还得不停地交换挪出空间给新插入的数组，因此在insert操作中复杂度为O(N)。由于数组已经有序，因此delMax操作非常迅速，直接删除即可。

哪么，我们思考，有没有更加高效的方法，使得三种操作的复杂度都是lgN呢？

##### 2.2.3.2 Binary Heap Implementation ######

###### 2.3.2.2.1 Binary Tree #######
在目前的介绍中我们发现，对于一个给定的数组，我们所有算法的扫描方式都是固定的，要么以**下标递增**的方式诸葛扫描，要么以**下标递减**的方式逐个扫描。我们不禁思考，有没有其他的方式组织和扫描一个数组呢？

请仔细观察下面的图片：

{: .img_middle_mid}
![binary tree](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/binary tree.png)

观察一下每棵树的分支，发现没有，对于每一个根节点，都会生长出两个子节点出来，并且最高层的所有子节点都有叶节点。这就是计算机中大名鼎鼎的一种数据结构**二叉树 Binary Tree**的特例，**满二叉树(Full Binary Tree)**。

除此之外，还有另外一种二叉树，叫做**完全二叉树(Complete Binary Tree)**，与满二叉树不同，完全二叉树只要求除了最后一层之外的其它层是满的，最优一层的叶节点尽可能的往左靠拢，如下图所示：


{: .img_middle_mid}
![Complete Binary Tree](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/Complete Binary Tree.png)

有人可能会问，这有什么用呀？别着急，如果我们以根节点从1开始逐层对完全二叉树进行编号，观察一下父节点(Parent)和子节点(Child)序号间的关系。发现没有：

+ 子节点k的府机电一定是k/2；
+ 父节点k的两个子节点一定是2k和2K+1。

不服？下面证明一下**m-叉树**的父节点和子节点编号的关系：

对于编号为k的节点，假设其处于第h层(根节点为第0层)，其m个子节点分别为C<sub>1</sub>，C<sub>2</sub>，...，C<sub>m</sub>。那么当层以前的所有节点个数为：


<p style="text-align: center;">
S<sub>h-1</sub> = 1 + m<sup>1</sup> + m<sup>2</sup> + ... + m<sup>h-1</sup> 
</p>

当层k节点前的节点个数为：

<p style="text-align: center;">
k - S<sub>h-1</sub> - 1 
</p>

那么k节点的第一个子节点编号为：

<p style="text-align: center;">
C<sub>1</sub> = m * (k - S<sub>h-1</sub> - 1) + S<sub>h</sub> + 1 = mk - m + S<sub>h</sub> - mS<sub>h-1</sub> + 1
</p>

有等比数列，又知：

<p style="text-align: center;">
S<sub>h</sub> - m *S<sub>h-1</sub> = 1
</p>

因此有：
<p style="text-align: center;">
C<sub>1</sub> = mk - m + 2
</p>

将m = 2带入即可得到子节点分别为2k和2k+1。

有了上面的观察和证明，我们就可以大胆的以树的形式把原来干巴巴的数组串起来，并且可以通过计算公式方便的**在父节点和子节点穿梭自如**。好了，下面进入正题，上面是Binary Heap？


###### 2.3.2.2.2 Binary Heap #######

所谓**二叉堆(Binary Heap)**就是满足一下invariants的数据结构：

+ 父节点与子节点保持固定的大小关系；
+ 大根堆中父节点大于子节点；
+ 小根堆中父节点小于子节点；
+ 两个子节点同样也是二叉堆。

下面就四种不同的情形(scenarios)介绍一下为了维持上述的两个invarant而需要做的操作，注意下面讨论的是大根堆，小根堆同理。

+ S1：子节点的值大于父节点的值；

{: .img_middle_mid}
![Bottom up reheapify](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/S1.png)

如上图所示，子节点T比父节点P的值大，那么只需要将子节点与父节点交换即可，如果交换后依旧比新的父节点大，那么循环操作直到二叉堆的性质满足为止，Java代码非常简单，如下所示。

{% highlight java linenos %}

private void swim(int k){
    while(k > 1 && less(k/2,k)){
        exchange(k/2, k);
        k = k/2;
    }
}

{% endhighlight %}

+ S2:需要插入新的元素有了S1做铺垫，插入新元素的操作就比较简单了，如下图所示，可以首先将新元素添加到末尾，然后调用swim()操作即可，并且能保证做多只需要lgN+1次比较，如下图所示。

{: .img_middle_mid}
![BH insert](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/S2.png)

Java代码如下所示：
{% highlight java linenos %}
public void insert(Key x){
    pq[++N] = x;
    swim(x);
}
{% endhighlight %}

+ S3：父节点的值比子节点的值要小：

{: .img_middle_mid}
![Top down reheapify](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/S3.png)

如上图所示，这种情况下首先需要找出大的子节点，然后不停地与较大的子节点交换，知道满足大根堆的性质，注意这里有2lgN次的比较，Java代码如下：
{% highlight java linenos %}
private void sink(int k){
    while(2*k <= N){
        int j = 2*k;
        if(j < N && less(j, j+1)) j++;
        if(!less(k,j)) break;
        exch(k,j);
        k = j;
    }
}
{% endhighlight %}

+ S4：删除最大(最小)的元素，如下图所示，此时需要把T删除，首先交换T和最后一个元素H，然后对H调用sink函数即可，这里最多2lgN次的比较。

{: .img_middle_mid}
![remove maximum](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/S4.png)

Java代码如下所示：

{% highlight java linenos %}
private void delMax(){
    while(2*k <= N){
        Key max = pq[1];
        exch(1, N--);
        sink(1);
        pq[N+1] = null;
        return max;
    }
}
{% endhighlight %}

妥善处理好了以上的四种情形，那么我们的大根堆实际上就建立完成了，insert()和delMax()操作都实现了，并且都能在O(lgN)的复杂度内解决。除此以外，其实还有另外一种叫做**斐波那契(Fibonacci Heap)**能够进一步把insert的操作降低到O(1)，但是由于实现过于复杂没有得到广泛的实际应用。


{: .img_middle_mid}
![binary heap summary](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/binary heap summary.png)


###### 2.3.2.2.3 Sorting #######

有了二叉堆作为底层的数据结构，堆排序就变得非常的方便，主要分为**建堆**和**排序**两个方面。

{: .img_middle_mid}
![binary heap summary](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/binary heap sorting.png)

###### 2.3.2.2.4 Build Heap #######

建堆的过程非常直观，从倒数第二层开始，往上延伸，每次迭代调用sink函数妥当安置每一个节点，当前层以上的节点都是不可预见未知的节点，直到迭代了根节点为止，Java代码如下所示：

{% highlight java linenos %}
for(int k = N/2; k >=1;k--){
    sink(a,k,N);
}
{% endhighlight %}

如下图所示，先处理E，再处理T-R-O-S

{: .img_middle_lg}
![Heap sorting](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/heap sorting.png)

这是一种**自底向上(Bottom up)**的方法，非常的快速，只需要O(2N)的复杂度即可。

###### 2.3.2.2.5 Sortdown #####

这一步非常简单，就是反复删除对顶的元素，然后提升最后一个元素，调用sink(1)即可，完整的Java代码如下所示：

{% highlight java linenos %}
public static void sort(Comparable[] pq){
    int N = pg.length;
    for(int j = N/2; j >=1; j--){
        sink(pq,k,N);
    }
    while(N>1){
        exch(pq,1,N);
        sinl(pg,1,--N);
    }
}
{% endhighlight %}

###### 2.3.2.2.6 Mathmatics Analysis #####

由上述的过程不难看出，堆排序在最坏情况的也能保证O(2NlgN)的复杂度，并且是inplace的！这是一个里程碑式的发现，回忆一下归并排序和快速排序，前者需要额外的O(N)存储空间，后者在最坏情况下退化到了Quadratic的复杂度。

因此，堆排序是一种同时优化了时间复杂度和空间复杂度的**次完美算法**，之所以这么说是次完美，原因有以下三点：

+ Inner loop longer than quicksort's；
+ Make poor use of cache memory；
+ Not Stable；

对于第一点的理解，堆排序每次需要比较两次，既需要比较两个子节点的大小关系，也需要比较父节点和较大子节点的大小关系。

第二点在现代计算机应用中更为突出，就是不同于快排只是跟相邻的元素做比较，堆排序中需要子节点或者父节点比较，而这些节点，往往离得非常远，因此在效率上较低(数组取值时间复杂度不应该是O(1)？和下标大小有关？)。

第三点显而易见，在反复的交换中难免会打乱原有的相对顺序。

此外，若是从信息论的角度分析堆排序，在删除节点的步骤中，与之交换的末尾节点的值必然不可能是新生成堆的最大值，也就是说，必然会往下sink。换句话说，新的对顶元素交换或者不交货的概率是不相等的，也就不是最优。


## 3. Sorting programming example: 8 Puzzle ##

>8Puzzle: 在3×3的格子中填满1-8的整数，有一个格子是空的。交换空格子和其相邻的格子，以达到最终有序的状态。请给出使得任意N*N（2<N<128）的格子用最少的步数达到有序状态的解。

{: .img_middle_mid}
![8puzzle](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/8Puzzle.jpg)

有两个类用来解决这个问题。

{% highlight java linenos %}
public class Board {
    public Board(int[][] blocks)           // construct a board from an N-by-N array of blocks
                                           // (where blocks[i][j] = block in row i, column j)
    public int dimension()                 // board dimension N
    public int hamming()                   // number of blocks out of place
    public int manhattan()                 // sum of Manhattan distances between blocks and goal
    public boolean isGoal()                // is this board the goal board?
    public Board twin()                    // a board that is obtained by exchanging any pair of blocks
    public boolean equals(Object y)        // does this board equal y?
    public Iterable<Board> neighbors()     // all neighboring boards
    public String toString()               // string representation of this board (in the output format specified below)

    public static void main(String[] args) // unit tests (not graded)
}

public class Solver {
    public Solver(Board initial)           // find a solution to the initial board (using the A* algorithm)
    public boolean isSolvable()            // is the initial board solvable?
    public int moves()                     // min number of moves to solve initial board; -1 if unsolvable
    public Iterable<Board> solution()      // sequence of boards in a shortest solution; null if unsolvable
    public static void main(String[] args) // solve a slider puzzle (given below)
}

{% endhighlight %}

我们用一个MinPQ存储每一个搜索节点SearchNode,每一个搜索节点包括一个Board，一个moves，一个priority，一个manhattan。我们把初始的SearchNode放入MinPQ，然后删除其最小的priority，放入其neighour。重复此步骤直到manhanttan == 0。这里有几点需要注意：

1. MinPQ<SearchNode>的比较需要SearchNode实现Comparable，比较priority,若相同则比较manhattan;
2. 不是每一个Board都可解，但是每一个Board和他的twin(交换任意两个相邻的格子)有且仅有一个可解。
3. 如何获得最终solution？获取最后一个SearchNode(Manhattan == 0),然后previous取其上一个SearchNode直到previous == null;
4. mahattan函数，计算每一个格子需要几步走到(直接移动)最终的格子的步数，然后求1-8的格子的和。


{% highlight java linenos %}
private class SearchNode implements Comparable{

        private Board board;
        private int moves;
        private SearchNode previous;
        private int priority;
        
        public SearchNode(Board board, int moves, SearchNode previous){
            this.board = board;
            this.moves = moves;
            this.previous = previous;
            this.priority = this.moves + this.board.manhattan();
        }
        
        @Override
        public int compareTo(Object x) {
            SearchNode searchNodeX = (SearchNode)x;
            if(this.priority >searchNodeX.priority) return 1;
            else if(this.priority < searchNodeX.priority) return -1;
            else{
                if(this.board.manhattan() > searchNodeX.board.manhattan()) return 1;
                else if(this.board.manhattan() < searchNodeX.board.manhattan()) return -1;
                else return 0;
            }
        }
        
        public String toString(){
            StringBuilder s = new StringBuilder(this.board.toString());
            s.append("move:" + this.moves + "\n");
            s.append("priority:" + this.priority + "\n");
            return s.toString();
        }
    }
{% endhighlight %}


{: .img_middle_lg}
![8puzzle](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/8puzzle game tree.png)

具体code见[这里](https://github.com/shunmian/-01-Algorithm-Princeton)。



## 4 总结 ##

{: .img_middle_hg}
![Sorting algorithm](/assets/images/posts/01_Algorithm/2015-09-04_Algorithm(Part II)： Sorting(二)：进阶排序/Chapter 2_Sorting Summary.png)

我们看到比较接近**holy sorting grail**的算法就是归并排序，快速排序和堆排序。这也就是为什么他们三个常常用来相提并论的原因，各有所长，适用于不同的场合。

## 5 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);

- [数学之美番外篇：快排为什么样快](http://mindhacks.cn/2008/06/13/why-is-quicksort-so-quick/);

