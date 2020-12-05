---
layout:     post
title:      A Star Algorithm
subtitle:   
date:       2020-12-3
author:     Hao
header-img: img/post/post_bg_coffee.jpg
catalog: true
mathjax: true
tags:
    - Path Planning
    - Searching
---

A Star 算法是一种很常用的路径查找和图形遍历算法，最初发表于1968年，由Stanford研究院的Peter Hart, Nils Nilsson以及Bertram Raphael提出。它可以被认为是Dijkstra算法的扩展，由于借助启发函数的引导，A*算法通常拥有更好的性能。

### 从 BFS 开始

为了更好的理解和学习 A Star 搜索算法，我们先学习广度优先搜索 (Breadth First Search)。

顾名思义，广度优先搜索是指从起始点开始，遍历起点周围的邻居节点，然后再对每一个邻居节点遍历它邻近的点，逐步的向外扩散，直到找到终点。这种算法就像洪水 (Flood fill) 一样向外扩张，算法的过程如下图所示：

![img](/img/post/breadth_first.gif)

上面这幅动图中算法遍历了图中所有的点，这通常没有必要。对于有明确终点的问题来说，一旦到达终点便可以提前终止算法，下面这幅图对比了这种情况：

![img](/img/post/early_exit.png)

在算法执行过程中，每个点需要记录到达该点的前一个点位置 (父节点)。一旦到达终点，便可以从终点开始，反过来顺着父节点的顺序找到起点，由此就构成了一条路径。

### Dijkstra 算法

Dijkstra算法是由计算机科学家Edsger W. Dijkstra在1956年提出的，用来寻找图中单节点的最短路径。\
在图中相邻节点之间的移动代价并不相等。例如，游戏中的一幅图，既有平地也有山脉，那么游戏角色在平地和山脉中的移动速度通常是不相等的。\
在Dijkstra算法中，需要计算每一个节点距离起点的总移动代价。同时，还需要维护一个优先级队列，对于所有待遍历的节点，放入优先级队列中会按照代价进行排序。在算法运行的过程中，每次都从优先队列中选出代价最小的作为下一个遍历的节点。直到到达终点为止。

![img](/img/post/dijkstra.gif)

+ 当图形为网格图，并且每个节点之间的代价相等时，那么Dijkstra算法将等同于广度优先算法。

### 最佳优先算法

在某些情况下，如果我们可以预先计算出每个节点到终点的距离，那么我们可以利用这个信息更快的到达终点。\
其原理也很简单。与 Dijkstra 算法相似，我们维护一个优先级队列，此时以每个节点到终点的距离作为优先级，每次始终选取离终点距离最近的节点作为将要遍历的节点。这种算法称之为最佳优先 (Best First) 算法。

这样做可以大大加快路径的搜索速度，如下图示例所示：

![img](/img/post/dijkstra.gif)

但这种算法有啥缺点呢？想想如果起点和终点之间存在障碍物，则最佳优先算法搜索的结果可能不是最短路径，如下图所示：

![img](/img/post/dijkstra.gif)

### A* 算法

介绍了上面几种算法，最后开始本文的重点：A* 算法。根据下面的描述我们将看到，A* 算法实际上是综合上面这些算法的特点。

A*算法通过下面这个函数来计算每个节点的优先级。

$$f(n) = g(n) + h(n)$$

其中：
+ $f(n)$ 是节点 n 的综合优先级。当我们选择下一个要遍历的节点时，总会选取综合优先级最高 (值最小) 的节点。
+ $g(n)$ 是节点 n 距离起点的代价。
+ $h(n)$ 是节点 n 距离终点的预计代价，也就是 A* 算法的启发式函数。

A* 算法在运行过程中，每次从优先队列中选取 $f(n)$ 值最小 (优先级最高) 的节点作为下一个待遍历的节点。另外，A* 算法使用两个集合来表示待遍历的节点和已经遍历过的节点，通常称之为 open_list 和 close_list。

完整的 A* 算法描述如下：

* 初始化 open_list 和 close_list.
* 将 start node 加入 open_list 中，并设置优先级 f 为 0 (优先级最高).
* 如果 open_list 不为空，则从 open_list 中选取 f 优先级最高的节点 n :
    * 如果节点 n 为终点，则 : 
        * 从终点开始逐步追踪 parent 节点，直到达到起点.
        * 返回找到的结果路径，算法结束.
    * 如果节点 n 不是终点，则 :
        * 将节点 n 从 open_list 中删除，并加入 close_list 中；
        * 遍历节点 n 的所有不在 close_list 中的邻居节点 : 
            * 如果邻近节点 m 在open_list 中，则 :
                * 计算节点 m 以 n 为 parent 的代价 g.
                * 如果该代价 < m 的代价，就更新节点 m 的代价 g 和优先级 f.
            * 如果邻近节点 m 不在open_list 中，则 :
                * 设置节点 m 的 parent 为节点 n.
                * 计算节点 m 的优先级.
                * 将节点 m 加入 open_list 中.

### 关于启发式函数

上面已经提到，启发函数会影响 A* 算法的行为。

+ 在极端情况下，当启发函数 $h(n)$ 始终为0，则将由 $g(n)$ 决定节点的优先级，此时算法就退化成了 Dijkstra 算法。
+ 如果 $h(n)$ 始终小于等于节点 n 到终点的代价，则 A* 算法保证一定能找到最短路径。但是当 $ h(n)$ 的值越小，算法将遍历越多的节点，也就导致算法越慢。
+ 如果 $h(n)$ 完全等于节点 n 到终点的代价，则 A* 算法将找到最佳路径，并且速度很快。可惜的是，并非所有场景下都能做到这一点。因为在没有达到终点之前，我们很难准确估算出距离终点还有多远。
+ 如果 $h(n)$ 的值比节点 $n$ 到终点的代价要大，则 $A*$ 算法不能保证找到最短路径，不过此时会很快。
+ 在另外一个极端情况下，如果 $h(n)$ 远大于 $g(n)$，则此时只有h(n)产生效果，这也就变成了最佳优先搜索。

通过上述这些分析我们可以知道，通过调节启发函数我们可以控制算法的速度和准确度。因为在某些场景下，我们可能未必需要最短路径，而是希望尽快找到一条路径。这也是 A* 算法的灵活之处。

### 关于距离

对于网格形式的图，有以下启发函数可以使用：

 + 图形中允许上下左右四个方向移动，可以使用曼哈顿距离。
 + 图形中允许八个方向移动，则可以利用对角距离。
 + 如果图形中允许任意方向运动，则可以使用欧式距离。

### 算法实现

前面介绍了很多 A* 的理论内容，但实际上 A* 的算法实现并不复杂，下面给出 Java 代码的示例。

算法源码见[这里]()。

参考自：
1. [路径规划之 A* 算法](https://paul.pub/a-star-algorithm/)