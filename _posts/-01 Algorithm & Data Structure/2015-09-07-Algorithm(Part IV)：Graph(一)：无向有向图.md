---
layout: post
title: Algorithm(四)：Graph Part I：有向无向图
categories: [-01 Algorithm]
tags: [Graph]
number: [-2.2.4]
fullview: false
shortinfo: 在Part I 中，我们介绍了基本的数据结构(**并查集**，**队列** 和 **栈**)，随后又分模块重点介绍了常用的**排序**(**Sorting**)和**查找**（**Searching**）算法和对应的数据结构。在随后的Part II 里，将会着重介绍另外几大类常用的算法，包括**图**算法(**Graph**)，**字符串**算法 (**String**) 和算法周边的知识(**Context**)。本节作为Part II 的开端， 将会介绍Graph的基本知识和一些有趣的实际应用。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 图介绍 ##

在Part I 中，我们介绍了基本的数据结构(**并查集**，**队列** 和 **栈**)，随后又分模块重点介绍了常用的**排序**(**Sorting**)和**查找**（**Searching**）算法和对应的数据结构。在随后的Part II 里，将会着重介绍另外几大类常用的算法，包括**图**算法(**Graph**)，**字符串**算法 (**String**) 和算法周边的知识(**Context**)。本节作为Part II 的开端， 将会介绍Graph的基本知识和一些有趣的实际应用。
此外，在之后的课程中，将会把每周的作业进行解读，提供给感兴趣的同学练练手。这些编程题目深入浅出，把算法和应用紧密结合起来，非常推荐做一下。


## 2 图算法

### 2.1 Undirected Graph

**无向图**(**Undirected Graph**)，顾名思义，就是没有方向的边和若干点相互连接形成的拓扑结构。不像前面介绍的排序和查找，我们可以毫不费力地用数组(**Array**)和链表(**List**)将其存储到计算机中，对于图而言，由于连接的多样性，需要谨慎考虑存储在计算机中的方式。先看下Java中Graph类的API:
{% highlight java linenos %}
public class Graph{
                   Graph(int V)            // create a V-vertex graph with no edges
                   Graph(In in)            // read a graph from input stream in
               int V()                     // number of vertices
               int E()                     // number of edges
              void addEdge(int v, int w)   // add edge v-w to this graph
   Iterable<Integer> adj(int v)            // vertices adjacent to v
            String toString()              // string representation
}
{% endhighlight %}

#### 2.1.1 邻接链表实现 ####

对于Graph API，节点的存储比较好办，只需要申请大小为V的int型数组即可，如果每个节点有额外的信息，也只需要引入在Part I 中介绍的Symbol Table。 但是对于边的话，选择就比较多样了，首先直接想到的是**邻接矩阵**(**Adjacency Matrix**)，但问题是如果图的连接比较稀疏的话，那么矩阵将会变成稀疏矩阵，这对于存储空间是极大的浪费，因此在实际应用中，我们考虑使用**邻接链表**(**Adjacency List**)来实现。具体来说，申请一个大小为V的数组，数组的类型是Bag,就像一个袋子一样，不考虑其中元素的顺序，只关心元素是否在袋子里，这样，我们就能通过下标快速地索引到当前节点所有的邻居。以下是Java代码的实现。


{% highlight java linenos %}

public class UndirectedGraph {
  private final int V;                    // number of vertices
  private int E;                          // number of edges
  private Bag<Integer>[] adj;             // adjacency lists
  
  public UndirectedGraph(int V){
      this.V = V;                         
      this.E = 0;                         
      adj = (Bag<Integer>[]) new Bag[V];  // Create array of lists.
      for (int v = 0; v < V; v++)         // Initialize all lists
          adj[v] = new Bag<Integer>();    //   to empty.
   }
  
   public UndirectedGraph(In in){
       this(in.readInt());                // Read V and construct this graph.
       int E = in.readInt();              // Read E.
       for (int i = 0; i < E; i++){       
             int v = in.readInt();        // Read a vertex,
             int w = in.readInt();        // read another vertex,
             addEdge(v, w);               // and add edge connecting them.
       }
    }
  
   public int V()  {  return V;  }
   public int E()  {  return E;  }
  
   public void addEdge(int v, int w){
       adj[v].add(w);                     // Add w to v’s list.
       adj[w].add(v);                     // Add v to w’s list.
       E++;
   }
  
