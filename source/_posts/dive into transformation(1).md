---
title: Dive into Transformation（一）
tags: 
	- 数学
	- 图形学
	- 前端
categories: 
	- Dive into Transformation
---

## 前言

最近在做的一个项目，其中需要缩放、平移画布的功能，和我们经常使用的 [Excalidraw](https://excalidraw.com/) 和 [draw.io](https://app.diagrams.net/) 的画布平移功能类似，如下图所示：

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/ezgif.com-gif-maker.gif)

由于我使用的是 svg 去做渲染，很容易想到用 transform 去实现画布的平移缩放

我们在平时的常规 UI 开发中也会有需要使用 transform 功能的场景，但是大多都是简单的变换位置，比如使用 rotation 去做动画，或者 translation 去调整元素的位置，稍微高级一些的可能会用到 skew, rotation, translation 的组合来做一些更酷炫的 css3 动画

本以为这是个比较简单的功能，但是稍微了解了一下以后发现里面的水比我想象的深，其中困扰我的有但不仅仅限于以下几个问题：

- transform 的顺序有关系吗?，比如先 scale 再 skew，或先 translate 再 scale
- transform 的 matrix 属性是什么意思？
- 如何以鼠标的位置为 transform-origin 对画布进行平移缩放？
- 如何将鼠标的位置根据画布的 transform 转换到画布上的点？比如画布缩放、平移以后在画布上绘制圆和矩形，如何正确的对应到画布的点上？

这些问题困扰着我，让我感觉它仿佛并不简单，在网上查阅一些资料以后，发现这些问题对应着图形学中的 **坐标变换** 相关知识，想要深入了解和理解这些又不得不接触线性代数，一个接着一个的坑 🥲

本文更多的是自己对于这部分知识的整理和输出，并不一定都是对的，如有错误欢迎指出

## 从一些基础的线性代数概念开始

> [!note]
> 以下标 \* 的标题表示拓展

对于线性代数和矩阵变换我了解的并不太多，很多稍微高级一点的概念因为现阶段用不到只是蜻蜓点水的过了一遍，比如行列式(determinant) 和 特征值/特征向量(eigenvalues/eigenvectors)等，更多的还是聚焦于向量的表示/变换和矩阵的复合等

这里推荐 3blue1brown 的 [Essence of linear algebra - YouTube](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab)，讲的很通俗易懂，我自己的大部分对于线代的理解也来源于这个系列，有条件的同学强烈建议去看一看该教程（线代大佬可以无视 🤣）

> 注：不要偷懒，一定要看看线性代数相关教程

### 向量

在一些其他编程语言，如 C++ 中，就有一种数据类型为 vector，它表示为一个可以动态扩容的数组，而数组的特点就是 **有向的列表**。我们在这里要说的向量和 **有向的列表** 有一定的关联，但又不完全一样

我们可以理解为向量就是 _空间中的有向箭头_，一个向量由其长度和方向决定，和它在空间中的位置无关，只要两个向长度和方向相同，我们就可以认为它们是同一个向量

例如下图中的 $\vec{AE}$ 和 $\vec{CD}$ 就是同一个向量

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/20220509163910.png)

因此，我们可以认为所有的向量的起点都是原点，也方便我们去研究和理解问题

#### 向量的运算

###### 向量乘以常量

一个向量 $\vec{u}$ 乘以一个常量 $\lambda$

- 如果 $\lambda > 0$, 那么结果向量 $\vec{v}$ 的方向和向量 $\hat{u}$ 的方向相同，不过长度变为原来的 $\lambda$ 倍
- 如果 $\lambda < 0$，那么结果向量 $\vec{v}$ 的方向和向量 $\hat{u}$ 的方向相反，长度变为原来的 $|\lambda|$ 倍
- 如果 $\lambda = 0$，那么结果就是 $\vec{0}$

###### 向量加减运算

向量的加法可以用两种方式去理解，例如下面的 $\vec{AB} + \vec{AC}$

第一就是平行四边形，即结果等于以 AB AC 为边所组成的平行四边形的对角线 $\vec{AD}$

第二就是将一个向量的尾部平移至另一个向量的头部，然后连接第一个向量的尾部和第二个向量的头部，就是两个向量的加和，即 $\vec{AD}$

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/20220509201101.png)

这种运算也有其物理意义，我们可以认为向量代表着空间上的位移，两个向量相加所得到的向量就是实际位移的长度

