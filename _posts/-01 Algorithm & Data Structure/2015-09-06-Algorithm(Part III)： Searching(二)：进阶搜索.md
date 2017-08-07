---
layout: post
title: Algorithm(三)：Searching Part II：进阶搜索
categories: [-01 Algorithm]
tags: [Searching]
number: [-2.2.4]
fullview: false
shortinfo: Searching 搜索是现代计算机和互联网的基础。本文介绍搜索算法的前世今生。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 2. 搜索算法 ##


{: .img_middle_lg}
![Sorting Algorithms](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/searching performance0.png)

### 2.2 进阶搜索 ###

#### 2.2.1 Balanced Search Tree ####

##### 2.2.1.1 2-3 Tree ####

2.3二叉搜索树在average case下insert/search 时间复杂度是O(logN)，但是在worst-case下是O(N)。如何保证在任何情况下insert/search 都是O(logN)呢。这就是我们要介绍的平衡二叉搜索树。理想情况下，我们想要保持我们的binary search tree perfectly balanced使得所有的search 时间复杂度都是O(logN)。但是不幸的是，保持perfectly blanaced 需要通过昂贵的insertion操作。因此我们变通一下，下面介绍 2-3 search tree。 

<blockquote><b>2-3 search tree (2-3搜索树)</b>: is either
<ul>
<li>a tree that is empty; </li>
<li>a 2-node with one key (and associated value) and two links, a left link to a 2-3 search treee with smaller keys, and a right link to a 2-3 search tree with larger keys; </li>
<li>a 3-node, with two keys (and associated values) and three links, a left link to a 2-3 search tree with smaller keys, a middle link to a 2-3 search tree with keys between the node's keys, and a right link to a 2-3 search tree with larger keys。</li>
</ul>
</blockquote>

{: .img_middle_mid}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/2-3 search tree2.png)


2-3 tree 有2个极好的特性是: 

1. Symmetric Order: node左边小于，右边大于。同时适用于2-node和3-node(中间介于两node)。

2. Perfect Balance: every path from root to null link has same length。

2-3 tree相应的search和insert操作如下图。

{: .img_middle_hg}
![Sorting Algorithms](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/2-3 Tree search & insert.png)

2-3 tree的代码实现比较复杂，有一个更简单的等价实现，即下面介绍的红黑树。

##### 2.2.1.2 红黑树 #####


{: .img_middle_lg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/red-balck BST introduction.png)

> **Red-Black BSTs:** 红黑二叉搜索树的本质是2-3 tree(一个节点最多有2个key，而红黑树则使用染色的方式来标识这两个key)作为平衡的二叉搜索树的一个特例。而保持平衡是通过以下5点红黑树的性质来确保。1.每个结点要门是红的，要么是黑的；2.根结点是黑的；3.每个叶节点是黑的；4.如果1个结点是红的，那么它的两个子结点都是黑的；5.所有从root到叶结点的黑结点数一样(完美黑结点平衡)。

我们先定义三种基本操作，

{: .img_middle_lg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/red-black BST 3 basic operatoin.png)

下面是基于上述3种基本操作实现的API。
需要注意的是新加的node的颜色为RED，代表parent连接本node的颜色。

{% highlight java linenos %}
public class RedBlackBST<Key extends Comparable<Key>, Value>{
    private Node root;
    private class Node // BST node with color bit (see page 433)
    private boolean isRed(Node h)    
    private Node rotateLeft(Node h)  
    private Node rotateRight(Node h) 
    private void flipColors(Node h)  
    private int size()               

    public void put(Key key, Value val){  // Search for key. Update value if found; grow table if new.
        root = put(root, key, val);
        root.color = BLACK;
    }

    // get remain untouched.
    public Value get(Key key) {
        return get(root, key);
    }

   
    private Value get(Node x, Key key) {
        while (x != null) {
            int cmp = key.compareTo(x.key);
            if      (cmp < 0) x = x.left;
            else if (cmp > 0) x = x.right;
            else              return x.val;
        }
        return null;
    }

    private Node put(Node h, Key key, Value val){
    if (h == null)return new Node(key, val, 1, RED);  // Do standard insert, with red link to parent.
    int cmp = key.compareTo(h.key);
    if      (cmp < 0) h.left  = put(h.left,  key, val);
    else if (cmp > 0) h.right = put(h.right, key, val);
    else h.val = val;

    if (isRed(h.right) && !isRed(h.left))    h = rotateLeft(h);
    if (isRed(h.left) && isRed(h.left.left)) h = rotateRight(h);
    if (isRed(h.left) && isRed(h.right))     flipColors(h);
    h.N = size(h.left) + size(h.right) + 1;
    return h; 
    }
}
{% endhighlight %}

其中**Search**的实现和BST一样，忽略颜色的影响。因为Red-Black Tree有更好的平衡，所以其Search不会遇到BST的worst case。

{: .img_middle_lg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/red-balck BST Search.png)

**Insertion**的实现稍显复杂，包括几种情况，见下图。

{: .img_middle_hg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/red-balck BST Insertion.png)

##### 2.2.1.3 B-tree #####

> **B-tree:**是2-3 tree的泛化，每个node可以包括M/2个Key到M-1个Key。

{: .img_middle_hg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/B tree.png)


##### 2.2.1.4 Geometric Appliction of BST #####

###### 2.2.1.4.1 1d Range Search #####

> **1d Range Search:**所有key为1维直线上的点，求某段(lo,hi)之间有哪些key。

