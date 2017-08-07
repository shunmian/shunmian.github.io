---
layout: post
title: Algorithm(四)：Graph Part II：最小生成树和最短路径
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

## 1 图介绍 ##

前文介绍了基本的无向图和有向图性质和一些算法应用，不难发现我们侧重的是**拓扑连接**，只是研究点和边的组合情况，并没有考虑边与边之间的不同。在本节中，我们将会把图问题进一步细化，引入**边权值(edge weight)**的概念。我们将会发现，一旦引入了边的权值，那么将会出现非常有意思的问题，比如：

1. 最小生成树(Minimum Spanning Tree);
2. 最短路径(Shortest Path);
3. 最小割(Minimum Cut);
4. 最大流(Maximum Flow)。


## 2 图算法

### 2.3 最小生成树  ###

><b>Minimum Spanning Tree(最小生成树)</b>:a minimum <b>spanning(生成)</b>(includes all vertices) <b>tree(树)</b>(connected acyclic) is a spanning tree whose weight (the sum of the weights of its edges) is no larger than the weight of any other spanning tree.


Given any cut in an edge- weighted graph, the crossing edge of minimum weight is in the MST of the graph.

#### 2.3.1 Greedy Algorithm  ####

<blockquote>
<b>Cut(割)</b>: A cut of a graph is a partition of its vertices into two nonempty disjoint sets. <b>A crossing edge(交叉边)</b> of a cut is an edge that connects a vertex in one set with a vertex in the other.<br/>
<b>Cut property</b>: Given any cut in an edge- weighted graph, the crossing edge of minimum weight is in the MST of the graph.
</blockquote>

那么如何实现**Greedy Algorithm**的以下两个关键步骤呢？

1. 如何选择**cut**;

2. 如何在**crossing edge**里选最小边。

对于以上两步的具体实现的不同产生了不同的算法：

1. Kruskal算法;

2. Prim算法;

3. Borüvka算法。

下面我们主要介绍Kruskal算法和Prim算法。

#### 2.3.2 API  ####

首先定义Edge，包括连个顶点和一个weight。
{% highlight java linenos %}
public class Edge implements Comparable<Edge>{
  
   private final int v;
   private final int w;
   private final double weight;
   
   public Edge(int v, int w, double weight){
      this.v = v;
      this.w = w;
      this.weight = weight;
   }

   public double weight(){  return weight;  }       //edge weight     
   public int either(){  return v;  }               //one vertex
   public int other(int vertex){                    //the other vertex
       if      (vertex == v) return w;
      else if (vertex == w) return v;
      else throw new RuntimeException("Inconsistent edge");
   }
   
   public int compareTo(Edge that){
      if      (this.weight() < that.weight()) return -1;
      else if (this.weight() > that.weight()) return +1;
      else                                    return  0;
   }
   
   public String toString(){
     return String.format("%d-%d %.5f", v, w, weight);  
   }
}
{% endhighlight %}

其次实现EdgeWeightedGraph，用邻接表实现，同一个edge出现两次引用，但是只有一个Edge object.


{% highlight java linenos %}
public class EdgeWeightedGraph{
  
   private final int V;             //number of vertices
   private int E;                   //number of edges
   private Bag<Edge>[] adj;         //adjacency lists
   
   public EdgeWeightedGraph(int V){
      this.V = V;
      this.E = 0;
      adj = (Bag<Edge>[]) new Bag[V];
      for (int v = 0; v < V; v++)
         adj[v] = new Bag<Edge>();
   }
   
   public int V() {  return V;  }       
   public int E() {  return E;  }       
   
   public void addEdge(Edge e){         
      int v = e.either(), w = e.other(v);
      adj[v].add(e);
      adj[w].add(e);
      E++;
   }
   public Iterable<Edge> adj(int v){ return adj[v];  }
   
   public Iterable<Edge> edges(){
     Bag<Edge> b = new Bag<Edge>();
       for (int v = 0; v < V; v++)
          for (Edge e : adj[v])
             if (e.other(v) > v) b.add(e);
  return b;
   }

}
{% endhighlight %}


最后是MST的API，其实现算法在2.3.3和2.3.4介绍。
{% highlight java linenos %}
public class MST {
  public MST(EdgeWeightedGraph G);  //constructor
  Iterable<Edge> edges();       //all of the MST edges
  double weight()           //weight of MST
}
{% endhighlight %}

EdgeWeightedGraph和Graph的区别在于前者隐式表示Edge(通过int v，int w)，而后者显示创建了Edge类(int v, int w, int weight)。

{: .img_middle_hg}
![Kruskal](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/4.3.3 MST API.png)

#### 2.3.3 Kruskal MST 算法  ####