向量的减法也比较简单，例如 $\vec{AB} - \vec{AC}$，我们可以理解为 $\vec{AB} + (- \vec{AC})$

关于向量的点乘和叉乘这里就不过多描述，后面会提到

#### 向量的线性组合，基向量

在上面也提到，$\vec{AD} = \vec{AB} + \vec{AC}$，即该向量可以由 **线性不相关(linearly independent)** 的 $\vec{AB}$ 和 $\vec{AC}$ 组合而成，这种一个向量由两个或多个向量的组合表示的现象，被称为 **线性组合(linear combination)**，用数学公式表达的话，即

$$
\begin{align}
\vec{u} = \alpha \vec{v} + \beta \vec{w}, \space \space \alpha, \beta \in R
\end{align}
$$

> 注： 一组向量线性不相关表示该组向量中的任何一个向量都不能由该组中的其他的向量组合表示

既然向量可以通过线性组合的方式来表示，那我们不妨建立一个二维坐标系，在空间中我们定义两个向量，然后在这个二维坐标系中的任何向量都可以用这两个向量线性组合表示：

$$
\begin{align}
\hat{i} = \begin{bmatrix}
1 \\
0
\end{bmatrix}
\\
\hat{j} = \begin{bmatrix}
0 \\
1
\end{bmatrix}
\end{align}
$$

其中 $\hat{i}$ 表示起点在 (0,0)，终点在 (1, 0) 的向量，$\hat{j}$ 表示起点在 (0,0)，终点在 (0, 1) 的向量，这两个向量我们称其为 **基向量(basis vector)**

例如下图中的几个向量，我们都可以用这两个基向量的组合来表示

$$
\begin{align}
\vec{AB} &= 2\vec{i} + 2\vec{j}
\\
\vec{AE} &= \vec{i} - \vec{j}
\\
\vec{CD} &= 2\vec{i} - 2\vec{j}
\end{align}
$$

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/20220509163910.png)

由 $\{\vec{i},\space\space \vec{j}\}$ 这两个向量线性组合所得到的所有的向量（即整个平面）的空间被称为这一组向量所张成的**空间(span)**

既然我们已经约定俗成了这两个基向量，那么向量的表示就可以不用那么繁琐，直接省略 $\hat{i}$ 和 $\hat{j}$ 了，即

$$
\begin{align}
\vec{AB} &= \begin{bmatrix}
2 \\
2
\end{bmatrix}
\\
\vec{AE} &= \begin{bmatrix}
1 \\
-1
\end{bmatrix}
\\
\vec{CD} &= \begin{bmatrix}
2 \\
-2
\end{bmatrix}
\end{align}
$$

### 线性变换

wiki 对于线性变换的定义如下：

> a **linear map** (also called a **linear mapping**, **linear transformation**, **vector space homomorphism**, or in some contexts **linear function**) is a [mapping](<https://en.wikipedia.org/wiki/Map_(mathematics)> 'Map (mathematics)') between $V \rightarrow W$  between two [vector spaces](https://en.wikipedia.org/wiki/Vector_space) that preserves the operations of [vector addition](https://en.wikipedia.org/wiki/Vector_addition 'Vector addition') and [scalar multiplication](https://en.wikipedia.org/wiki/Scalar_multiplication 'Scalar multiplication').

翻译过来，线性变换就是从向量空间 $V$ 到向量空间 $W$ 的映射函数，同时保留向量的加法和标量乘法，这里有几个关键字：

1. **向量空间** 即包含一组向量的集合，例如下面的向量空间 $V$，就表示一个三维空间的所有向量

$$
\begin{align}
V &= \{v_1, ..., v_n\} \\
v &= \{x,\space y,\space z\}, \space \space x,y,z \in R
\end{align}
$$

2. **映射函数**， 这表示线性变换的本质就是一个函数，和我们在编程中的函数一样，都是接受一个 input，返回一个 output，这里的 input 和 output 都是向量空间中的成员，即向量

到这里，我们可以暂时的将线性变换理解为：<font color='tomato'>线性变换是一个函数，它的输入是一个向量，输出也是一个向量</font>

但是判断一个变换是否为线性变换，也需要满足两个条件：

1. 所有的直线变换后必须还是直线，不能被弯曲
2. 原点必须保持在原位

从上面两点我们也可以得出一个结论：变换后的网格仍然保持平行等距的特性

例如下面这张图中，就是我们常见的缩放的线性变换，向量 $\vec{v}$ 从 $[50, 150]$ 变换到 $[100, 300]$ 的位置

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/Animation.gif)