   public Iterable<Integer> adj(int v){ 
       return adj[v];
   }
}

{% endhighlight %}

下图是其他两种表示方法的复杂度分析，可见邻接矩阵虽然在查询两个节点是否连接上比较迅速，但是如果要遍历某个给定节点的所有边，复杂度却为O(V)


{: .img_middle_hg}
![representation](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/undirected graph data structure.png)


#### 2.1.2 无向图搜索 ####

##### 2.1.2.1 Depth-first Search #####

还记得特修斯斩杀米诺陶诺斯的故事吗，当时特修斯手持线团和宝剑，冲进迷宫，直接K.O.了迷宫怪牛，然后顺着线团出来了。从小听这个故事就非常好奇特修斯是怎么利用线团来防止迷路的，直到学了无向图中的**深度优先搜索算法**，我才恍然大悟。

问题重现：


{: .img_middle_mid}
![maze](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/maze.png)

<br/>

>假设特修斯现在在左下角，米诺陶诺斯在中央，特修斯手中有一根无限长的线团，那么他应该采取怎么样的策略才能顺利到达中央并且安全返回。

不难看出，特修斯可以一股脑一条道走到黑，直到碰到了死胡同，然后顺着线团回头，直到到达最近的交点处，选择另外一条没做过的路继续走到黑。另外如果发现脚下有之前留下的线团，那么立即掉头。大致过程如下所示。**问题的关键不在于暂时走错路，而在于如何保证不走两次同样的路(走过就不会再走)。**

{: .img_middle_lg}
![maze2](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/maze2.png)

机智的特修斯真是深谙算法，不过我也纳闷那个时候能找到那么长的线团吗？

下面介绍如何利用深度优先算法实现上述功能，不过在此之前先要阐明一下算法设计的一些理念，众所周知：

<p style="text-align: center;">数据结构 + 算法 = 程序 </p>

在设计图的相关算法的时候也是类似的道理，需要把具体的算法和底层的图结构**解耦**，这样代码维护起来就会非常高效。还有一种方法是将图算法作为图数据结构的实例方法，就像我们Part I大部分例子一样。但是由于图算法和图数据结构之间的联系比较弱，这样实现就会使得图数据结构非常臃肿，因此我们在这里将图算法和图数据结构解耦表示(**即OOP里，若对象的实例变量(数据结构)和实例方法(算法)联系弱，则最好将实例方法解耦单独表示**)。


在Graph类基础上，我们可以用Graph-Pcocessing来对Graph实例进行处理，这样把Graph和Graph-Processing的好处是可以将Graph(**数据结构**)和图处理(**算法**)解耦。这里我们首先介绍深度优先搜索遍历一个图来获取图中与vertex s连接的vertex v以及具体路径。

{: .img_middle_lg}
![Breadth-first Search](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/DFS Algorithm.png)

{% highlight java linenos %}
public class DepthFirstPaths{                   // using depthFirstSearch to find path to s in graph G
   private boolean[] marked;                    // has dfs() been called for this vertex?
   private int[] edgeTo;                        // last vertex on know path to this vertex
   private final int s;                         // source
   public DepthFirstSearchPaths(Graph G, int s){
      marked = new boolean[G.V()];
      edgeTo = new int[G.V()];
      this.s = s;
      dfs(G, s); 
   }
   private void dfs(Graph G, int v){
      marked[v] = true;
      for (int w : G.adj(v))
         if (!marked[w]) {
             edgetTo[w] = v;
             dfs(G, w);
         }
   }
   
   public boolean hasPathTo(int v){
       return marked[v];
   }
   
   public Iterable<Integer> pathTo(int v){
       if(!hasPathTo(v)) return null;
       Stack<Integer> path = new Stack<Integer>();
       for (int x = v; x != s; x = edgeTo[x])
           path.push(x);
       path.push(s);
       return path;
   }
}
{% endhighlight %}

有了上述的算法，我们就可以对给定的Graph和初始节点s，找到与s相连接的所有节点，一旦算法完毕，查询节点是否可达的时间是O(1)的复杂度，而对应的查询源节点s到达任意一点路径的时间正比于路径的长度。

这样，特修斯如果想知道如何从入口s到达迷宫中的任意一点，只需要调用一下`pathTo(int target)`方法就搞定，简直是屠牛神器。


##### 2.1.2.2 Breadth-first Search #####

