---
layout: post
title: Algorithm(五)：String Part II：Searching
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

## 1 Prefix String Search: Tries ##

> **Tries**: 又称**前缀树**或字典树，是一种有序树，用于保存关联数组，其中的键通常是字符串。与二叉查找树不同，键不是直接保存在节点中，而是由节点在树中的位置决定。一个节点的所有子孙都有相同的前缀，也就是这个节点对应的字符串，而根节点对应空字符串。一般情况下，不是所有的节点都有对应的值，只有叶子节点和部分内部节点所对应的键才有相关的值。

### 1.1 R-way Tries ###

> **R-way Tries**: **R元树***是Tries的一种具体实现形式。每一个node：有1个node数组，指向R(Radix)个结点指针(指向下一个结点或者null)；以及value(有值或null)。

{: .img_middle_hg}
![Counting Sorting](/assets/images/posts/01_Algorithm/2015-09-10_Algorithm(Part V)：String(二)：Searching/1.1 R-Way Search.png)

{% highlight java linenos %}

public class TrieST<Value>{
	
	private static final int R = 256;
	private Node root = new Node();
	private static class Node {
		private Object value;
		private Node[] next = new Node[R];
	}
	
	public void put(String key, Value val){ 
		root = put(root, key, val, 0); 
	}
	private Node put(Node x, String key, Value val, int d){
		if (x == null) x = new Node();
		if (d == key.length()) { x.value = val; return x; }
		char c = key.charAt(d);
		x.next[c] = put(x.next[c], key, val, d+1);
		return x;
	}
	
	public boolean contains(String key){ 
		return get(key) != null; 
	}

	 public Value get(String key){
		 Node x = get(root, key, 0);
		 if (x == null) return null;
		 return (Value) x.value;
	 }
	 private Node get(Node x, String key, int d){
		 if (x == null) return null;
		 if (d == key.length()) return x;
		 char c = key.charAt(d);
		 return get(x.next[c], key, d+1);
	 }
}
{% endhighlight %}



### 1.2 Ternary Search Tries ###

> **Ternary Search Tries**: **三元搜索树**是**前缀树**的另一种一种实现，树的各个节点之间的结构类似二叉搜索树。三元搜索数比R元树更节省空间(每个节点存储了一个字符、一个值对象或值指针以及三个指向子节点的指针。这三个字节点常被称为等位字节点、低位字节点和高位字节点。)，但是牺牲了部分查找速度。三元搜索树常用于实现拼写检查和自动完成功能。


{: .img_middle_hg}
![Ternary Search Trie](/assets/images/posts/01_Algorithm/2015-09-10_Algorithm(Part V)：String(二)：Searching/1.2.1 Ternary Search Trie.png)

{% highlight java linenos %}

public class TST<Value>{
	
	private Node root;
	private class Node{
		 private Value val;
		 private char c;
		 private Node left, mid, right;
	}
 
	public void put(String key, Value val){ 
		root = put(root, key, val, 0); 
	}
 	private Node put(Node x, String key, Value val, int d){
 		char c = key.charAt(d);
 		if (x == null) { x = new Node(); x.c = c; }
 		if (c < x.c) x.left = put(x.left, key, val, d);
 		else if (c > x.c) x.right = put(x.right, key, val, d);
 		else if (d < key.length() - 1) x.mid = put(x.mid, key, val, d+1);
 		else x.val = val;
 		return x;
 	}
 	
 	public boolean contains(String key){ 
 		return get(key) != null; 
 	}

 	public Value get(String key){
 		Node x = get(root, key, 0);
 		if (x == null) return null;
 		return x.val;
 	}
 	private Node get(Node x, String key, int d){
 		if (x == null) return null;
 		char c = key.charAt(d);
 		if (c < x.c) return get(x.left, key, d);
 		else if (c > x.c) return get(x.right, key, d);
 		else if (d < key.length() - 1) return get(x.mid, key, d+1);
 		else return x;
 	}
 }
{% endhighlight %}


{: .img_middle_hg}
![Ternary Search Trie vs Trie & Hash](/assets/images/posts/01_Algorithm/2015-09-10_Algorithm(Part V)：String(二)：Searching/1.2.2 Ternary Search Trie vs Trie & Hash.png)

### 1.3 Character-Based Operations ###


{: .img_middle_hg}
![Ternary Search Trie vs Trie & Hash](/assets/images/posts/01_Algorithm/2015-09-10_Algorithm(Part V)：String(二)：Searching/1.3.1 Character-Based Operation_API.png)

{: .img_middle_hg}
![Ternary Search Trie vs Trie & Hash](/assets/images/posts/01_Algorithm/2015-09-10_Algorithm(Part V)：String(二)：Searching/1.3.2 Character-Based Operation_Implementation.png)


### 1.4 Tries Summary ###

{: .img_middle_mid}
![Ternary Search Trie vs Trie & Hash](/assets/images/posts/01_Algorithm/2015-09-10_Algorithm(Part V)：String(二)：Searching/1.4 Tries Summary.png)

## 2 Substring Search ##

### 2.1 Knuth-Morris-Pratt ###

### 2.2 Boyer-Moore ###

### 2.3 Rabin-Karp ###


## 3 正则表达式 ##

### 3.1 Regular Expressions ###

### 3.2 REs and NFAs ###

### 3.3 NFA simulation ###

### 3.4 NFA construction ###

### 3.5 Application ###



## 4 总结 ##



## 5 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);