<blockquote>
<b>Kruskal MST Algorithm</b>: 
<ul>
<li>considerting edges in ascending order of weight.<br/></li>
<li>Add next edge to trees unless doing so would create acyclic(using union-find)</li>
</ul>
</blockquote>



{: .img_middle_lg}
![Kruskal](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/Kruskal.png)

{% highlight java linenos %}
public class KruskalMST{
  
   private Queue<Edge> mst;
   
   public KruskalMST(EdgeWeightedGraph G){
      mst = new Queue<Edge>();
      MinPQ<Edge> pq = new MinPQ<Edge>();
      for (Edge e : G.edges())
         pq.insert(e);
      UF uf = new UF(G.V());
      
      while (!pq.isEmpty() && mst.size() < G.V()-1){
         Edge e = pq.delMin();                        // Get min weight edge on pq
         int v = e.either(), w = e.other(v);          // and its vertices.
         if (uf.connected(v, w)) continue;            // Ignore ineligible edges.
         uf.union(v, w);                              // Merge components.
         mst.enqueue(e);                              // Add edge to mst.
      }
   }
   
   public Iterable<Edge> edges(){ return mst;  }
   
   public double weight();   
} 
{% endhighlight %}

#### 2.3.4 Prim MST 算法  ####

><b>Prim's MST Algorithm</b>: Start with any vertex as a single vertex tree; then add V-1 edges to it, always taking next (coloring black) the minimum weight edge that connects a vertex on the tree to a vertex not yet on the tree (a crossing edge for the cut de ned by tree vertices). Two versions, lazy & eager.

Lazy Version.


{: .img_middle_lg}
![Prim lazy](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/prim lazy.png)

{% highlight java linenos %}
public class LazyPrimMST{
  
   private boolean[] marked;                        // MST vertices
   private Queue<Edge> mst;                         // MST edges
   private MinPQ<Edge> pq;                          // crossing (and ineligible) edges

   public LazyPrimMST(EdgeWeightedGraph G){
     pq = new MinPQ<Edge>();
     marked = new boolean[G.V()];
     mst = new Queue<Edge>();
     visit(G, 0);                                   // assumes G is connected
     
     while (!pq.isEmpty()){
       Edge e = pq.delMin();                        // Get lowest-weight
       int v = e.either(), w = e.other(v);          // edge from pq.
       if (marked[v] && marked[w]) continue;        // Skip if ineligible.
          mst.enqueue(e);                           //Add edge to tree.
          if (!marked[v]) visit(G, v);              //Add vertex to tree
          if (!marked[w]) visit(G, w);              // (either v or w).
     }
   }


   private void visit(EdgeWeightedGraph G, int v){  // Mark v and add to pq all edges from v to unmarked vertices.
      marked[v] = true;
      for (Edge e : G.adj(v))
         if (!marked[e.other(v)]) pq.insert(e);
   }
   
   public Iterable<Edge> edges(){return mst;}
   
   public double weight()
}
{% endhighlight %}

Eager Version.

{: .img_middle_lg}
![Prim eager](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/prim eager.png)

{% highlight java linenos %}
public class EagerPrimMST{
 
  private Edge[] edgeTo;                            // shortest edge from tree vertex
  private double[] distTo;                          // distTo[w] = edgeTo[w].weight()
  private boolean[] marked;                         // true if v on tree
  private IndexMinPQ<Double> pq;                    // eligible crossing edges

  public EagerPrimMST(EdgeWeightedGraph G){
    edgeTo = new Edge[G.V()];
    distTo = new double[G.V()];
    marked = new boolean[G.V()];
    for (int v = 0; v < G.V(); v++)
      distTo[v] = Double.POSITIVE_INFINITY;
    pq = new IndexMinPQ<Double>(G.V());
    distTo[0] = 0.0;
    pq.insert(0, 0.0);                              // Initialize pq with 0, weight 0.
    while (!pq.isEmpty())
      visit(G, pq.delMin());                        // Add closest vertex to tree.
  }
  
  private void visit(EdgeWeightedGraph G, int v){   // Add v to tree; update data structures.
    marked[v] = true;
    for (Edge e : G.adj(v)){
      int w = e.other(v);
      if (marked[w]) continue;                      //v-w is ineligible.
      if (e.weight() < distTo[w]){                  // Edge e is new best connection from tree to w.
        edgeTo[w] = e;
        distTo[w] = e.weight();
        if (pq.contains(w)) pq.changeKey(w, distTo[w]);
        else                pq.insert(w, distTo[w]);
      }
    }
  }
  
  public Iterable<Edge> edges();
  
  public double weight();
}

{% endhighlight %}