与深度优先搜索不同，**广度优先搜索**(**BFS**)更加理智和贪婪，从数据结构的角度出发，广度优先搜索利用的是队列，而不是栈的模型来进行节点的选择。这么做背后的理念就是尽量地保守，不得已的时候才往下探索一层。因此，广度优先搜索有一个非常好的性质，就是能找到从某一源节点到其他节点**最短跳数**的路径。

这种性质使得其使用非常广泛，像好莱坞娱乐圈的**Kevin Bacon Number**以及学术圈的**Erdos Number**，计算的都是以某一节点为中心的最少跳数。下面简要说说Java的实现，与DFS非常类似。

{: .img_middle_lg}
![Breadth-first Search](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/BFS Algorithm.png)

{% highlight java linenos %}
public class BreadthFirstPaths{
   private boolean[] marked;                        // Is a shortest path to this vertex known?
   private int[] edgeTo;                            // last vertex on known path to this vertex
   private final int s;                             // source
   private int[] distTo;                            // distTo[v] = length of shortest s->v path

   public BreadthFirstPaths(Graph G, int s){
      marked = new boolean[G.V()];
      distTo = new int[G.V()];
      edgeTo = new int[G.V()];
      for (int v = 0; v < G.V(); v++)
        distTo[v] = INFINITY;
      this.s = s;
      bfs(G, s);
   }
   
   private void bfs(Graph G, int s){
      Queue<Integer> queue = new Queue<Integer>();
      marked[s] = true;                               // Mark the source
      distTo[s] = 0;
      queue.enqueue(s);                               // and put it on the queue.
      while (!queue.isEmpty()){
         int v = queue.dequeue();                     // Remove next vertex from the queue.
         for (int w : G.adj(v)){
            if (!marked[w]){                          // For every unmarked adjacent vertex,
                edgeTo[w] = v;                        // save last edge on a shortest path,
                distTo[w] = distTo[v] + 1;
                marked[w] = true;                     // mark it because path is known,
                queue.enqueue(w);                     // and add it to the queue.
            }
         }
      }
   }
   
   public boolean hasPathTo(int v){  
       return marked[v];  
   }

   public Iterable<Integer> pathTo(int v){
       if(!hasPathTo(v)) return null;
       Stack<Integer> path = new Stack<Integer>();
       for (int x = v; x != s; x = edgeTo[x])
           path.push(x);
       path.push(s);
       return path;
   }
}
{% endhighlight %}

for any vertex v reachable from s, BFS computes a shortest path from s to v(no path from s to v has fewer edges).


#### 2.1.4 Connected Component ####

其实经过了DFS和BFS处理的Graph, **连通分量**(**Connected Component**)的概念就呼之欲出了。因为既然我们已经找出了相互之间可达的节点，那么这些节点不就可以聚为一类吗？而每一类我们就称为**连通分量**。

剩下的问题是，如何找到所有的连通分量，因为如果Graph本身不完全连通，那么一次DFS或者BFS显然是不够的。就像如果怪牛本身就在二楼，特修斯无论怎么找也不可能找到，于是就需要在二楼再来一遍DFS。因此，我们需要对所有节点进行DFS处理，才能找到所有的连通分量。但这样可能带来复杂度的提升，后面我们将会介绍，由于有`marked`数组做信息记录，因此这个操作实际上非常快。

还记得本系列的第一篇文章介绍的**并查集**吗,有同学可能会问，既然Union的操作就是添加边的操作，那其实完全可以用并查集来实现连通分量的计算呀，为啥要用DFS和BFS。恩，是的，的确如此，但是，并查集的缺点在于不平衡，即便有很多手段让并查集生成的树尽量平衡，但是依旧不够快。下面，将会基于前文的DFS向大家介绍一种非常高效的连通分量查找算法。代码如下。

{% highlight java linenos %}

public class CC{

   private boolean[] marked;
   private int[] id;
   private int count;
   
   public CC(Graph G){
      marked = new boolean[G.V()];
      id = new int[G.V()];
      for (int s = 0; s < G.V(); s++)
         if (!marked[s]){
             dfs(G, s);
             count++; 
        }
   }
   
   private void dfs(Graph G, int v){
      marked[v] = true;
      id[v] = count;
      for (int w : G.adj(v))
         if (!marked[w])dfs(G, w);
   }
   
