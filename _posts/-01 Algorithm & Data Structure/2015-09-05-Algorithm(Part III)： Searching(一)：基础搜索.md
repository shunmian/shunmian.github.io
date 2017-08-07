---
layout: post
title: Algorithm(三)：Searching Part I：基础搜索 
categories: [-01 Algorithm]
tags: [Searching]
number: [-2.2.4]
fullview: false
shortinfo: Searching 搜索是现代计算机和互联网的基础。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}


## 1. 搜索算法API ##

{: .img_middle_lg}
![Sorting Algorithms](/assets/images/posts/01_Algorithm/2015-09-05_Algorithm(Part III)：Searching(一)：基础搜索/Searching API.png)

## 2. 搜索算法 ##

{% highlight java linenos %}

{% endhighlight %}
{: .img_middle_lg}
![Sorting Algorithms](/assets/images/posts/01_Algorithm/2015-09-05_Algorithm(Part III)：Searching(一)：基础搜索/searching performance0.png)

### 2.1 基础搜索 ###

#### 2.1.1 顺序搜索(无序链表)实现 ####

{% highlight java linenos %}
public class SequentialSearchST<Key, Value>{}
    private Node first;         // first node in the linked list
    private class Node{
        // linked-list node
        Key key;
        Value val;
        Node next;
    
        public Node(Key key, Value val, Node next){
            this.key  = key;
            this.val  = val;
            this.next = next;
        } 
    }

     public Value get(Key key){  // Search for key, return associated value.
        for (Node x = first; x != null; x = x.next)
           if (key.equals(x.key))
              return x.val;    // search hit
        return null;           // search miss
    }

     public void put(Key key, Value val){   // Search for key. Update value if found; grow table if new.
        for (Node x = first; x != null; x = x.next)
           if (key.equals(x.key)){  
            x.val = val; return;  }         // Search hit: update val.
        first = new Node(key, val, first);  // Search miss: add new node.
     }
}
{% endhighlight %}


Inserting N distinct keys into an initially empty linked-list symboltable uses ~N<sup>2</sup>>/2 compares.


#### 2.1.2 二分法搜索(有序数组)实现 ####

{% highlight java linenos %}
public class BinarySearchST<Key extends Comparable<Key>, Value> {
    private int N;
    private Key[] keys;
    private Value[] values;
        
    public BinarySearchST(int capacity){
        this.keys =(Key[]) new Comparable[capacity];
        this.values = (Value[]) new Object[capacity];
    }
    
    public int size(){
        return N;
    }
    
    public boolean isEmpty(){
        return size() == 0;
    }
    
    public int rank(Key key){   // get the rank of current key in a ordered array using binary search
        int lo = 0;
        int hi = N-1;
        while(lo <= hi){
            int mid = lo + (hi-lo)/2;
            int cmp = key.compareTo(keys[mid]);
            if(cmp < 0) hi = mid-1;
            else if(cmp > 0) lo = mid + 1;
            else return mid;
        }
        return lo;
    }
    
    public Value get(Key key){
        if(isEmpty()) return null;
        int i = rank(key);
        if(i<N && keys[i].compareTo(key) == 0) return values[i];
        else return null;
    }
    
    public void put(Key key, Value value){
        int i = rank(key);
        if(i<N && keys[i].compareTo(key) == 0) {values[i] = value; return;}
        for(int j = N; j > i; j--){
            keys[j] = keys[j-1];
            values[j] = values[j-1];
        }
        values[i] = value;
        keys[i] = key;
        N++;
    }
    
    public Iterable<Key> keys()  {
        Iterable<Key> iterable = Arrays.asList(keys);
        return iterable;
    }
}
{% endhighlight %}

binary search(by ordered array)Inserting a new key into an ordered array of size N uses ~ 2N array accesses in the worst case, so inserting N keys into an initially empty table uses ~ N 2 array accesses in the worst case.

小结: 2.1顺序搜索(无序链表)实现和2.2二分法搜索(有序数组)实现的peformance总结如下表:

{: .img_middle_lg}
![Sorting Algorithms](/assets/images/posts/01_Algorithm/2015-09-05_Algorithm(Part III)：Searching(一)：基础搜索/searching performance1.png)


我们能否可以实现某种算法使得insert和search都是O(logN)的时间复杂度？ 答案是YES！这要用到下面介绍的二叉树(binary search tree) by using array(O(logN)) search advantage and linked list quick insertion advantage.

#### 2.1.3 二叉搜索树实现 ####

binary search tree combines the flexibility of insertion in a linked list with the efficiency of search in an ordered array. Use two links per node(instead of the one link per node found in linked lists).

> <b>Binary Search Tree(BST)</b>: a binary tree where each node has a Comparabe key (and an associated value) and satisfies the restriction that the key in any node is larger than the keys in all nodes in that node's left subtree and smaller than the keys in all nodes in that node's right subtree.

{: .img_middle_lg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-05_Algorithm(Part III)：Searching(一)：基础搜索/binary tree vs binary search tree.png)

{% highlight java linenos %}
public class BST<Key extends Comparable<Key>,Value>  {
    
    Node root = null;
    
    class Node{
        Key key;
        Value val;
        int N;
        Node left;
        Node right;

        public Node(Key key, Value value, int N){
            this.key = key;
            this.val = value;
            this.N = N;
        }
    }
    
    public Value get(Key key){
        return get(root, key);
    }
    
    private Value get(Node x, Key key){
        if(x == null) return null;
        int cmp = x.key.compareTo(key);
        if(cmp < 0) return get(x.right, key);
        else if(cmp > 0) return get(x.left, key);
        else return x.val;
    }
    
    
    public void put(Key key, Value value){
        put(root, key, value);
    }
    
    private Node put(Node x, Key key, Value val){
        // 这部分递归非常巧妙，将新Node作为child node被相应parent连接起来的操作用递归返回值赋值的方式实现。这就必须得同时包括若Node x不为null时，递归返回值是x，来赋值的情况。
        if (x == null) return new Node(key, val, 1);
        int cmp = key.compareTo(x.key);
        if      (cmp < 0) x.left  = put(x.left,  key, val);
        else if (cmp > 0) x.right = put(x.right, key, val);
        else x.val = val;
        x.N = size(x.left) + size(x.right) + 1;
        return x;
    }
}
{% endhighlight %}

Search hits in a BST built from N random keys require ~ 2lnN compares on the average, correspondence to QuickSort partitioning. But the worst case is that it is not balanced which requires N compares。

{: .img_middle_lg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-05_Algorithm(Part III)：Searching(一)：基础搜索/binary search tree situation.png)




## 3 总结 ##

三种基本搜索的performance总结如下图。

{: .img_middle_lg}
![Binary Tree](/assets/images/posts/01_Algorithm/2015-09-05_Algorithm(Part III)：Searching(一)：基础搜索/searching performance2.png)

1. 顺序搜索(无序链表)：搜索是O(N)；插入虽然是O(1)，但是插入前需先搜索，若搜索不到则插入，否则update，因此实际插入是O(N)。

2. 二分法搜索(有序数组)：搜索是O(lgN)；插入是O(N)，因为插入前，需要将所有大于插入key的item都往后移动1个位置。二分法搜索(有序数组)比顺序搜索(无序链表)是一个提高。

3. 二叉搜索树：平均情况下搜索是O(lgN)，插入是O(lgN)。但是若树不平衡，比如按升序依次插入，则搜索是O(N)，插入式O(N)。

那么能否实现平衡的二叉搜索树呢？如果能，又如何实现呢？


## 4 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);





