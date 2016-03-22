---
comments: true
date: 2015-01-16T11:35:20Z
tags: [ 'project-euler', 'algorithm', 'permutation']
title: Permutations and Cantor expression
url: /blog/2015/01/16/permutations-and-cantor/
---

# 概要

Permutation(排列) 和 Combination(组合) 是数学中的经典计数问题, 高中的数学中有一
小部分知识是关于计算排列数和组合数的, 在大学的后续学习中, 排列和组合更是在统计概
率应用颇多. 而在计算机科学领域, 我们更关心的是列出排列和组合. 本文是我在完成
hackerrank 上面的[Project Euler #24](https://www.hackerrank.com/contests/projecteuler/challenges/euler024)
后, 以及之前在[Leetcode](https://oj.leetcode.com/problemset/algorithms/)刷题的时
候碰到的排列相关的问题, 决定写篇小文章来记述一下其中涉及到 nth permutation 和
next permutation 问题.

# 名词解释

在进行下一步行文之前, 个人认为有必要介绍一下何为排列的顺序. 排列的顺序跟元素本身
的大小无关(有些元素之间本身并不具有可比性), 而是跟元素在序列(或者数组)中的索引相
关. 具体举一个例子会更加清楚.

序列[1, 2, 3] 的全部排列依序为[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1].

序列[3, 2, 1] 的全部排列依序为[3,2,1], [3,1,2], [2,3,1], [2,1,3], [1,3,2], [1,2,3].

上面两个序列之间相同的便是索引的摆列顺序: [0,1,2], [0,2,1], [1,0,2], [1,2,0],
[2,0,1], [2,1,0]

那么如果序列中有相同的元素该如何处理? 仍然跟索引相关, 但有相同的元素便会导致出现
相关的索引. 具体的一个例子如下:

序列[1, 2, 3, 1] 的全部排列依序为:[1,1,2,3], [1,1,3,2], [1,2,1,3], [1,2,3,1],
[1,3,1,2], [1,3,2,1], [2,1,1,3], [2,1,3,1], [2,3,1,1], [3,1,1,2],
[3,1,2,1], [3,2,1,1]
对应的索引排列顺序为: [0,0,1,2], [0,0,2,1], [0,1,0,2],
[0,1,2,0], [0,2,0,1], [0,2,1,0], [1,0,0,3], [1,0,2,0], [1,2,0,0], [2,0,0,1],
[2,0,1,0], [2,1,0,0]

# Cantor Expression
何为康托展开? 对康托展开进行介绍之前, 我这边先介绍一下我们比较熟悉的2进制表示. 我们可以把自然数 x 表示成下面的式子:

<div>$$ x = b_n2^n + b_{n−1}2^{n−1} + ... + b_22^2 + b_1 $$ where $$ 0 \le n \lt 2 $$</div>

与其类似的, 康托展开将会表示成下面的式子:

<div>$$ x = a_nn! + a_{n−1}(n−1)! + ... + a_22! + a_1 $$ where $$ 0 \le a_i \le i $$</div>

# Cantor Expression And nth Permutation

康托展开是如何和第 n 个排列联系到一起的? 考虑[1,2,3]的全排列, [3,1,2] 会是其中的 第几个?

* 第一个数是3, 说明第一个数是1,2的其他组合都是[3,1,2]前面的排列, 也即2 * 2!
* 第二个数是1, 没有其他的排列
* 第三个数是2, 本来1是可以做为小于前面的排列的, 但1已经出现在了第二位了.故也没有其他排列.

也即[3,1,2]前面有 2 * 2! + 0 * 1! = 4个排列, 也即第5个排列.
从上面的例子可以看出, 一个排列可以根据自自己的元素排列推导出相应的康托展开, 而对
应的康托展开计算出来的值便是对应的该排列在全排列中的索引值.

那如何计算第 n 个排列? 比如说[1,2,3,4,5]中的第16个排列是?

* 16 - 1 = 15, 首先去掉自身的1个排列.
* 15 / 4! = 0 * 4! + 15 比第一个数小的数量为0, 故第一个数为1, 剩余下来的排列数还 有15
* 15 / 3! = 2 * 3! + 3 比第二个数小的数量为2, 但1已经出现在第一个数中, 故第二个 数得为4, 才能满足上面的条件. 剩余下来的排列数为3
* 3 / 2! = 1 * 2! + 1 比第三个数小的数量为1, 但1已经出现在第一个数中, 故第三个数 得为3才能满足上面的条件. 声誉下来的排列数为1. 比第一个数小的数量为0, 故第一个数 为1, 剩余下来的排列数还有15
* 1 / 1! = 1 * 1! + 1 比第四个数小的数量为1, 1,3,4 已经出现在前面的排列中了, 故 只剩下5满足情况, (比5小的只剩下1个2). 故第四个数是5. 剩余的排列数为0.
* 第五个数只可能为最后剩下来的2.

## 具体实现

上述介绍了相关原理, 下列出具体的实现如下:
```scala
case class Cantor(size: Int) {
  if (size > 20) {
    throw new Exception("size too big for permutations, should less than 21")
  }

  private var factor = 1l
  private val factors = 1l +: (for (i <- 1 to size) yield {
    factor = i * factor
    factor
  })

  def revert(index: Long): Seq[Int] = {
    var i = index - 1
    val used = Array.fill(size)(false)

    for (j <- 1 to size) yield {
      val a = i / factors(size - j)
      i = i % factors(size - j)
      val selected = (0 until size).filter(!used(_))(a.toInt)
      used(selected) = true
      selected
    }
  }

  def order(arr: Seq[Int]): Long = {
    assert(arr.toSet.size == size)
    assert(arr.forall(_ < size))
    val used = Array.fill(size)(false)
    val coefficient = arr.map { x =>
      used(x) = true
      (0 until x).filter(!used(_)).size
    }
    1 + coefficient.zipWithIndex.foldLeft(0l) { (x, y) =>
      x + y._1 * factors(size - 1 - y._2)
    }
  }
}
```

# next_permutation

有了上面的实现后, 计算 next_permutation 就会变成直接调用接口的问题.

* 先计算该排列的order,
* 计算 order + 1 对应的排列. 注意不能计算最后一个排列的 next_permutation.

## next_permutation 的另一个实现

具体可以见这位同学的这篇文章: <http://fisherlei.blogspot.jp/2012/12/leetcode-next-permutation.html>

我这边简单描述一下算法:

* 从右到左, 寻找第一个违反递增规律的对象(对象对应的索引违反递增规律)j, 也即 idx(j) < idx(j + 1)
* 从右到左, 寻找第第一个索引比 j 大的 i, 也即 idx(i) > idx(j)
* 交换 i, j 对应的对象.
* 将 j 之后的元素 reverse.

可以参照 Scala stdlib 中的 permutation 实现. [SeqLike.scala](https://github.com/scala/scala/blob/2.11.x/src/library/scala/collection/SeqLike.scala#L160)
