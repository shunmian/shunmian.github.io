---
layout: post
title: Algorithm(五)：String Part III：数据压缩
categories: [-01 Algorithm]
tags: [Graph]
number: [-2.2.4]
fullview: false
shortinfo: 图是重要的一种数据结构，能巧妙优雅地解决许多其他数据结构和算法不能解决的问题。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}




## 1 数据压缩 ##

### 1.1 Introduction ###

> **数据压缩有两个目的**: 空间上，节省数据存储的空间；时间上，节省数据传输的时间。

我们日常接触到的数字图片，声音，视频以及各种其它数据，其中包含了大量冗余的信息：文本里某些字母出现的频率远大于其它字母；bitmap包含了大量同样的相连的像素；其它例如声音及视频有大量重复的pattern等。

关于压缩，有一些基本定理需要我们理解。

> 上限：没有一种通用的算法可以压缩**所有**的bit流(10100101...,数据的表现形式)。因为如果有，不断压缩上一次压缩的结果，最终我们会获得0bit，而将0bit解码成原来的bit流是不可能的，需要额外的信息。

> 不确定性：通过程序产生需要的数据这种策略是不确定的，因为这种策略无法移植或提高。

{: .img_middle_lg}
![Counting Sorting](/assets/images/posts/01_Algorithm/2015-09-11_Algorithm(Part V)：String(三)：数据压缩/1.1.1 Undecidability.png)

例如上面这幅图我们可以用下面这段简短的程序产生，但是不可能基于这种策略来解决其他bitstream，而且我们也找不到基于这种策略的哪种算法是最好的。

### 1.2 Run-length Coding ###

{: .img_middle_hg}
![1.2 Run-length encoding](/assets/images/posts/01_Algorithm/2015-09-11_Algorithm(Part V)：String(三)：数据压缩/1.2 Run-length encoding.png)

### 1.3 Huffman Compression ###

{: .img_middle_lg}
![Counting Sorting](/assets/images/posts/01_Algorithm/2015-09-11_Algorithm(Part V)：String(三)：数据压缩/1.3 Huffman Coding.png)

{% highlight java linenos %}
import edu.princeton.cs.algs4.BinaryStdIn;
import edu.princeton.cs.algs4.BinaryStdOut;

public class HuffmanCoding {
  
  private static class Node implements Comparable<Node>{
    private final char ch;  // used only for leaf nodes
    private final int freq; // used only for compress
    private final Node left, right;

    public Node(char ch, int freq, Node left, Node right){
      this.ch = ch;
      this.freq = freq;
      this.left = left;
      this.right = right;
    }

    public boolean isLeaf(){ 
      return left == null && right == null; 
    }

    public int compareTo(Node that){ 
      return this.freq - that.freq; 
    }
  }

  public void expand(){
    Node root = readTrie();
    int N = BinaryStdIn.readInt();

    for (int i = 0; i < N; i++){
      Node x = root;
      while (!x.isLeaf()){
        if (!BinaryStdIn.readBoolean())
          x = x.left;
        else
          x = x.right;
      }
      BinaryStdOut.write(x.ch, 8);
    }
   
    BinaryStdOut.close();
  }
  
  private static Node readTrie(){
    if (BinaryStdIn.readBoolean()){
      char c = BinaryStdIn.readChar(8);
      return new Node(c, 0, null, null);
    }
    Node x = readTrie();
    Node y = readTrie();
    return new Node('\0', 0, x, y);
  } 
  
  private static void writeTrie(Node x){
    if (x.isLeaf()){
      BinaryStdOut.write(true);
      BinaryStdOut.write(x.ch, 8);
     return;
    }
    BinaryStdOut.write(false);
    writeTrie(x.left);
    writeTrie(x.right);
  }
}

{% endhighlight %}


### 1.4 LZW Compression ###

{: .img_middle_hg}
![Counting Sorting](/assets/images/posts/01_Algorithm/2015-09-11_Algorithm(Part V)：String(三)：数据压缩/1.4 LZW Coding.png)


{% highlight java linenos %}

{% endhighlight %}

## 4 总结 ##

本文介绍了6种经典的搜索方法来实现Symbol Table: 

1. 顺序搜索(无序链表)实现
2. 二分法搜索(有序数组)实现
3. 二叉搜索树实现
4. 平衡二叉搜索树(红黑树)实现
5. 哈希表实现(单独链表法 & 线性探查法)

其中哈希表的搜索在最坏的情况下是~lg(N)，在最好的情况下是~O(1)，是5个算法中最优的算法。但是需要注意hashCode冲突的情况。



## 5 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);





