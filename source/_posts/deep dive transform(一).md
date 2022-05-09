---
title: deep dive transform(一)
tags:
  - 数学
  - 技术
---

### 前言

最近在做的一个项目，其中需要缩放、平移画布的功能，和我们经常使用的 [Excalidraw](https://excalidraw.com/) 和 [draw.io](https://app.diagrams.net/) 的画布平移功能类似，如下图所示：

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/ezgif.com-gif-maker.gif)

由于我使用的是 svg 去做渲染，就很容易利用 css 的 transform 去实现画布的平移缩放

我们在平时的常规 UI 开发中也会有需要使用 transform 功能的场景，但是大多都是简单的变换位置，比如使用 rotation 去做动画，或者 translation 去调整元素的位置，稍微高级一些的可能会用到 skew, rotation, translation 的组合来做一些更酷炫的 css3 动画

本以为这是个比较简单的功能，但是稍微了解了一下以后发现里面的水比我想象的深，其中困扰我的有但不仅仅限于以下几个问题：

- transform 的顺序有关系吗?，比如先 scale 再 skew，或先 translate 再 scale
- transform 的 matrix 属性是什么意思？
- 如何以鼠标的位置为 transform-origin 对画布进行平移缩放？
- 如何将鼠标的位置根据画布的 transform 转换到画布上的点？比如画布缩放、平移以后在画布上绘制圆和矩形，能正确的对应到画布的点上？

这些问题困扰着我，让我感觉它仿佛并不简单，在网上查阅一些资料以后，发现这些问题对应着图形学中的 **坐标变换** 相关知识，想要深入了解和理解这些又不得不接触线性代数，又是一个超大的坑 🥲

### 从一些基础的线性代数概念开始

对于线性代数和矩阵变换我了解的并不太多，很多稍微高级一点的概念因为现阶段用不到只是蜻蜓点水的过了一遍，比如行列式(determinant) 和 特征值/特征向量(eigenvalues/eigenvectors)等，更多的还是聚焦于向量的表示/变换和矩阵的复合等

这里推荐 3blue1brown 的 [Essence of linear algebra - YouTube](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab)，讲的很通俗易懂，我自己的大部分对于线代的理解也来源于这个系列

#### 向量

#### 线性变换

####

### 仿射变换和齐次坐标系
