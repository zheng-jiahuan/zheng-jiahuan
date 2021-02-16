---
title: 图相关知识回顾
author: zjh
date: 2019-10-10
last_modified_at: 2021-02-16
categories: [compiler]
tags: [graph]
---

## 引言
编译器中经常会涉及一些图相关的概念，本文借此粗略整理了一些图相关的概念，一是回顾回顾，二是方便日后查阅。


## KeyWords
Graph; Tree; Tree Edge; Back Edge; Forward Edge; Cross Edge; Cycle; DAG; Articulation Point (or Vertex); Connected Componet; Strongly-connected Component; Reverse Post-order; Topological sort


## Content
 
### 图
 
 一般地，图分有向图（Directed Graph）和无向图（Undirected Graph）
 
#### 无向图（Undirected Graph）
 

      Graphs : A graph is a set of vertices and a collection of edges that each connect a pair of vertices
     
![undirected graph](/assets/img/2019-10-10-graph-basics/undirected-graph.png)

 关于无向图的一些名词定义：
 

    A path in a graph is a sequence of vertices connected by edges, with no repeated edges.
    A cycle is a path (with at least one edge) whose first and last vertices are the same.
    We say that one vertex is connected to another if there exists a path that contains both of them.
    A graph is connected if there is a path from every vertex to every other vertex.
    An  acyclic  graph is a graph with no cycles.
    A tree is an acyclic connected graph.
    A spanning tree of a connected graph is a subgraph that contains all of that graph's vertices and is a single tree.
    Bridge: Abridge (or cut-edge) is an edge whose deletion increases the number of connected components. Equivalently, an edge is a bridge if and only if it is not contained in any cycle.
    Biconnectivity : An articulation vertex (or cut vertex) is a vertex whose removal increases the number of connected components. A graph is  biconnected  if it has no articulation vertices.

 几个名词的补充：

     A tree is an undirected graph in which any two vertices are connected by only one path. A tree is an acyclic graph and has N - 1 edges where N is the number of vertices. Each node in a graph may have one or multiple parent nodes. However, in a tree, each node (except the root node) comprises exactly one parent node.
    Articulation Point: In a graph, a vertex is called an articulation point if removing it and all the edges associated with it results in the increase of the number of connected components in the graph.
    Bridges: An edge in a graph between vertices say u and v is called a Bridge, if after removing it, there will be no path left between u and v.

![Fig.1](/assets/img/2019-10-10-graph-basics/fig-1.png)
![Fig.2](/assets/img/2019-10-10-graph-basics/fig-2.png)


 如上图中的Fig.1，如果此时我们将1号点(以及与1号点相关的边)移除，就会变成Fig.2，而后者的连通分量( Connected Component) 就变成了两个，原先为一个，现在变成了两个，连通分支增加了，所以1号点属于Articulation Point。同样地，对于上图中的Fig1，如果我们将0-1这条边删掉，会造成连通分支个数的增加，所以0-1这条边属于bridge。

给定一个网络，Articulation Point和Bridge越多，则证明这个网络越容易受攻击，因为删除其中一个，整个网络就不能互相连通了。

    A graph is said to be Biconnected if:
    It is connected, i.e. it is possible to reach every vertex from every other vertex, by a simple path.
    Even after removing any vertex the graph remains connected.

![Fig.3](/assets/img/2019-10-10-graph-basics/fig-3-biconnected.png)
上面这幅图就是biconnected的。懂了什么是Biconnected Graph，那么什么是Biconnected Component呢？Biconnected Component是图的一个子图，这个子图是Biconnected的。 
![Fig.4](/assets/img/2019-10-10-graph-basics/fig-4-biconnected.png)

对于上图来说，该图含有4个Biconnected Component。

#### 有向图（Digraph）

```
  Digraphs : A directed graph(or digraph) is a set of vertices and a collection of directed edges that each connects an ordered pair of vertices.
```
![Fig.5](/assets/img/2019-10-10-graph-basics/fig-5-directed-graph.png)

 关于有向图的一些名词定义 ：
 

    We say that a vertex w is reachable from a vertex v if there exists a directed path from v to w.
    We say that two vertices v and w are strongly connected if they are mutually reachable: there is a directed path from v to w and a directed path from w to v.
    A digraph is strongly connected if there is a directed path from every vertex to every other vertex.
    A directed acyclic graph (or DAG) is a digraph with no directed cycles.

 值得注意的是，我们谈及Strong Connectivity的时候，都是针对有向图的。Strongly Connected Components是有向图的一个子图，这个子图是strongly connected。
 
