---
title: Dijkstra算法实现要点
date: 2016-06-13 08:12:19
tags: 算法
---

使用一个hashmap存储每个节点到起始节点的不断更新后的最小距离值

使用一个hashmap存储上面表中更新后的最小距离值所对应的前一个节点

使用一个hashset存储settled节点

使用一个hashset存储unsettled节点

reference: [Dijkstra's shortest path algorithm in Java](http://www.vogella.com/tutorials/JavaAlgorithmsDijkstra/article.html)

