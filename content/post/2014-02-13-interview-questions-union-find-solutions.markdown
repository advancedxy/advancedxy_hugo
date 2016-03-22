---
comments: true
date: 2014-02-13T22:32:36Z
tags: ['algorithms', 'coursera', 'job-interview']
title: 'Interview questions: union find solutions'
url: /blog/2014/02/13/interview-questions-union-find-solutions/
author: Xianjin YE
---
最近在学习Coursera上的[Algorithms, Part I](https://class.coursera.org/algs4partI-004),其每个week除了正常的Programming Assignment外还有面试问题.由于面试问题没有提交评分要求,想来在blog这里分享解决方案应该是没有违反honor code的.故这篇blog将是Algorithms面试问题题解系列的第一篇.

# Question 1

原题引用如下:

>
> **Social network connectivity.** Given a social network containing N members and a log file containing M timestamps at which times pairs of members formed friendships, design an algorithm to determine the earliest time at which all members are connected (i.e., every member is a friend of a friend of a friend ... of a friend). Assume that the log file is sorted by timestamp and that friendship is an equivalence relation. The running time of your algorithm should be MlogN or better and use extra space proportional to N.

<!--more-->

## 问题分析

这是个非常典型的Union Find问题.由于题目中给定了友谊是等价关系,也即表示友谊有自反性(自己是自己的朋友),传递性(A和B是朋友,B和C是朋友,那么A和C也是朋友).那么什么时候整个网络是联通的?A和所有其他成员都是朋友(不管是直接朋友还是间接朋友,从等价关系上来看,这两个是一致的).
用Union Find的来建模上述问题便是:

1. 一开始所有人都只是跟自己是朋友 -> Union Find数据初始化
2. $t_i$时刻, $m_i$和$m_j$成为朋友 -> connect($m_i$, $m_j$)
3. 判断网络是否联通 -> Union Find中的count == 1

## 问题解决

具体的代码就不写了,直接给个伪代码吧.

{{< gist advancedxy 9033895 "Network.java" >}}

# Question 2

原题引用如下:

>
> **Union-find with specific canonical element.** Add a method find() to the union-find data type so that find(i) returns the largest element in the connected component containing i. The operations, union(), connected(), and find() should all take logarithmic time or better.

## 问题分析

这个问题的要求对union find的数据结构进行更改,并且要求所有的操作:union, connected, find的时间复杂度为log(N).先不考虑要做的修改,union find的数据结构能够满足这一要求不能是简单的quick find或者quick union.利用课上学习的WeightedQuickUnionUF能够满足这一时间要求.那么,
要实现题目中提出的find返回最大数的要求,最简单的做法便是再用一个数组记录每个root上对应的最大值.具体实现见代码

## 问题解决

{{< gist advancedxy 9033895 "WeightedQuickUnionLargestUF.java" >}}

# Question 3

原题引用如下:

> **Successor with delete.** Given a set of N integers S={0,1,...,N−1} and a sequence of requests of the following form:
> 1. Remove x from S
> 2. Find the successor of x: the smallest y in S such that y≥x.
> 
> design a data type so that all operations (except construction) should take logarithmic time or better.

## 问题分析

首先,得确定一个问题,就是remove和find操作是两个独立的操作.在一系列的requests中,remove和find可以不连续出现,也可以不是对同一个x元素进行操作.比如说下面的一系列操作就是合法的:remove(m), find(n), remove(o), remove(p), find(m).

其次,remove和find的时间要求是logN的时间.对于这样的时间要求,我的第一反应是二叉树的变体来实现,但由于这是union find的面试题,这给了我很大的提示使用union find类似的数据结构来解决这一问题.

仔细观察remove和find的关系:

1. remove完全是为find服务的.remove的最终动作可能只是个更改flag位,以便于让find能够确定该元素是否被删除
2. find(x)的结果只受到了remove(y)的影响,其中y >= x.

那么很naive的一个实现便是用boolean[N]来存储N个整数是否被删除,对于删除数据而言只是flag的更改,是O(1)的操作.find(x)
的操作最坏情况下会遍历所有x到N-1的元素,时间复杂度是O(N)级别的,不满足题目要求.

思考之前提出来的实现,改进的空间在find上.如何改进find?我从union find里得到的直觉便是最开始的数组id[N]里全部的元素
全部指向自己.如果x被删除了,则将x指向下一个元素,也即$id[x] = x + 1 $.那么find的实现便和union find的实现类似,并且可以
应用path compression,让整个数据结构更加flat,可以加速搜索.

但上面的实现是不满足要求的.即使可以对remove的操作做优化:id[x] = find(x+1),将被删除的元素指向下一个元素的find值.这样做
可能导致一些情况下,find和remove的速度较快,但不可避免的worst case是:remove(0), remove(1),....remove(N-2),find(1).最后
的find操作是O(N)的时间复杂度.

仔细观察remove的操作,一个不可忽视的缺陷便是会产生单节点树,也即母节点下面只有一个子节点.这也是上面产生worst case的原因.
再仔细观察连续被删除的整数和紧接着的下一个整数,也即find返回的值.这个其实可以算作是一个联通的区域,上面的实现会将find返回
值放在root位置.

等等!!为什么我们一定要将最大的值放在root位置,这是我们不能获得log(N)时间复杂度的元首.我们完全可以再用一个数组来标识对应区域
的最大值,就像我们在第二个题目的实现一样.

## 问题解决

参考第二个题目的实现.

remove(x) => connect(x, x+1)

find(x) => find(x) // return the largest element


# Question 4

原题引用如下:

>
> **Union-by-size.** Develop a union-find implementation that uses the same basic strategy as weighted quick-union but keeps track of tree height and always links the shorter tree to the taller one. Prove a lgN upper bound on the height of the trees for N sites with your algorithm.

## 问题答案

没什么好说的,直接上代码

{{< gist advancedxy 9033895 "HeightedQuickUnionUF.java" >}}

## 算法时间复杂度证明

设函数h(n)表示该算法下高度为n的最小数目的sites.那么最初始的值为:

h(0) = 1 // 高度为0, 也即只有一个节点..

h(1) = 2 // 高度为1,至少有两个节点

由于只有在两个节点所代表的树高度一致时才会进行高度的更新,且高度只加一,故
$$ h(n+1) = h(n) + h(n) = 2h(n) = 2*2*h(n-1) = ... = 2^{ n+1 } $$

也即高度为n的树包含的sites至少为$2^n$,也即对于N个sites的树来说,高度最高为$log_2N$.得证!