   public boolean connected(int v, int w){  
       return id[v] == id[w];  
   }
   
   public int id(int v) {  return id[v];  }
   public int count()   {  return count;  }
}
{% endhighlight %}

可见， CC类中扩展了DFS的方法，唯一不同的是遍历调用了`dfs`函数，和并查集比起来，最后生成的其实是一棵扁平化的并查集树，时间复杂度固然高效O(1)。

{: .img_middle_lg}
![Flood fill](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/
connected component.png)

需要注意的是CC和DFS的不同之处在于DFS是计算某个起点``s``是否有路径到图``graph``里任意点``v``，如果有，则给出path；而CC和DFS一样，除了将某个起点``s``换成图``graph``里的每个点``v``作为起点``s``。

寻找连通分量在实际中的应用也是非常广泛，例如在星空背景下的一颗颗星星，就可以看出一个个的连通分量，另外熟悉Photoshop的同学应该知道有一种flood fill技巧，如下图泰姬陵的天空一样进行局部的颜色修改，却不影响边界的颜色。这里的原理就是把颜色相近的像素看成连通，而边缘的阶跃看成不连通，这样只要找出连通分量，就能把边缘勾勒出来，非常的巧妙。

{: .img_middle_lg}
![Flood fill](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/flood fill.png)

#### 2.1.5 Challenges ####


{: .img_middle_hg}
![Flood fill](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/undirected graph challenge.png)

### 2.2 Directed Graph ###

**有向图**(**Directed Graph**)比起无向图来情况复杂很多，因为限制更加严格，由双向车道一下子变成单行线，衍生出来的问题也非常多，比如强连通分量，最短路径，拓扑排序等等。下面将会一一介绍这些算法和性质。


#### 2.2.1 邻接表实现 ####

在此之前，首先还是介绍一下有向图的存储和API。和无向图相似，有向图的存储也使用邻接链表的方式，不同的是这个时候的边有方向，因此原来无向图中同一条边需要存两次，在有向图中值需要存一次，其他的基本一致，java代码中类名由`Graph`变成`Digraph`即可。

{: .img_middle_lg}
![Flood fill](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/4.2.1 digraph API.png)

{% highlight java linenos %}
public class Digraph{
     private final int V;
     private int E;
     private Bag<Integer>[] adj;
     
     public Digraph(int V){           //create a V-vertex digraph with no edges
        this.V = V;
        this.E = 0;
        adj = (Bag<Integer>[]) new Bag[V];
        for (int v = 0; v < V; v++)
           adj[v] = new Bag<Integer>();
     }
     
     public int V()  {  return V;  }      //number of vertices
     public int E()  {  return E;  }      //number of edges
     
     public void addEdge(int v, int w){     //add edge v->w to this digraph

        adj[v].add(w);
        // adj[w].add[v] 无向图需要加两次，有向图只能按方向加一次。
        E++;
     }
     
     public Iterable<Integer> adj(int v){  
      return adj[v];  }
     
     public Digraph reverse(){          //reverse of this Digraph
        Digraph R = new Digraph(V);
        for (int v = 0; v < V; v++)
           for (int w : adj(v))
              R.addEdge(w, v);
        return R; }
}
{% endhighlight %}

另外，**深度优先搜索**和**广度优先搜索**同样适用于有向图，不同之处在于

1. 深度优先搜索侧重的是**可达性**分析，也就是说，从给定的一点出发，是否能经过一系列的路径到达指定的一点。在实际应用中这种情况大范围存在，比如**程序控制分析**(**program control-flow analysis**)和**标记-清理式垃圾回收器**(**mark-sweep garbage collector**)。大家可能会问，为什么在介绍无向图的时候没有那么多的应用，原因很简单，现实生活中的很多应用都是有向图。
2. 广度优先搜索侧重的是多源节点的广度搜索，做法和单元节点比较起来，只需要在初始化的时候把所有源节点压入队列即可，非常elegant。另外有向图的BFP在**网略爬虫**(**Web Crawler**)中有大量应用。以上问题和无向图类似，下面介绍若干有向图特有的性质和问题。

#### 2.2.2 有向图搜索 ####

有向图的搜索和无向图一样，包括DFS和BFS，实现也一样(无向图是特殊的有向图？)。

可以用有向图的BFS来搜索根网页的等级。为什么不用DFS？由于有些网页第一次被访问时，会产生新网页，因此会trap我们的执行，越走越深。