### 2.4 最短路径 ###

><b>Shortest Path Tree</b>: Given an edge-weighted digraph and a designated source vertex s, a shortest-paths tree for vertex s is a subgraph containing s and all the vertices reachable from s that forms a directed tree rooted at s such that every tree path is shortest path in the digraph.

为了实现ShortestPathTree，我们必须有DirectedEdge， EdgeWeightedDigraph

**DirectedEdge**

{% highlight java linenos %}
public class DirectedEdge {

  private final int v; // edge tail
  private final int w; // edge head
  private final double weight; // edge weight

  public DirectedEdge(int v, int w, double weight) {
    this.v = v;
    this.w = w;
    this.weight = weight;
  }

  public double weight() {
    return weight;
  }

  public int from() {
    return v;
  }

  public int to() {
    return w;
  }

  public String toString() {
    return String.format("%d->%d %.2f", v, w, weight);
  }
}
{% endhighlight %}

**EdgeWeightedDigraph**

{% highlight java linenos %}
public class EdgeWeightedDigraph {
  private final int V;        // number of vertices
  private int E;            // number of edges
  private Bag<DirectedEdge>[] adj;  // adjacency lists

  public EdgeWeightedDigraph(int V) {
    this.V = V;
    this.E = 0;
    adj = (Bag<DirectedEdge>[]) new Bag[V];
    for (int v = 0; v < V; v++)
      adj[v] = new Bag<DirectedEdge>();
  }

  public int V() {
    return V;
  }

  public int E() {
    return E;
  }

  public void addEdge(DirectedEdge e) {
    adj[e.from()].add(e);
    E++;
  }

  public Iterable<DirectedEdge> adj(int v) {
    return adj[v];
  }

  public Iterable<DirectedEdge> edges() {
    Bag<DirectedEdge> bag = new Bag<DirectedEdge>();
    for (int v = 0; v < V; v++)
      for (DirectedEdge e : adj[v])
        bag.add(e);
    return bag;
  }
}
{% endhighlight %}

**SP**

{% highlight java linenos %}
public class SP {
  public SP(EdgeWeightedDigraph G, int s);  //constructor
  public double distTo(int v);        //distance from s to v, infinity if no path
  public boolean hasPathTo(int v)       //path from s to v?
  Iterable<DirectedEdge> pathTo(int v)    //path from s to v, null if none
}
{% endhighlight %}

#### 2.4.1 Shortest-paths Properties ####

{: .img_middle_hg}
![Dijkstra](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/4.4.1 shortest-paths properties.png)

#### 2.4.2 Dijkstra 算法 ####

><b>Dijkstra’s algorithm</b>: initialzing dis[s] to 0 and all other distTo[] entries to positive infinity, then we relax and add to the tree a non-tree vertex with the lowest distTo[] value, continuing until all vertices are on the tree or no non-tree vertex has a finiste distTo[] value.


{: .img_middle_lg}
![Dijkstra](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/Dijkstra.png)


{% highlight java linenos %}
public class DijkstraSP {
  private DirectedEdge[] edgeTo;
  private double[] distTo;
  private IndexMinPQ<Double> pq;

  public DijkstraSP(EdgeWeightedDigraph G, int s){
    edgeTo = new DirectedEdge[G.V()];
    distTo = new double[G.V()];
    pq = new IndexMinPQ<Double>(G.V());
    for (int v = 0; v < G.V(); v++)
      distTo[v] = Double.POSITIVE_INFINITY;
    distTo[s] = 0.0;
    pq.insert(s, 0.0);
    while (!pq.isEmpty())
      relax(G, pq.delMin());
  }

  private void relax(EdgeWeightedDigraph G, int v) {
    for (DirectedEdge e : G.adj(v)) {
      int w = e.to();
      if (distTo[w] > distTo[v] + e.weight()) {
        distTo[w] = distTo[v] + e.weight();
        edgeTo[w] = e;
        if (pq.contains(w))
          pq.changeKey(w, distTo[w]);
        else
          pq.insert(w, distTo[w]);
      }
    }
  }
  public double distTo(int v)
  public boolean hasPathTo(int v)
  public Iterable<Edge> pathTo(int v)
}
{% endhighlight %}

#### 2.4.3 AcyclicSP 算法####

假设1个edge-weighted digraph是无环的，那么寻找shortest path是否会比通常的digraph要简单？答案是YES!


><b>Acyclic edge-weighted digraphs</b>: For many natural applications, edge-weeighted digraphs are know to have no directed cycles. We now consider an algorithm for finding shortest paths that is simpler and faster than Dijkstra's algorithm for edge-weighted DAGs. Specifically, vertex relaxation, in combination with topological sorting, immediately presents a solution to the single-source shortes-paths probelm for edge-weighted DAGs. We initialize distTo[s] to 0 and all other distTo[] values to infinity, then relax the vertices , one by one, taking the vertices in topological order.


