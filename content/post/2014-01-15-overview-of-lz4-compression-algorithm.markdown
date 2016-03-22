---
comments: true
date: 2014-01-15T22:00:08Z
tags: ['compression', 'tech']
title: overview of lz4 compression algorithm
url: /blog/2014/01/15/overview-of-lz4-compression-algorithm/
---

# lz4介绍
   
Google Code项目上的介绍文字.
{% blockquote Yann Collet http://code.google.com/p/lz4/ lz4介绍 %}
LZ4 is a very fast lossless compression algorithm, providing compression speed at 400 MB/s per core, scalable with multi-cores CPU. It also features an extremely fast decoder, with speed in multiple GB/s per core, typically reaching RAM speed limits on multi-core systems. 
{% endblockquote %}

lz4是一个非常快速的无损压缩算法,单核压缩能达到400M/s,对多核cpu的性能扩展也很好.同时lz4的解压缩实现也是极速,单核能够达到GB/s的速度,对于多核的系统来说能够达到内存的极限.

# lz4各个语言的实现
lz4最初的实现是用c来实现的.但除了c之外,其他人也实现了包括下面语言的lz4压缩.更多的见google code项目的说明

1. Java : by Adrien Grand, at [la4-java](https://github.com/jpountz/lz4-java)
2. C++11 multi-threads : by Takayuki Matsuoka, at [lz4mt]( https://github.com/t-mat/lz4mt )
3. Python : by Steeve Morin, at [py-lz4]( http://pypi.python.org/pypi/lz4 )
4. Ruby : by Komiya Atsushi, at [lz4-ryby]( http://rubygems.org/gems/lz4-ruby )

基本上主流语言都有对lz4压缩算法的实现,这个对于后续的应用开发带来了很大的便利.

# lz4的具体实现

这个暂时没精力覆盖,不过网上已经有网友对lz4的实现有较深入的剖析.具体见下面3篇文章.

1. [速度之王 — LZ4压缩算法(一)](http://blog.csdn.net/zhangskd/article/details/17009111)
2. [速度之王 — LZ4压缩算法(二)](http://blog.csdn.net/zhangskd/article/details/17282895)
3. [速度之王 — LZ4压缩算法(三)](http://blog.csdn.net/zhangskd/article/details/17283875)

# lz4的应用场景

1. 操作系统内核镜像的压缩和解压缩
2. 对延迟敏感的大数据存储计算系统,比如hadoop
3. 视频的流化.
4. 替换目前浏览器的gzip压缩