{: .img_middle_hg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/1d range search.png)


###### 2.2.1.4.2 Line Segment Intersection #####

> **line segment intersection:**求正交线段的所有交点。

{: .img_middle_hg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/line segment intersection.png)

###### 2.2.1.4.3 KD Trees #####


> **KD Tree:**多维(K Dimension)树是对BST **key的维度的泛化**(和B tree对同样维度**key的个数的泛化**不同)。最直观的例子是将1d range search扩展到2drange search。应用2D Tree对x和y坐标轮流插入和搜索，可以有效完成2维的如求某矩形包括的点和某点的最近点等问题。

{: .img_middle_hg}
![Kd Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/Kd Tree.png)

###### 2.2.1.4.4 Interval Search Trees #####

> **1d Interval Search Tree:**1维区间搜索是求给定区间有相交的区间。与**1d Range Search**(key是点，而这里是线段)不一样。


{: .img_middle_hg}
![Kd Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/1d interval search.png)

###### 2.2.1.4.5 Rectangle Intersection #####

> **2d orthogonal rectangle intersection:**2维平面求与给定矩形相交的矩形。

{: .img_middle_hg}
![Kd Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/2d orthogonal rectangle intersection.png)

###### 2.2.1.4.6 BSTs几何应用总结 #####

{: .img_middle_hg}
![Kd Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/BSTs Geometric Applicatoin Summary.png)

#### 2.2.2 Hash Table #####

##### 2.2.2.1 Hash Function #####

> Hash Function: computing array index(Hash Code) based on key.

哈希函数:

1. all class in java implements a method hashCode(), which retunrs 32-bit int. 
2. Default implementation is the memeory address.
3. Java customized some typical class such as Integer, Double, String, URL, Date...

{: .img_middle_hg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/hashCode implementation.png)

实现哈希表需要注意一下几点：

1. Equality Test, checking whether two keys are equal.

2. Collision Resolution, 当两个key产生同一个Hash Code时; 对于Collision Resolution, 通常两种方法, Separate Chaining和Linear Probing。

##### 2.2.2.2 单独链表法 #####

{: .img_middle_hg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/hash separate chaining.png)

{% highlight java linenos %}

public class SeparateChainingHashST<Key,Value>  {
    private int M = 97;                         // number of chains
    private Node[] st = (Node[]) new Object[M]; // array of chains
    
    class Node{
        private Object key;
        private Object value;
        private Node next;
        
        public Node(Key key, Value value, Node x){
            this.key = key;
            this.value = value;
            Node old = x;
            x = this;
            this.next = old;
        }
    }
    
    private int hash(Key key){
        return (key.hashCode() & 0x7fffffff) % M; //omit sign bit, and convert into array index
    }
    
    private Value get(Key key){
        int i = hash(key);
        for (Node x = st[i]; x!= null; x = x.next)
            if(key.equals(x.key)) return (Value)x.value;
        return null;
    }
    
    public void put(Key key, Value value){
        int i = hash(key);
        for (Node x = st[i]; x!=null; x= x.next)
            if(key.equals(x.key)) {
                x.value = value; return;
            }
        st[i] = new Node(key, value, st[i]);
    }
}
{% endhighlight %}

##### 2.2.2.3 线性探查法 #####

> **线性探查法:**找到hash(Key)对应的数组下标i，如果已经被占，则继续i+1，i+2...顺序找到空位为止。数组长度要大于Key个数，通常是两倍，然后resizing。

{: .img_middle_hg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/hash linear probing.png)



{% highlight java linenos %}
public class SeparateChainingHashST<Key,Value>  {
    
    private int M = 30000;                          
    private Value[]  vals = (Value[]) new Object[M];
    private Key[]    keys = (Key[])   new Object[M]; 
    
    private int hash(Key key){
        return (key.hashCode() & 0x7fffffff) % M; //omit sign bit, and convert into array index
    }
    
    private Value get(Key key){
        for (int i = hash(key); keys[i] != null; i = (i+1) % M)
            if(keys[i].equals(key))
                return vals[i];
        return null;
    }
    
    
    public void put(Key key, Value value){
        int i;
        for (i = hash(key); keys[i] != null; i = (i+1) % M)
            if(keys[i].equals(key)) break;
        keys[i] = key;
        vals[i] = value;
    }
}
{% endhighlight %}


##### 2.2.2.4 哈希背景 #####


{: .img_middle_hg}
![hash context](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/hash context.png)


## 3 Symbol Table Application ##

### 3.1 Set ###

{: .img_middle_hg}
![SET](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/SET.png)

### 3.2 Dictionary Client ###

{: .img_middle_hg}
![dictionary](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/dictionary.png)

### 3.3 Indexing Client ###

{: .img_middle_hg}
![dictionary](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/file indexing.png)

### 3.4 Sparse Vectors ###

{: .img_middle_hg}
![dictionary](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/sparse vectors.png)

## 4 Searching programming example: kdtree ##

用2dtree实现二维屏幕最近点和矩形框内所有点的搜索。

具体code见[这里](https://github.com/shunmian/-01-Algorithm-Princeton)。


## 5 总结 ##

{: .img_middle_hg}
![Chapter 3 Searching Summary](/assets/images/posts/01_Algorithm/2015-09-06_Algorithm(Part III)：Searching(二)：进阶搜索/Chapter 3 Searching Summary.png)


## 6 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);

- [Python数据结构时间复杂度](https://wiki.python.org/moin/TimeComplexity)