{: .img_middle_lg}
![AcyclicSP](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/AcyclicSP.png)


{% highlight java linenos %}
public class AcyclicSP{
  
   private DirectedEdge[] edgeTo;
   private double[] distTo;
   
   public AcyclicSP(EdgeWeightedDigraph G, int s){
      edgeTo = new DirectedEdge[G.V()];
      distTo = new double[G.V()];
      for (int v = 0; v < G.V(); v++)
         distTo[v] = Double.POSITIVE_INFINITY;
      distTo[s] = 0.0;
      Topological top = new Topological(G);
      for (int v : top.order())
         relax(G, v);
   }
   
   private void relax(EdgeWeightedDigraph G, int v)
   public double distTo(int v)         
   public boolean hasPathTo(int v)      
   public Iterable<DirectedEdge> pathTo(int v) 
}
{% endhighlight %}


#### 2.4.4 Bellman-Ford 算法####

><b>Bellman-Ford algorithm</b>: The queue-based implementation of the Bellman-Ford algorithm solves the single-source shortest-paths problem from a given source s(or finds a negative cycle reachable from s) for any edge-weighted digraph with E edges and V vertices, in time proportional to EV and extra space proportional to V, in the worst case.


{: .img_middle_lg}
![Bellman-ford](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/Bellman-ford.png)


{% highlight java linenos %}
public class BellmanFordSP{
  private double[] distTo;        // length of path to v
  private DirectedEdge[] edgeTo;      // last edge on path to v 
  private boolean[] onQ;          // Is this vertex on the queue?
  private Queue<Integer> queue;     // vertices being relaxed
  private int cost;           // number of calls to relax()
  private Iterable<DirectedEdge> cycle;   // negative cycle in edgeTo[]?
  
  public BellmanFordSP(EdgeWeightedDigraph G, int s){
    distTo = new double[G.V()];
    edgeTo = new DirectedEdge[G.V()];
    onQ = new boolean[G.V()];
    queue = new Queue<Integer>();
    for (int v = 0; v < G.V(); v++)
      distTo[v] = Double.POSITIVE_INFINITY;
    distTo[s] = 0.0;
    queue.enqueue(s);
    onQ[s] = true;
    while (!queue.isEmpty() && !hasNegativeCycle()){
      int v = queue.dequeue();
      onQ[v] = false;
      relax(G, v);
    }
  }
     private void relax(EdgeWeightedDigraph G, int v)
     public double distTo(int v)          
     public boolean hasPathTo(int v)      
     public Iterable<Edge> pathTo(int v)  
     private void findNegativeCycle()
     public boolean hasNegativeCycle()
     public Iterable<DirectedEdge> negativeCycle()
}
{% endhighlight %}

#### 2.4.4 小结 ####


{: .img_middle_lg}
![STSummary](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/STSummary.png)


## 3 Programming assignment ##

### 3.1 WordNet ###

SAP思路：要计算最短路径，因此bfs的distTo可以用到。只要将BreadthFirstPaths增加返回marked和distTo的函数，就可以知道和vertex S 连接的点及其距离。然后在int v和 int w的公共marked点里，求其最小的点。SAP就是一个Graph和Graph processing 解耦的典型例子，SAP建立在BreadFirstPaths基础上。

{% highlight java linenos %}
public class SAP {
  Digraph G;
  
  // consructor takes a digraph (not necessarily a DAG)
  public SAP(Digraph G){
    this.G = G; 
  }
  
  // length of shortest ancestral path between v and w;-1 if no such path
  public int length(int v, int w){
    return this.shortPath(v, w)[0];
  }
  
  // a common ancestor of v and w that participates ina shortest ancestral path: -1 if no such path
  public int ancestor(int v, int w){
    return shortPath(v,w)[1];
  }
  
  // length of shortest ancestral path between any vertext in v and any vertex in w; -1 if no such path
  public int length(Iterable<Integer> v, Iterable<Integer> w){
    return this.shortPath(v, w)[0];

  }
  
  // a common ancestor that participates in shortest ancestral path: -1 if no such path
  public int ancestor(Iterable<Integer>v, Iterable<Integer> w){
    return this.shortPath(v, w)[1];
  }
  