{: .img_middle_lg}
![Topological sort](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/4.2.2 digraph search_web crawler.png)


#### 2.2.3 Topological Sort ####

**拓扑序(Topological Sort)**是有向图中特殊的一种性质，并且要求进一步提高，不仅要求是有向，而且必须**没有环**，这就是传说中的**DAG(Directed Acyclic Graph)有向无环图**。这里没有环的意思是从任意一点找不到任何路径可以回到自身。这个要求相当严格，可以想象，如果满足以上性质，那么整个DAG可以拉长成一条线，有严格的先后顺序，这就是**拓扑序**，就像下图所示:



{: .img_middle_lg}
![Topological sort](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/Topological sort2.png)

那么问题来了，既然一个DAG必然存在拓扑序，那么怎么设计算法高效的找出整个拓扑序呢。令人吃惊的是，寻找拓扑序的算法异常的简单，只需要在DAG上跑若干遍DFS算法，然后把访问的节点后序压栈(Post)，最后所有跑完所有DFS后，依次弹栈的顺序就是拓扑序的顺序。


1. Run depth-first search
2. Return vertices in reverse postorder

{% highlight java linenos %}
public class DepthFirstOrder{
  
   private boolean[] marked;
   private Stack<Integer> reversePost;  // vertices in reverse postorder
   
   public DepthFirstOrder(Digraph G){
      reversePost   = new Stack<Integer>();
      marked  = new boolean[G.V()];
      for (int v = 0; v < G.V(); v++)
         if (!marked[v]) dfs(G, v);
   }
   
   private void dfs(Digraph G, int v){
      marked[v] = true;
      for (int w : G.adj(v))
         if (!marked[w])
            dfs(G, w);
      reversePost.push(v);
   }
   
   public Iterable<Integer> reversePost(){return reversePost;}
}
{% endhighlight %}

还不服？下面证明一下，考虑存在一条从v到w的边，即*v->w*，根据上述算法，研究当调用`dfs(v)`的一瞬间发生了什么:

1. `dfs(w)`已经被调用并且返回，这种情况发生在除了v之外还有其他的边指向w，如*u->w*，并且调用`dfs(u)`完毕，已经返回，则必然此时`dfs(w)`也已经返回。在这种情况下，w在v之前被压入栈中，最后弹栈时必然v在w前面，符合实际的v->w。
2. `dfs(w)`还没有被调用，容易知道这种情况下w将会在调用`dfs(v)`后递归调用到`dfs(w)`，最后也是当`dfs(w)`返回后`dfs(v)`才能返回。这种情况下w依旧在v之前被压入栈中。
3. `dfs(w)`被调用了，，但是还没有返回，这种情况不可能发生，因为如果上述情况发生，就意味着`dfs(w)`在等待`dfs(v)`的返回，即存在一条边*w->v*,而这样，v和w酒构成了一个环，违反了DAG的设定。

综上所述，算法正确，只要存在*v->w*，那么出栈顺序中v也一定在w前面。

是不是已经晕了，没事儿，只要记住结论:

> 对DAG图上所有未标记的节点跑DFS算法，并且采用ReversePost方式压栈，最后一次出栈的顺序就是拓扑序。

{: .img_middle_hg}
![Connectivity](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/4.2.3 topological order.png)





#### 2.2.4 Cycle Detection ####

DFS的用处远不止如此，另一个重要应用就是用于检测有向图中的环，换句话说，一个有向图，要么是DAG，要么就存储在环，二选一。

这里的原理就是对DFS搜索路径上的所有节点进行标记，若果在某一个的递归中，发了援救标记的节点，说明存在路径到达了自身，这个就像贪吃蛇一样碰到了自己的身子，接下来的就是记录环的情况，然后函数返回。关键代码如下。

{% highlight java linenos %}
private void dfs(Digraph G, int v){  
    onStack[v] = true;
    marked[v] = true;
    for(int w:G.adj(v)){
        //已经有环了，洗洗睡吧
        if(cycle != null) return;
        //未访问过则继续往下探索
        else if(!marked[w]){
            edgeTo[w] = v;
            dfs(G,w);
        //访问过并且处于当前DFS路径，有环！
        }else if onStack[w]{
            cycle = new Stack<Integer>();
            //记录环的情况
            for(int x=v;x!=w;x=edgeTo[x])
                cycle.push(x);
            cycle.push(x);
            cycle.push(v);
        }
    }
    //当前节点搜索完毕，换一条路径
    onStack[v] =false;
}
{% endhighlight %}