那么有一个问题，**我们该如何表示线性变换函数？**

我们在上面提到一个向量可以由一组线性不相关的基向量线性组合得到，同时也约定俗成了笛卡尔坐标系下的基向量

$$
\begin{align}
\hat{i} =
\begin{bmatrix}
1 \\
0
\end{bmatrix}
\\
\hat{j} =
\begin{bmatrix}
0 \\
1
\end{bmatrix}
\end{align}
$$

而且我们从线性变换中推导出来的结论 -- 即 **网格线** 仍保持平行等距，我们可以得出另一个结论，即 **变换后的向量线性组合不发生变化**，例如：

$$
\vec{v} = 3\vec{i} + 2\vec{j}
$$

那么在经过线性变换以后，我们可以得到

$$
\vec{v'} = 3 \vec{i'} + 2\vec{j'}
$$

> [!IMPORTANT]
> 我们可以用经过线性变换后的基向量来表示一次线性变换，然后所有的向量都可以由这一组向量线性组合得到

例如我们在上图中的 gif 图中，可以看出是将所有的向量执行了 scale 操作，其基向量变换后的值为

$$
\begin{align}
\vec{i} =
\begin{bmatrix}
2 \\
0
\end{bmatrix}
\\
\vec{j} =
\begin{bmatrix}
0 \\
2
\end{bmatrix}
\end{align}
$$

我们可以再稍微简化一下，将这次线性变换表示为

$$
\begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
$$

左边一列为 $\hat{i}$ ，右边为 $\vec{j}$，学过线性代数的童鞋可能一眼就发现了，这不就是矩阵吗？没错，但是更妙的还在后面，我们还可以套上矩阵的乘法来表示向量变换后的结果以及复合线性变换，例如对于下面的向量

$$
\begin{align}
\vec{v} = \begin{bmatrix}
3 \\
2
\end{bmatrix}
\end{align}
$$

我们对其进行如上的线性变换，可以得到变换后的结果为

$$
\begin{align}
\vec{v'} &= f(\vec{v})
\\\\
&= \begin{bmatrix}
\vec{i'} && \vec{j'} \\
\end{bmatrix}
\begin{bmatrix}
3 \\
2
\end{bmatrix}
\\\\
&= \begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
\begin{bmatrix}
3 \\
2
\end{bmatrix}
\\\\
&=
3 \cdot
\begin{bmatrix}
2 \\
0
\end{bmatrix}
+
2 \cdot
\begin{bmatrix}
0 \\
2
\end{bmatrix}
\\\\
&=
\begin{bmatrix}
3 \cdot 2 + 2 \cdot 0 \\
3 \cdot 0 + 2 \cdot 2
\end{bmatrix}
\\\\
&=
\begin{bmatrix}
6 \\
4
\end{bmatrix}
\end{align}
$$

那么到这里我们就明白了以下几点：

1. 线性变换的性质
2. 线性变换的表示方式
3. 对于向量的线性变换运算方式

#### 复合变换

复合变换就是线性变换的组合，例如我们有两个线性变换 $f, g$，则它们的复合就是

$$
f(g(\vec{v}))
$$

复合变换的结果仍然是一个等价的变换，还是上面那个例子，对一个向量做 g, f 变换以后，可以等价为对这个向量做 h 变换

$$
h(\vec{v}) = f(g(\vec{v}))
$$

例如下面这两张图中，一个是先后进行了两次变换，另一个是只进行了一次变换，但是它们的结果是相同的

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/rotateThenScale.gif)

<center>先是旋转，再进行缩放</center>

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/rotateAndScale.gif)

<center>一次复合变换</center>

那么，我们有办法计算多个线性变换复合后的等价线性变换吗？答案当然是有的

假定我们有以下两个线性变换：

$$
\begin{align}
f &=
\begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
\\\\
g &=
\begin{bmatrix}
\cos(\frac{\pi}{3}) && sin(\frac{\pi}{3}) \\
-\sin(\frac{\pi}{3}) && cos(\frac{\pi}{3})
\end{bmatrix}
\end{align}
$$