  private int[] shortPath(int v, int w){
    
    int[] result = new int[2];
    result[0] = Integer.MAX_VALUE;
    result[1] = Integer.MAX_VALUE;
    
    UpgradedBFSDirectedPath BFSV = new UpgradedBFSDirectedPath(G, v);
    boolean[] markedV = BFSV.getMarked();
    int[]   distV = BFSV.distTo;
    
    UpgradedBFSDirectedPath BFSW = new UpgradedBFSDirectedPath(G, w);
    boolean[] markedW = BFSW.getMarked();
    int[]   distW = BFSW.distTo;
    
    int tempLength = Integer.MAX_VALUE;
    
    for(int i = 0; i < markedV.length;i++){
      if((markedV[i] & markedW[i]) && ((distV[i] +  distW[i]) < tempLength)){
        tempLength = distV[i] +  distW[i];
        result[0] = tempLength;
        result[1] = i;
      }
    }
    
    if(result[0] == Integer.MAX_VALUE) {
      result[0] = -1;
      result[1] = -1;
      return result;
    }
    return result;
  }

  
  private int[] shortPath(Iterable<Integer> v, Iterable<Integer> w){
    
    int[] result = new int[2];
    result[0] = Integer.MAX_VALUE;
    result[1] = Integer.MAX_VALUE;
    
    int minLength = Integer.MAX_VALUE;
    
    for(int i: v){
      for(int j: w){
        int length = shortPath(i, j)[0];
        int ancestor = shortPath(i,j)[1];
        if (length !=-1 && length < minLength){
          minLength = length;
          result[0] = minLength;
          result[1] = ancestor;
        } 
      }
    }
    if(result[0] == Integer.MAX_VALUE) {
      result[0] = -1;
      result[1] = -1;
      return result;
    }
    return result;
  }
  
  public static void main(String[] args) {
    StdOut.printf("hello world0:");
    In in = new In(args[0]);
    Digraph G = new Digraph(in);
    StdOut.printf("Digraph:" + G);
    SAP sap = new SAP(G);
    StdOut.printf("hello world1:");
    while(!StdIn.isEmpty()){
      int v = StdIn.readInt();
      int w = StdIn.readInt();
      StdOut.printf("hello world2:");
      int length   = sap.length(v, w);
      int ancestor = sap.ancestor(v, w);
      StdOut.printf("length = %d, ancestor = %d\n", length, ancestor);
    }
  }
  
  
  /*UpgradedBFSDirectedPath, with getMarked[]added-----------------------------*/
  
  private class UpgradedBFSDirectedPath{
    
    private static final int INFINITY = Integer.MAX_VALUE;
    private boolean[] marked;  // marked[v] = is there an s->v path?
    private int[] edgeTo;      // edgeTo[v] = last edge on shortest s->v path
    private int[] distTo;      // distTo[v] = length of shortest s->v path

    public UpgradedBFSDirectedPath(Digraph G, int s) {
      marked = new boolean[G.V()];
      distTo = new int[G.V()];
      edgeTo = new int[G.V()];
      for (int v = 0; v < G.V(); v++)
        distTo[v] = INFINITY;
      bfs(G, s);
    }

    public UpgradedBFSDirectedPath(Digraph G, Iterable<Integer> sources) {
      marked = new boolean[G.V()];
      distTo = new int[G.V()];
      edgeTo = new int[G.V()];
      for (int v = 0; v < G.V(); v++)
        distTo[v] = INFINITY;
      bfs(G, sources);
    }

    // BFS from single source
    private void bfs(Digraph G, int s) {
      Queue<Integer> q = new Queue<Integer>();
      marked[s] = true;
      distTo[s] = 0;
      q.enqueue(s);
      while (!q.isEmpty()) {
        int v = q.dequeue();
        for (int w : G.adj(v)) {
          if (!marked[w]) {
            edgeTo[w] = v;
            distTo[w] = distTo[v] + 1;
            marked[w] = true;
            q.enqueue(w);
          }
        }
      }
    }

    // BFS from multiple sources
    private void bfs(Digraph G, Iterable<Integer> sources) {
      Queue<Integer> q = new Queue<Integer>();
      for (int s : sources) {
        marked[s] = true;
        distTo[s] = 0;
        q.enqueue(s);
      }
      while (!q.isEmpty()) {
        int v = q.dequeue();
        for (int w : G.adj(v)) {
          if (!marked[w]) {
            edgeTo[w] = v;
            distTo[w] = distTo[v] + 1;
            marked[w] = true;
            q.enqueue(w);
          }
        }
      }
    }

    public boolean hasPathTo(int v) {
      return marked[v];
    }


    public int distTo(int v) {
      return distTo[v];
    }
    public boolean[] getMarked(){
      return this.marked;
    }

    public Iterable<Integer> pathTo(int v) {
      if (!hasPathTo(v)) return null;
      Stack<Integer> path = new Stack<Integer>();
      int x;
      for (x = v; distTo[x] != 0; x = edgeTo[x])
        path.push(x);
      path.push(x);
      return path;
    }
  }
}