![Fig.6](/assets/img/2019-10-10-graph-basics/fig-6-scc.png)

 对于上面这个图来说，我们可以找到两个强连通分量（scc），图中已经标识出来了。

 关于有向图Depth-first orders的三种典型应用：
 

    Preorder: Put the vertex on a queue before the recursive calls.
    Postorder: Put the vertex on a queue after the recursive calls.
    Reverse postorder: Put the vertex on a stack after the recursive calls.

 有向图的拓扑排序
 
    Topological sort : given a digraph, put the vertices in order such that all its directed edges point from a vertex earlier in the order to a vertex later in the order.

 Reverse postorder经常缩写成RPO，需要注意的一点，也是很重要的一点就是：给定一个DAG图，对其进行RPO遍历，得到的遍历结果属于该图的Topological  order。这个RPO我们后面还会提及。

  几个重要命题：
 

    A digraph has a topological order if and only if it is a DAG.
    Reverse postorder in a DAG is a topological sort.

 有向图中的几种边
 
 我们经常能够听到这几种边，下面将通过一个例子了解下这几个边
 

    Tree Edge
    Back Edge
    Forward Edge
    Cross Edge
    Critical Edge


![Fig.7](/assets/img/2019-10-10-graph-basics/fig-7-dfs-tree.png)

 我们对一个有向图进行深度优先遍历，会得到一个DFS Tree

 ```c
 DFS(u):
  visited[u] = true
  for each successor v of u:
    if not visited[v]:
       DFS(v)
 ```
 在这篇博文中，它对各个边的定义是这样的：
 

     Tree Edge : It is a edge which is present in tree obtained after applying DFS on the graph   
     Back edge : It is an edge (u, v) such that v is ancestor of edge u but not part of DFS tree   
     Forward Edge : It is an edge (u, v) such that v is descendant but not part of the DFS tree   
     Cross Edge : It is a edge which connects two node such that they do not have any ancestor and a descendant relationship between them   

 在这篇博文中，它对各个边的定义是这样的： 
 
 With the graph version of DFS, only some edges (the ones for which visited[v] is false) will be traversed. These edges will form a tree, called the depth-first-search tree of G starting at the given root, and the edges in this tree are called tree edges. The other edges of G can be divided into three categories:
 

     Back edges point from a node to one of its ancestors in the DFS tree.   
     Forward edges point from a node to one of its descendants.   
     Cross edges point from a node to a previously visited node that is neither an ancestor nor a descendant.   

 这些边是非常有用处的，比如，我们说一个图是无环的，当且仅当这个图没有Back Edge (a graph is acyclic if and only if it has no back edges)
 
 那么给定一个边，我们怎么能够得出它是属于哪一类的边呢？博文中给了这样一个DFS算法：
 
```
AnnotatedDFS(u, parent):
  parent[u] = parent
  start[u] = clock; clock = clock + 1
  visited[u] = true
  for each successor v of u:
    if not visited[v]:
      AnnotatedDFS(v, u)
  end[u] = clock; clock = clock + 1
```

这个算法给每个vertex都添加上了一个起始计数start和结束计数end，对于一条边来说，通过对比这条边的两个顶点的start和end计数，可以得出这条边所属类型. Tree Edge是最好识别的，如果parent[v] = u，那么uv就是Tree Edge.

![Fig.8](/assets/img/2019-10-10-graph-basics/fig-8-edges.png)

 我们上面对Tree Edge、Back Edge、Forward Edge以及Cross Edge进行了解释，但是在CFG中经常能够碰到的Critical Edge还没有进行解释，那么什么是Critical Edge呢？ 
 
 1.wiki中的定义
 

      A critical edge is an edge which is neither the only edge leaving its source block, nor the only edge entering its destination block.
     

 2.Engerring A Compiler 2nd Page 512中的定义
 

      In a CFG, an edge whose source has multiple successors and whose sink has multiple predecessors is called a critical edge.

![Fig.9](/assets/img/2019-10-10-graph-basics/fig-9-critical-edge.png)
 依据Critical Edge的定义，我们知道，上图中，蓝色的有向边uv就属于Critical Edge，因为该蓝色边的起点u有多个successor，终点v有多个predecessor。
 
 注：有向边的起点常常称之为source、tail，有向边的终点常常称之为sink、destination、head等。 