容易看出来 f 将向量缩放为原来的两倍，g 为顺时针旋转 60°，当我们对一个向量执行 $f(g(\vec{v}))$ 的操作时，其等价的线性变换是什么？

其实我们直接利用矩阵的乘法，将两个矩阵相乘就可以得到结果

$$
\begin{align}
h &=
\begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
\begin{bmatrix}
\cos(\frac{\pi}{3}) && sin(\frac{\pi}{3}) \\
-\sin(\frac{\pi}{3}) && cos(\frac{\pi}{3})
\end{bmatrix}
\\\\
&=
\begin{bmatrix}
\begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
\begin{bmatrix}
\frac{1}{2}  \\
-\frac{\sqrt{3}}{2}
\end{bmatrix}
&&
\begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
\begin{bmatrix}
\frac{\sqrt{3}}{2} \\
\frac{1}{2}
\end{bmatrix}
\end{bmatrix}
\\\\
&=
\begin{bmatrix}
1 && \sqrt{3}\\
-\sqrt{3} && 1
\end{bmatrix}
\end{align}
$$

这里具体的公式不重要，重要的是我们怎么理解这个公式，或者说这个公式为什么逻辑上讲得通，我们对这个运算过程进行一个抽象

在经过 g 变换以后，原来的 $\vec{i} \rightarrow \vec{i'}$，$\vec{j} \rightarrow \vec{j'}$

$$
\begin{align}
g = \begin{bmatrix}
\vec{i'} && \vec{j'}
\end{bmatrix}
\end{align}
$$

那么我们再次进行 f 变换，就相当于对 g 变换后的基向量进行了再一次变换，即 $\vec{i'} \rightarrow \vec{i''}$，$\vec{j'} \rightarrow \vec{j''}$，得到的 $\vec{i''}, \vec{j''}$ 就是我们最终变换后的结果

$$
\begin{align}
\vec{i''} &= \begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
\begin{bmatrix}
\frac{1}{2}  \\
-\frac{\sqrt{3}}{2}
\end{bmatrix}
\\
\vec{j''} &= \begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
\begin{bmatrix}
\frac{\sqrt{3}}{2} \\
\frac{1}{2}
\end{bmatrix}
\\
h &= \begin{bmatrix}
1 && \sqrt{3}\\
-\sqrt{3} && 1
\end{bmatrix}
\end{align}
$$

#### 逆变换

理解了上面的复合变换，逆变换就很好理解了，逆变换就是一个线性变换的逆。当我们先进行了一次线性变换以后，再进行另一次线性变换以后，如果向量和进行线性变换之前相同，那么就说这两次线性变换互逆

例如对于下面的线性变换

$$
A = \begin{bmatrix}
2 && 0 \\
0 && 2
\end{bmatrix}
$$

我们很容易想到它的逆为

$$
A^{-1} = \begin{bmatrix}
\frac{1}{2} && 0 \\
0 && \frac{1}{2}
\end{bmatrix}
$$

我们将这两次线性变换进行组合，得到的结果为

$$
AA^{-1} = \begin{bmatrix}
1 && 0 \\
0 && 1
\end{bmatrix}
$$

这个结果就是 _identity matrix_，即不发生任何变换的线性变换

我们也可以用以下的公式计算出一个矩阵的逆

$$
\begin{bmatrix}
a && b \\
c && d
\end{bmatrix}
^{-1}
=
\frac{1}{ad-bc}
\begin{bmatrix}
d && -b \\
-c && a
\end{bmatrix}
$$

#### 坐标系的相互转换

上面我们说的基向量的选择是根据我们在笛卡尔坐标系下的约定俗成的基向量，但是我们完全也可以不选择 $\hat{i}$ 和 $\hat{j}$ 作为基向量，而是在空间中另外选择任意两个线性不相关的向量 $\hat{u}$, $\hat{v}$ 作为基向量，其他向量由这两个向量的线性组合得到

我们假定这两个基向量的值为

$$
\begin{align}
\vec{u} =
\begin{bmatrix}
0.6 \\
0.4
\end{bmatrix}
\\
\vec{v} =
\begin{bmatrix}
0.4 \\
-1
\end{bmatrix}
\end{align}
$$

从下面的图里我们可以看到这两组基向量在笛卡尔坐标系下的展示