{% endhighlight %}

WordNet思路: 这里有一些细节需要澄清，

1. 首先，synset是一个vertex, hypernerm里存的是edges, 即synset->synset集合，因此synset和hypernerm可以构建图。
2. WordNet API 需要返回synset里(vertex)的所有单词及判断单词是否在里面，由于需要快速查找，插入，因此用SET来保存单词Noun(iVar String noun & iVar ArrayList id(每一个单词可以属于多个vertext)，实现Comparable)。
3. idList数组保存vertex id 和 String的关系；
4. 判断WordNet是否为单root，hypernerm出现一项，标记其为true，最后数false的个数，有且只有一个为单root。

{% highlight java linenos %}
public class WordNet {
  
  private class Noun implements Comparable<Noun>{
    private String noun;
    private ArrayList<Integer> id = new ArrayList<Integer>();
    
    public Noun(String noun){
      this.noun = noun;
    }
    
    @Override
    public int compareTo(Noun o) {
      return this.noun.compareTo(o.noun);
    }
    
    public ArrayList<Integer> getId(){
      return this.id;
    }
    
    public void addId(Integer x){
      this.id.add(x);
    }
  }
  
  // constructor takes the name of the two input files
  private SET<Noun> nounSET;
  private Digraph G;        //store hypernym
  private SAP sap;
  private ArrayList<String> idList;//store synset

  
  public WordNet(String synsets, String hypernyms){
    In inSynsets = new In(synsets);
    In inHypernyms = new In(hypernyms);
    
    //counting the total number of vertex
    int maxVertex = 0;
    idList = new ArrayList<String>();
    nounSET = new SET<Noun>();
    
    //start to read synsets.txt
    String line = inSynsets.readLine();
    while(line != null){
      maxVertex++;
      String[] synsetLine = line.split(",");
      //String[0] is id
      Integer id = Integer.parseInt(synsetLine[0]);
      //String[1] is noun, split it and add to the set
      String[] nounSet = synsetLine[1].split(" ");
      for(String nounName: nounSet){
        Noun noun = new Noun(nounName);
        if(nounSET.contains(noun)){
          noun = nounSET.ceiling(noun);
          noun.addId(id);
        }else{
          nounSET.add(noun);
          noun.addId(id);
        }
      }
      
      // add it to the idList
      idList.add(synsetLine[1]);
      //continue readind synsets
      line = inSynsets.readLine();
    }
      
    G = new Digraph(maxVertex);
    // the candidate root
    boolean[] isNotRoot = new boolean[maxVertex];
      
    //start to read hypernyms.txt
    line = inHypernyms.readLine();
    while(line != null){
      String[] hypernymsLine = line.split(",");
      //String[0] is id
      int v = Integer.parseInt(hypernymsLine[0]);
      isNotRoot[v] = true;
      //String[1] and others is its ancestor, constrcuting G
      for(int i = 1; i < hypernymsLine.length; i++){
        G.addEdge(v, Integer.parseInt(hypernymsLine[i]));
      }
      line = inHypernyms.readLine();
    }
      
    //test for root: no cycle
    DirectedCycle directedCycle = new DirectedCycle(G);
    if(directedCycle.hasCycle()){
      throw new java.lang.IllegalArgumentException();
    }
      
    //test for root: no more than on candidate root
    int rootCount = 0;
    for (boolean notRoot: isNotRoot){
      if(!notRoot){
        rootCount++;
      }
    }
    if(rootCount > 1){
      throw new java.lang.IllegalArgumentException();
    }
    sap = new SAP(G);
  }
  
  // returns all WordNet nouns
  
  public Iterable<String> nouns(){
    Queue<String> nouns = new Queue<String>();
    for(Noun noun: nounSET){
      nouns.enqueue(noun.noun);
    }
    return nouns;
  }
  
  // is the word a WordNet noun?
  public boolean isNoun(String word){
    Noun noun = new Noun(word);
    return nounSET.contains(noun);
  }
  
  // distance between nounA and nounB(defined below)
  public int distance (String nounA, String nounB){
    if(!isNoun(nounA)){
      throw new java.lang.IllegalArgumentException();
    }
    if(!isNoun(nounB)){
      throw new java.lang.IllegalArgumentException();
    }
    
    Noun nodeA = nounSET.ceiling(new Noun(nounA));
    Noun nodeB = nounSET.ceiling(new Noun(nounB));
    return sap.length(nodeA.getId(), nodeB.getId());
  }
  