#### 2.2.5 Strong Components ####

有向图的**强连通分量**(**Strong Components**)对应于无向图的连通分量，也具有相等关系的性质，具体来说，满足一下3个特性:

1. v is strongly connected to V（自反性）；
2. if v is strongly connected to w, then w is strongly connected to v（对称性）；
3. if v is strongly connecte to w, and w to x, then v is strongly connected to x（传递性）.

而前面介绍的有向图的**可达性**，并不满足对称性。Java中强连通分量的API接口和无向图的连通分量一样:

{% highlight java linenos %}
// components in undirected graph
public int connected(int v, int w){  
    return cc[v] == cc[w];
}

// strong components in directed graph
public int stronglyConnected(int v, int w){  
    return scc[v] == scc[w];
}
{% endhighlight %}

可见一旦算法结束，查询连通分量的效率是妥妥的常数时间，那么，究竟该如何计算一个有向图的强连通分量呢?

有意思的是，历史上搜索强连通分量的算法发展历经坎坷，下面介绍一种1980s年的**Kosaraju-Sharir算法**，该算法看起来非常简单，正确性的证明却非常需要技巧，算法一共分为两步:

1. 计算对称图中Kernel DAG的拓扑顺序(compute reverse postorder in G<sup>R</sup>);
2. 根据拓扑序一次对各节点调用DFS,没调用一次DFS，生成一个Strong component。

以上算法非常巧妙，结合了拓扑序和DFS，而计算拓扑序实际上也是DFS,因此KS算法可以总结为**两轮DFS**。具体代码如下，和前面无向图计算连通分量非常相似。



{: .img_middle_mid}
![Connectivity](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/4.2.4 strong connected component.png)


{% highlight java linenos %}
public class KosarajuSharirSCC{
  
  private boolean[] marked;     //reached vertices
  private int[] id;             //component identifiers
  private int count;            //number of strong components

  public KosarajuSharirSCC(Digraph G){
    marked = new boolean[G.V()];
    id = new int[G.V()];
    DepthFirstOrder order = new DepthFirstOrder(G.reverse());
    for (int s : order.reversePost())
      if (!marked[s]){  
        dfs(G, s); count++;
      }
  }
  
    private void dfs(Digraph G, int v){
       marked[v] = true;
       id[v] = count;
       for (int w : G.adj(v))
          if (!marked[w])
              dfs(G, w);
    }
    
    public boolean stronglyConnected(int v, int w){return id[v] == id[w];}
    public int id(int v){return id[v];}
    public int count(){return count;}
}
{% endhighlight %}

和无向图的CC不同的是SCC用reverseG的拓扑序来进行DFS。
这里调用了前面Topological Sort的`DepthFirstOrder`类，然后修改了一下类名，就形成了新的`KosarjuShairSCC`类。关于正确性的证明，参考一下前面拓扑序的证明就可以感受一下KS算法证明的复杂性。大家需要知道的一点是，原始图G和对称图G<sup>R</sup>的连通分量完全一致，因此对称图中利用后续压栈的方法能得到不同连通分量之间的拓扑顺序。然后为了从拓扑序的末端开始往上游延伸，故第一步选择在对称图进行Kernel DAG的拓扑序查找。因此，强连通分量算法最牛掰的地方是能把任意有向图转化为**Kernel DAG**图，许多算法都要基于此进行，一般都是在不同的连通分量上进行算法设计，最终把各连通分量的结果合并即可。

至此，关于有向图，我们介绍了**可达性分析(DFS)**，**拓扑排序(DFS)**，**有向环检测(DFS)**，和**强连通分量(2*DFS)**。可见随处都是DFS，我们是彻彻底底地和DFS杠上了。相比之下，BFS的应用主要在**最短跳数**的求解上，希望以上的概括能给大家理解DFS和BFS一个比较好的直观感受。



## 3 Programming assignment:WordNet ##

本次的编程作业一如既往的有意思，核心任务主要是：

>在WordNet上面构建一个有向图，要求计算出任意给定两个点的最短距离，如果没有直接可达的路径，则计算出距离最近的共同祖先节点的距离和，否则返回-1。