![](https://floribunda-1303001230.cos.ap-nanjing.myqcloud.com/20220518224927.png)

既然可以使用不同的基向量去表示某一向量，那么问题来了，对于空间中的同一个向量，我们怎么从一个坐标系表示转换到另一个坐标系表示?

例如上图中的 $\vec{a}$，我们可以用两种方式表示它

$$
\begin{align}
\vec{a} &= \vec{u} + \vec{v}
\\
\vec{a} &=
\begin{bmatrix}
1 \\
1
\end{bmatrix}
\\\\
or \space \space \vec{a} &= \vec{i} + (-0.6) \vec{j}
\\
\vec{a} &= \begin{bmatrix}
1 \\
-0.6
\end{bmatrix}
\end{align}
$$

但是到底怎么算出来呢？

其实按照我自己的经验来看，要理解坐标系转换这件事还挺反直觉的，就像十进制和二进制之间的转换一样，我们早已习惯了在十进制下的运算，所以二进制转十进制比较方便，但是一旦反过来，我们就比较难理解其原理，或者换种说法就是很难掰过来自己的惯性思维 🤣

这里提供一个简单的思路，我们先去理解 **如何将非标准坐标系下的向量表示转换为标准坐标系下的向量表示**

要解决这个问题，我们只需要知道非标准坐标系下的 $\hat{u}$ 和 $\hat{v}$ 向量在标准坐标系下的表示是什么即可，例如对于上面的 $\vec{a}$，我们知道它在非标准坐标系下的表示为 ${1\brack 1}$，那么我们就可以将其转化为:

$$
\begin{align}
\vec{a}
&= 1\vec{u} + 1\vec{v}
\\\\
&= 1 \begin{bmatrix}
0.6 \\
0.4
\end{bmatrix}
+
1 \begin{bmatrix}
0.4 \\
-1
\end{bmatrix}
\\\\
&=
\begin{bmatrix}
0.6 + 0.4 \\
0.4 - 1
\end{bmatrix}
\\\\
&=
\begin{bmatrix}
1 \\
-0.6
\end{bmatrix}
\end{align}
$$

这个很好理解，因为就是将非标准坐标系下的基准向量理解为笛卡尔坐标系下的一组普通向量，然后得到这组向量的对应线性组合就行

我们也可以将上述的过程表示为

$$
\begin{align}
\vec{a}
&= \begin{bmatrix}
0.6 && 0.4\\
0.4 && -1
\end{bmatrix}
\begin{bmatrix}
1 \\
1
\end{bmatrix}
\\\\
&= \begin{bmatrix}
1 \\
-0.6
\end{bmatrix}
\end{align}
$$

这里的核心就是两个坐标系的基准向量之间的相互转换，从非标准到标准的转换就是把 $\vec{u}$ 和 $\vec{v}$ 为 $\vec{i}$ 和 $\vec{j}$ 表示

反过来也成立，从标准坐标系到非标准坐标系就是将 $\vec{i}$ 和 $\vec{j}$ 转为 $\vec{u}$ 和 $\vec{v}$，那么我们就可以得到
$\vec{a}$ 在非标准坐标系下的表示为

$$
\begin{align}
\vec{a} &=
\begin{bmatrix}
0.6 && 0.4 \\
0.4 && 1
\end{bmatrix}
^{-1}
\begin{bmatrix}
1 \\
-0.6
\end{bmatrix}
=
\begin{bmatrix}
1 \\
1
\end{bmatrix}
\end{align}
$$

这样我们就可以得到一个公式，假设标准坐标系 $s$ 到非标准坐标系 $n$ 的线性变换为 $A$，那么：

- 非标准 -> 标准
  $$
  \begin{align}
  \vec{v} = A\begin{bmatrix}
  x_{n} \\
  y_{n}
  \end{bmatrix}
  \end{align}
  $$
- 标准 -> 非标准

$$
\begin{align}
\vec{v} = A^{-1}\begin{bmatrix}
x_{s} \\
y_{s}
\end{bmatrix}
\end{align}
$$

到这里线性代数的基本知识就告一段落了，里面还有很多内容本篇没涉及到，例如**行列式(determinant)**，**特征值(eigenvalue)，特征向量(eigenvector)**, **向量内积外积**，**线性方程组** 等，这些都是线代中的很重要的概念，但是由于篇幅原因，以及我们具体的使用场景限制，关于线性代数的内容就介绍到这里，读者如果想深入理解变换请务必*严肃学习！*
