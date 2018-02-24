---
layout: post
title: Matching Squares 算法简介
categories: [algorithm]
description: 绘制等值线的主要算法
keywords: Matching Squares, Isoline, contour line
---

以下内容主要基于 [Wikipedia](https://en.wikipedia.org/wiki/Marching_squares).

Matching Squares 算法是一种计算机图形学算法。它可以根据给定的二维数组，生成相应的等值线。等值线也分两种：值位于线上，称为 isolines; 值位于线与线之间的区域，称为 isobands。经典的应用是绘制地形图，以及天气等值线图等。

与 Matching Squares 类似的是 [Marching cubes](https://en.wikipedia.org/wiki/Marching_cubes)，它可以根据给定的三维体，提取等值面的多边形网格。

步骤如下:

- 逐一遍历 grid 中每一个 cell.
- 通过比较 cell corner 的值与 contour level，计算出 cell index.
- 构建一个 lookup table, 以 cell index 为 key，描述该 cell 的 geometry.
- 沿着 cell 的边界，用线性插值计算等值线的确切位置.

## Basic Algorithm

一图胜千言:

![marching_squares_algorithm](/images/Marching_squares_algorithm.svg)

稍作解释:

### 第一步（二值化）

以 isovalue 为阈值，将二维数组二值化，生成一幅 binary image:

- 大于 isovalue 的为 1
- 小于 isovalue 的为 0

### 第二步（to cell)

binary image 上每一个 2x2 的像素块代表着一个 cell。可见上图中绿色网格部分。可以看出，这个 cell 构成的 grid 要比之前二维数组的 grid 小一些。

### 第三部（给 cell 编号）

开始遍历每一个 cell，并根据 cell 的四个角的真值，给出其 index。

而 Look-up table 实际上是按照**顺时针**读取四个角的真值而组成的二进制数值，如 index-2 就是 0010，仅有右下角为真。

### 第四步（给 cell 画线）

根据 index 读取 Look-up table 里的 geometry，绘制在每个 cell 中，形成一个大概的等值线图。

### 第五步（细化 cell）

最终要根据实际数值，进行线性插值，来确定每一根线到底画在哪里。

## 鞍点的处理

细心一点可以发现上图中 case-5 与 case-10 实际上是有歧义的。这两个 case 实际上拥有两种可能的画法。一个比较常规的解决方法是再根据 cell 的四个顶点的值计算一次平均。确定中心点的真值。如下图所示:

![marching-squares-isoline](/images/Marching-squares-isoline.png)