  // a synset (second field of synsets.txt) that is the common ancestor of nounA and nounB
  // in a shortest ancestral path (defined below)
  public String sap(String nounA, String nounB){
    if(!isNoun(nounA)){
      throw new java.lang.IllegalArgumentException();
    }
    if(!isNoun(nounB)){
      throw new java.lang.IllegalArgumentException();
    }
    
    Noun nodeA = nounSET.ceiling(new Noun(nounA));
    Noun nodeB = nounSET.ceiling(new Noun(nounB));
    return idList.get(sap.ancestor(nodeA.getId(), nodeB.getId()));
  }
  
  public static void main(String[] args) {
    String synsets = args[0];
    String hypernyms = args[1];
    
    WordNet wordNet = new WordNet(synsets, hypernyms);
  }
}

{% endhighlight %}



Outcast思路：求和。由于N*N矩阵对称，因此可以先填充再求和减少计算次数。


{% highlight java linenos %}
public class Outcast {
  private WordNet wordNet;
  
  public Outcast(WordNet wordNet){
    this.wordNet = wordNet;
  }
  
  public String outcast(String[] nouns){

        int[] distance = new int[nouns.length];  
        for (int i=0; i<nouns.length; i++){  
            for (int j=i; j<nouns.length; j++){  
                int dist = wordNet.distance(nouns[i], nouns[j]);
                distance[i] += dist;  
                if (i != j){  
                    distance[j] += dist;  
                }  
            }  
        }  
        int maxDistance = 0;  
        int maxIndex = 0;  
        for (int i=0; i<distance.length; i++){  
            if (distance[i] > maxDistance){  
                maxDistance = distance[i];  
                maxIndex = i;  
            }  
        }  
        return nouns[maxIndex];  
    }  

  public static void main(String[] args) {
    WordNet wordnet = new WordNet(args[0],args[1]);
    Outcast outcast = new Outcast(wordnet);
    for(int t = 2; t < args.length; t++){
      In in = new In(args[t]);
      String[] nouns = in.readAllStrings();
      StdOut.println(args[t] + ":" + outcast.outcast(nouns));
    }
  }
}


{% endhighlight %}

### 3.2 Seam Carving###

思路：

1. 用一个virtual top vertice 和 vritual bottom vertice,然后求它们的最短路径。
2. Vertice(像素), DirectedEdge是每一个像素和它下面,下左,下右。
3. 建立一个EdgeWeightedDigraph，然后用AcyclicSP,选取最短路径。


{% highlight java linenos %}
import java.awt.Color;
import edu.princeton.cs.algs4.AcyclicSP;
import edu.princeton.cs.algs4.DirectedEdge;
import edu.princeton.cs.algs4.EdgeWeightedDigraph;
import edu.princeton.cs.algs4.Picture;

public class SeamCarver {
  private Picture picture;
  
  private double[][] energyMatrix;          //energyMatrix
  private EdgeWeightedDigraph G;            //G
  private Color[][] colors;             // the basic information
  
  public SeamCarver(Picture picture){         // create a seam carver object based on the given picture
    this.picture = picture;
    this.colors = new Color[picture.width()][picture.height()];   // make colors ready before calling other function
    for(int col = 0; col < picture.width(); col++){
      for(int row = 0; row < picture.height(); row++){
        this.colors[col][row] = this.picture.get(col, row);
      }
    }
    this.initialize();
  }
  
  private void initialize(){            
    this.energyMatrix = new double[this.width()][this.height()];  //construct energyMatrix from colors
    for(int col = 0; col < this.width(); col++){
      for(int row = 0; row < this.height(); row++){
        energyMatrix[col][row] = this.energy(col, row);
      }
    }
    
    this.G = new EdgeWeightedDigraph(this.width()*this.height()+2);   //construct energyMatrix from colors
    for (int col = 0; col < this.width(); col++){           //add top 
      this.G.addEdge(new DirectedEdge(0, this.colRowToV(0, col), 0));
    }
    for(int col = 0; col < this.width(); col++){            //add bottom
      this.G.addEdge(new DirectedEdge(this.colRowToV(this.height()-1, col), this.height()*this.width()+1, this.energyMatrix[col][this.height()-1]));
    }
    for(int row = 0; row < this.height(); row++){           //add all other edges
      for(int col = 0; col < this.width(); col++){
        int vertice = this.colRowToV(row, col);
        if(row >= this.height()-1) continue;
        if(col > 0)this.G.addEdge(new DirectedEdge(vertice, this.colRowToV(row+1, col-1), this.energyMatrix[col-1][row+1]));
        this.G.addEdge(new DirectedEdge(vertice, this.colRowToV(row+1, col),this.energyMatrix[col][row+1]));
        
        if(col < width()-1) this.G.addEdge(new DirectedEdge(vertice, this.colRowToV(row+1, col+1),this.energyMatrix[col+1][row+1]));
      }
    }
  }
  
