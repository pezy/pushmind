---
layout: post
title: Octree and K-d tree
categories: [algorithm]
description: 分析八叉树与 K-d 树的异同
keywords: Octree, K-d tree, OSG
---

去年曾经写过[一篇文章](./octree-note)来阐述利用 OpenSceneGrahp 如何实现 Octree.
现在想想, 发现此货太干, 也过分偏近实用, 总希望从理论上在讲述一下 Octree.

后来偶然发现[知乎上](http://www.zhihu.com/question/25111128)针对这个话题, 曾经有一场讨论.
而答案里, 曾出现有人混淆 Octree 和 3-d tree(k-d tree) 的区别. 工作中, 同事貌似也不做区分, 认为原理类似.

可经过我的实践与观察, 发现差别还是很大的.

-----

首先 Octree 普遍翻译为八叉树，而其在二维空间中表现为 Quadtree, 即四叉树。故合称为 `QO trees`.

其次，无论四叉、八叉树，还是 KD 树，都是 box tree.

最后，QO trees 与 KD tree, 最大的区别在于：

1. 前者基于有限空间(finite-sized box), 而后者却是无限空间(near-infinite box).
1. 面对一个子空间, 前者一刀4/8份(全维度切割),后者一刀2份(单一维度切割).