{: .img_middle_lg}
![wordnet](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/wordnet.png)

举个例子详细说明一下题目的要求，比如上述的一个**有向无环图(DAG)**，中间的**miracle**和最右边的**group_action**，有共同的祖先**event**，故他们的最短距离为2，同理可以计算出任意两个节点之间的最短距离。重写BFS，需要对BFS的算法有较深入的理解：

1. 包括为什么使用队列而不是栈（队列可以保证逐层遍历）；
2. 如果v和x在同一层，并且同时有v->w和x->y-w,那么w是在v和x的下一层还是下两层(下一层，因为逐层遍历，w作为v的邻居先被压入队列，marked[w]已经true，通过x->y->w已经访问不到w)；
3. “多源节点的广度搜索，做法和单元节点比较起来，只需要在初始化的时候把所有源节点压入队列即可，非常elegant”。这句话如何理解(可以将多源节点想象成虚拟单节点的邻居，这就退化成了单节点的BFS问题，而单节点的BFS问题第一步就是将多源节点压入队列)。

因此，概括一下对于DAG的情况下最短距离的思路：

1. 将初始的集合(或者单节点)分别压入两个队列vQueue和wQueue；
2. 利用这两个队列分别同时进行广度优先搜索，也就是说，两个队列的搜索深度同步递增；
3. 循环2，知道其中一组遇到对方标记过的节点，算法终止，这个时候查看对方标记的节点的深度值depthW，再加上自己搜索到此的深度值depthV，得到深度值总和，就是最短距离。

注意一下，上面的算法是针对**DAG**的，也就是说没有环，因此我们在第一次遇到之后就可以终止算法，但是，这题的输入是**不限于DAG**的。

题目要求实现三个类：

1. SAP.java;
2. WordNet.java;
3. Outcast.java;

题目假定输入的有向图是任意的有向图，也就意味着可能是**有环的**，**不连通的**，甚至有**多个根节点**，这里的根节点其实就是WordNet里面的叶子节点，理论上所有的WordNet最终都指向一个节点**Entity**。这里的核心两个算法如下：

1. 如何高效计算任意一个有向图的任意两个结合(或者一对节点)间最短距离；
2. 如何能检测出一个有向图是**有环的**，**不连通的**，甚至有**多个根节点**。

第一个的思路也非常简单，是基于上述基本版本的改进版。既然可能存在环，那么我们就不能一遇到对方的标记的节点就终止算法，而是应该把这个时候计算得到的深度总和记录下来，形成一个record，形式为<vertextID,vertexDepth>。这样，当两个BFS都搜索完毕时，对比所有的record，找到一个depth最短的，返回即可。

这样做的确能保证算法的正确性，但是仔细一想，假设当前已经找到了一个被标记的节点深度总和为5，而当前两个BFS的搜索迭代到了第8层，那么我们还有迭代下去的必要吗？答案很显然，没有必要，因为哪怕本次迭代又找到了一个被标记的节点，充其量最短距离也不会小于8。这启发我们在BFS的时候可以进行early stopping。思路是在本次迭代搜索开始前，先对比搜索深度是否已经超过了目前已知的最短deDepth记录，如果超过那么直接结束了迭代。

第二个问题可以拆分为三个小问题:

1. 判断有向图有环的方法是在进行DFS的时候维护一条当前搜索链，当发现某一次搜索重新探测到了已经被标记为当前搜索链的节点，那么必然存在一个环。
2. 判断是否连通最简单的方法就是转换为无向图然后进行BFS或者DFS，看最后的连通分量的数量。
3. 判断是否有多个WordNet根节点只需要统计每个节点的**出度(Outdegree)**，任何出度为0的丢失WordNet根节点。

克服了以上两个核心问题，剩下的就是慢慢抠细节了，详细的代码，Specification和CheckLisk及test file在[这里](https://github.com/shunmian/Algorithm-II/tree/master/Week1_WordNet)。

最后贴一个跑分表，竟然超过了100分，貌似时间比较快，有奖励...

{: .img_middle_lg}
![assessment](/assets/images/posts/01_Algorithm/2015-09-07_Algorithm(Part IV)：Graph(一)：无向有向图/assessment.png)



## 4 总结 ##





## 5 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);

- 本文转自[[ALGORITHMS] GRAPHS (I)](http://aaronxic.com/algorithms-graph-i/#.Vxy4lKNcSko);