  private int colRowToV(int row, int col){    //convert x,y index into vertices
    return row *this.width() + col+1;
  }
  
  public Picture picture(){           // current picture
    
    Picture newPicture = new Picture(this.width(), this.height());
    for(int row = 0; row < newPicture.height(); row++){
      for(int col = 0; col < newPicture.width(); col++){
        newPicture.set(col, row, this.colors[col][row]);
      }
    }
    this.picture = newPicture;
    return this.picture;
  }           
  
  public int width(){               // width of current picture
    return this.colors.length;
  }
  
  public int height(){              // height of current picture
    return this.colors[0].length;
  }
  
  public double energy(int x, int y){       // energy of pixel at column x and row y
    if(x<0 || x >= this.width() || y < 0 || y >= this.height()) throw new java.lang.IllegalArgumentException();
    if(x == 0 || x == this.width()-1 || y == 0 || y == this.height()-1) return 1000.00;
    double energy = Math.sqrt(xEnergy(x,y) + yEnergy(x,y));
    return energy;
  }
  
  
  private double xEnergy(int x, int y){
    
    Color leftColor = this.colors[x-1][y];
    Color rightColor = this.colors[x+1][y];
    double Rx = leftColor.getRed()-rightColor.getRed();
    double Gx = leftColor.getGreen()-rightColor.getGreen();
    double Bx = leftColor.getBlue()-rightColor.getBlue();
    double xEnergySquare = Math.pow(Rx, 2) + Math.pow(Gx,2) + Math.pow(Bx,2);
    return xEnergySquare;
  }

  
  private double yEnergy(int x, int y){
  
    Color topColor = this.colors[x][y-1];
    Color bottomColor = this.colors[x][y+1];
    double Ry = topColor.getRed()-bottomColor.getRed();
    double Gy = topColor.getGreen()-bottomColor.getGreen();
    double By = topColor.getBlue()-bottomColor.getBlue();
    double yEnergySquare = Math.pow(Ry, 2) + Math.pow(Gy,2) + Math.pow(By,2);
    return yEnergySquare;
  }
  
  public int[] findHorizontalSeam(){        // sequence of indices for horizontal seam
    SeamCarver horizontalSC = new SeamCarver(transpose());
    return horizontalSC.findVerticalSeam();
  }
  
  private Picture transpose(){
    int width = this.height();
    int height = this.width();
    
    Picture newPicture = new Picture(width, height);
    for(int col = 0; col < width; col++){
      for(int row = 0; row < height; row++){
        Color color = this.colors[row][col];
        newPicture.set(col, row, color);
      }
    }
    return newPicture;
  }
  
  public int[] findVerticalSeam(){        // sequences of indices for vertical seam
    int[] solutionSteps = new int[height()];
    AcyclicSP aSP = new AcyclicSP(this.G,0);
    int i = 0;
    for(DirectedEdge e: aSP.pathTo(width()*height()+1)){
      if (e.to() == width()*height()+1) continue;
      int col = e.to() % width()-1;
      solutionSteps[i] = col;
      i++;
    }
    return solutionSteps;
  }
  
  public void removeHorizontalSeam(int[] seam){   // remove horizontal seam from current picture
  
    Color[][] newColors = new Color[this.width()][this.height()-1];
    for(int col= 0; col< width(); col++){
      for(int row = 0; row < this.height()-1; row++){
        if(row < seam[col]) newColors[col][row] = this.colors[col][row];
        else{
          newColors[col][row] = this.colors[col][row+1];
        }
      }
    }
    this.colors = newColors;
    this.initialize();
  }
  
  public void removeVerticalSeam(int[] seam){   // remove vertical seam from current picture
    
    Color[][] newColors = new Color[this.width()-1][this.height()];
    for(int row = 0; row < this.height(); row++){
      for(int col = 0; col < this.width()-1; col++){
        if(col < seam[row]) newColors[col][row] = this.colors[col][row];
        else{
          newColors[col][row] = this.colors[col+1][row];
        }
      }
    }
    this.colors = newColors;
    this.initialize();
  }
}
{% endhighlight %}




## 4 总结 ##


{: .img_middle_hg}
![STSummary](/assets/images/posts/01_Algorithm/2015-09-08_Algorithm(Part IV)：Graph(二)：最小生成树和最短路径/
Chapter 4 Graph Summary.png)

## 5 参考资料 ##
- [Algorithm](http://algs4.cs.princeton.edu/home/);

- [Visualize Algorithm](http://visualgo.net/);





