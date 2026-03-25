---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
translation:
  title: NumPy
  headings:
    Overview: 概述
    NumPy Arrays: NumPy 数组
    NumPy Arrays::Basics: 基础
    NumPy Arrays::Shape and Dimension: 形状与维度
    NumPy Arrays::Creating Arrays: 创建数组
    NumPy Arrays::Array Indexing: 数组索引
    NumPy Arrays::Array Methods: 数组方法
    Arithmetic Operations: 算术运算
    Matrix Multiplication: 矩阵乘法
    Broadcasting: 广播
    Mutability and Copying Arrays: 可变性与数组复制
    Mutability and Copying Arrays::Mutability: 可变性
    Mutability and Copying Arrays::Making Copies: 复制数组
    Additional Features: 其他功能
    Additional Features::Universal Functions: 通用函数
    Additional Features::Comparisons: 比较
    Additional Features::Sub-packages: 子包
    Additional Features::Implicit Multithreading: 隐式多线程
    Exercises: 练习
---

(np)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`NumPy <single: NumPy>`

```{index} single: Python; NumPy
```

```{epigraph}
"让我们说清楚：科学工作与共识毫无关系。共识是政治的事务。相反，科学只需要一个恰好正确的研究者，这意味着他或她的结果可以通过参照现实世界加以验证。在科学中，共识是无关紧要的。重要的是可重复的结果。" —— 迈克尔·克莱顿
```

除了 Anaconda 中已有的内容外，本讲座还需要以下库：

```{code-cell} ipython3
:tags: [hide-output]

!pip install quantecon
```

## 概述

[NumPy](https://en.wikipedia.org/wiki/NumPy) 是一个一流的数值编程库

* 在学术界、金融界和工业界广泛使用。
* 成熟、快速、稳定，并持续开发中。

在前几讲中，我们已经看到了一些涉及 NumPy 的代码。

在本讲座中，我们将开始对以下内容进行更系统的讨论：

1. NumPy 数组，以及
1. NumPy 提供的基本数组处理操作。


（有关备选参考资料，请参阅 [NumPy 官方文档](https://numpy.org/doc/stable/reference/)。）

我们将使用以下导入语句。

```{code-cell} python3
import numpy as np
import random
import quantecon as qe
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n
from mpl_toolkits.mplot3d.axes3d import Axes3D
from matplotlib import cm
```



(numpy_array)=
## NumPy 数组

```{index} single: NumPy; Arrays
```

NumPy 解决的核心问题是快速数组处理。

NumPy 定义的最重要结构是一种数组数据类型，正式称为 [numpy.ndarray](https://numpy.org/doc/stable/reference/arrays.ndarray.html)。

NumPy 数组支撑着科学 Python 生态系统的绝大部分。

### 基础

要创建一个只包含零的 NumPy 数组，我们使用 [np.zeros](https://numpy.org/doc/stable/reference/generated/numpy.zeros.html#numpy.zeros)

```{code-cell} python3
a = np.zeros(3)
a
```

```{code-cell} python3
type(a)
```

NumPy 数组与原生 Python 列表有些类似，但有以下区别：

* 数据*必须是同质的*（所有元素具有相同的类型）。
* 这些类型必须是 NumPy 提供的[数据类型](https://numpy.org/doc/stable/reference/arrays.dtypes.html)（`dtypes`）之一。

这些 dtypes 中最重要的是：

* float64：64 位浮点数
* int64：64 位整数
* bool：8 位真或假

还有用于表示复数、无符号整数等的 dtypes。

在现代机器上，数组的默认 dtype 是 `float64`

```{code-cell} python3
a = np.zeros(3)
type(a[0])
```

如果我们想使用整数，可以按如下方式指定：

```{code-cell} python3
a = np.zeros(3, dtype=int)
type(a[0])
```

(numpy_shape_dim)=
### 形状与维度

```{index} single: NumPy; Arrays (Shape and Dimension)
```

考虑以下赋值语句

```{code-cell} python3
z = np.zeros(10)
```

这里 `z` 是一个**扁平**数组——既不是行向量也不是列向量。

```{code-cell} python3
z.shape
```

这里形状元组只有一个元素，即数组的长度（只有一个元素的元组以逗号结尾）。

要给它添加一个额外的维度，我们可以修改 `shape` 属性

```{code-cell} python3
z.shape = (10, 1)   # 将扁平数组转换为列向量（二维）
z
```

```{code-cell} python3
z = np.zeros(4)     # 扁平数组
z.shape = (2, 2)    # 二维数组
z
```

在最后一种情况下，要生成 2x2 数组，我们也可以将元组传递给 `zeros()` 函数，如 `z = np.zeros((2, 2))`。



(creating_arrays)=
### 创建数组

```{index} single: NumPy; Arrays (Creating)
```

正如我们所见，`np.zeros` 函数创建一个零数组。

你大概能猜到 `np.ones` 创建什么。

与之相关的是 `np.empty`，它在内存中创建数组，以便稍后用数据填充

```{code-cell} python3
z = np.empty(3)
z
```

这里你看到的数字是垃圾值。

（Python 分配了 3 个连续的 64 位内存块，这些内存槽中现有的内容被解释为 `float64` 值）

要设置等间距数字的网格，使用 `np.linspace`

```{code-cell} python3
z = np.linspace(2, 4, 5)  # 从 2 到 4，共 5 个元素
```

要创建单位矩阵，使用 `np.identity` 或 `np.eye`

```{code-cell} python3
z = np.identity(2)
z
```

此外，NumPy 数组可以从 Python 列表、元组等使用 `np.array` 创建

```{code-cell} python3
z = np.array([10, 20])                 # 从 Python 列表创建 ndarray
z
```

```{code-cell} python3
type(z)
```

```{code-cell} python3
z = np.array((10, 20), dtype=float)    # 这里 'float' 等价于 'np.float64'
z
```

```{code-cell} python3
z = np.array([[1, 2], [3, 4]])         # 从列表的列表创建 2D 数组
z
```

另请参阅 `np.asarray`，它执行类似的功能，但不会对已在 NumPy 数组中的数据进行独立复制。

要从包含数值数据的文本文件中读取数组数据，使用 `np.loadtxt`——详情请参阅[文档](https://numpy.org/doc/stable/reference/routines.io.html)。



### 数组索引

```{index} single: NumPy; Arrays (Indexing)
```

对于扁平数组，索引与 Python 序列相同：

```{code-cell} python3
z = np.linspace(1, 2, 5)
z
```

```{code-cell} python3
z[0]
```

```{code-cell} python3
z[0:2]  # 两个元素，从元素 0 开始
```

```{code-cell} python3
z[-1]
```

对于 2D 数组，索引语法如下：

```{code-cell} python3
z = np.array([[1, 2], [3, 4]])
z
```

```{code-cell} python3
z[0, 0]
```

```{code-cell} python3
z[0, 1]
```

以此类推。

列和行可以按如下方式提取

```{code-cell} python3
z[0, :]
```

```{code-cell} python3
z[:, 1]
```

整数类型的 NumPy 数组也可用于提取元素

```{code-cell} python3
z = np.linspace(2, 4, 5)
z
```

```{code-cell} python3
indices = np.array((0, 2, 3))
z[indices]
```

最后，`dtype` 为 `bool` 的数组可用于提取元素

```{code-cell} python3
z
```

```{code-cell} python3
d = np.array([0, 1, 1, 0, 0], dtype=bool)
d
```

```{code-cell} python3
z[d]
```

我们将在下面看到为什么这很有用。

顺便说一句：可以使用切片符号将数组的所有元素设置为一个数字

```{code-cell} python3
z = np.empty(3)
z
```

```{code-cell} python3
z[:] = 42
z
```

### 数组方法

```{index} single: NumPy; Arrays (Methods)
```

数组有很多有用的方法，所有这些方法都经过精心优化

```{code-cell} python3
a = np.array((4, 3, 2, 1))
a
```

```{code-cell} python3
a.sort()              # 原地排序 a
a
```

```{code-cell} python3
a.sum()               # 求和
```

```{code-cell} python3
a.mean()              # 均值
```

```{code-cell} python3
a.max()               # 最大值
```

```{code-cell} python3
a.argmax()            # 返回最大元素的索引
```

```{code-cell} python3
a.cumsum()            # a 元素的累积和
```

```{code-cell} python3
a.cumprod()           # a 元素的累积积
```

```{code-cell} python3
a.var()               # 方差
```

```{code-cell} python3
a.std()               # 标准差
```

```{code-cell} python3
a.shape = (2, 2)
a.T                   # 等价于 a.transpose()
```

另一个值得了解的方法是 `searchsorted()`。

如果 `z` 是一个非递减数组，那么 `z.searchsorted(a)` 返回 `z` 中第一个 `>= a` 的元素的索引

```{code-cell} python3
z = np.linspace(2, 4, 5)
z
```

```{code-cell} python3
z.searchsorted(2.2)
```


## 算术运算

```{index} single: NumPy; Arithmetic Operations
```

运算符 `+`、`-`、`*`、`/` 和 `**` 都对数组*逐元素*作用

```{code-cell} python3
a = np.array([1, 2, 3, 4])
b = np.array([5, 6, 7, 8])
a + b
```

```{code-cell} python3
a * b
```

我们可以向每个元素加一个标量，如下所示

```{code-cell} python3
a + 10
```

标量乘法类似

```{code-cell} python3
a * 10
```

二维数组遵循相同的一般规则

```{code-cell} python3
A = np.ones((2, 2))
B = np.ones((2, 2))
A + B
```

```{code-cell} python3
A + 10
```

```{code-cell} python3
A * B
```

(numpy_matrix_multiplication)=
特别地，`A * B` *不是*矩阵乘积，而是逐元素乘积。


## 矩阵乘法

```{index} single: NumPy; Matrix Multiplication
```

```{index} single: NumPy; Matrix Multiplication
```

我们使用 `@` 符号进行矩阵乘法，如下所示：

```{code-cell} python3
A = np.ones((2, 2))
B = np.ones((2, 2))
A @ B
```

该语法也适用于扁平数组——NumPy 会对你的意图做出合理猜测：

```{code-cell} python3
A @ (0, 1)
```

由于我们是右乘，元组被视为列向量。



(broadcasting)=
## 广播

```{index} single: NumPy; Broadcasting
```

（本节扩展了 [Jake VanderPlas](https://jakevdp.github.io/PythonDataScienceHandbook/02.05-computation-on-arrays-broadcasting.html) 提供的关于广播的精彩讨论。）

```{note}
广播是 NumPy 非常重要的特性。同时，高级广播相对复杂，以下的一些细节在第一次阅读时可以略读。
```

在逐元素操作中，数组可能没有相同的形状。

当发生这种情况时，NumPy 将在可能的情况下自动将数组扩展为相同的形状。

NumPy 中这种有用（但有时令人困惑）的特性称为**广播**。

广播的价值在于：

* 可以避免 `for` 循环，这有助于数值代码快速运行，以及
* 广播可以让我们在不实际在内存中创建某些维度的情况下对数组执行操作，这在数组很大时非常重要。

例如，假设 `a` 是一个 $3 \times 3$ 数组（`a -> (3, 3)`），而 `b` 是一个有三个元素的扁平数组（`b -> (3,)`）。

当将它们相加时，NumPy 将自动将 `b -> (3,)` 扩展为 `b -> (3, 3)`。

逐元素加法将产生一个 $3 \times 3$ 数组

```{code-cell} python3

a = np.array(
        [[1, 2, 3], 
         [4, 5, 6], 
         [7, 8, 9]])
b = np.array([3, 6, 9])

a + b
```

以下是该广播操作的可视化表示：

```{code-cell} python3
---
tags: [hide-input]
---
# 改编自 Jake VanderPlas 书中的代码（见 https://jakevdp.github.io/PythonDataScienceHandbook/06.00-figure-code.html#Broadcasting）
# 原始来源于 astroML：见 https://www.astroml.org/book_figures/appendix/fig_broadcast_visual.html


def draw_cube(ax, xy, size, depth=0.4,
              edges=None, label=None, label_kwargs=None, **kwargs):
    """draw and label a cube.  edges is a list of numbers between
    1 and 12, specifying which of the 12 cube edges to draw"""
    if edges is None:
        edges = range(1, 13)

    x, y = xy

    if 1 in edges:
        ax.plot([x, x + size],
                [y + size, y + size], **kwargs)
    if 2 in edges:
        ax.plot([x + size, x + size],
                [y, y + size], **kwargs)
    if 3 in edges:
        ax.plot([x, x + size],
                [y, y], **kwargs)
    if 4 in edges:
        ax.plot([x, x],
                [y, y + size], **kwargs)

    if 5 in edges:
        ax.plot([x, x + depth],
                [y + size, y + depth + size], **kwargs)
    if 6 in edges:
        ax.plot([x + size, x + size + depth],
                [y + size, y + depth + size], **kwargs)
    if 7 in edges:
        ax.plot([x + size, x + size + depth],
                [y, y + depth], **kwargs)
    if 8 in edges:
        ax.plot([x, x + depth],
                [y, y + depth], **kwargs)

    if 9 in edges:
        ax.plot([x + depth, x + depth + size],
                [y + depth + size, y + depth + size], **kwargs)
    if 10 in edges:
        ax.plot([x + depth + size, x + depth + size],
                [y + depth, y + depth + size], **kwargs)
    if 11 in edges:
        ax.plot([x + depth, x + depth + size],
                [y + depth, y + depth], **kwargs)
    if 12 in edges:
        ax.plot([x + depth, x + depth],
                [y + depth, y + depth + size], **kwargs)

    if label:
        if label_kwargs is None:
            label_kwargs = {}
        ax.text(x + 0.5 * size, y + 0.5 * size, label,
                ha='center', va='center', **label_kwargs)

solid = dict(c='black', ls='-', lw=1,
             label_kwargs=dict(color='k'))
dotted = dict(c='black', ls='-', lw=0.5, alpha=0.5,
              label_kwargs=dict(color='gray'))
depth = 0.3

# 绘制无边框的图形和坐标轴
fig = plt.figure(figsize=(5, 1), facecolor='w')
ax = plt.axes([0, 0, 1, 1], xticks=[], yticks=[], frameon=False)

# 第一个块
draw_cube(ax, (1, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '1', **solid)
draw_cube(ax, (2, 7.5), 1, depth, [1, 2, 3, 6, 9], '2', **solid)
draw_cube(ax, (3, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '3', **solid)

draw_cube(ax, (1, 6.5), 1, depth, [2, 3, 4], '4', **solid)
draw_cube(ax, (2, 6.5), 1, depth, [2, 3], '5', **solid)
draw_cube(ax, (3, 6.5), 1, depth, [2, 3, 7, 10], '6', **solid)

draw_cube(ax, (1, 5.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (2, 5.5), 1, depth, [2, 3], '8', **solid)
draw_cube(ax, (3, 5.5), 1, depth, [2, 3, 7, 10], '9', **solid)

# 第二个块
draw_cube(ax, (6, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '3', **solid)
draw_cube(ax, (7, 7.5), 1, depth, [1, 2, 3, 6, 9], '6', **solid)
draw_cube(ax, (8, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '9', **solid)

draw_cube(ax, (6, 6.5), 1, depth, range(2, 13), '3', **dotted)
draw_cube(ax, (7, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)
draw_cube(ax, (8, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '9', **dotted)

draw_cube(ax, (6, 5.5), 1, depth, [2, 3, 4, 7, 8, 10, 11, 12], '3', **dotted)
draw_cube(ax, (7, 5.5), 1, depth, [2, 3, 7, 10, 11], '6', **dotted)
draw_cube(ax, (8, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)

# 第三个块
draw_cube(ax, (12, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '4', **solid)
draw_cube(ax, (13, 7.5), 1, depth, [1, 2, 3, 6, 9], '8', **solid)
draw_cube(ax, (14, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '12', **solid)

draw_cube(ax, (12, 6.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (13, 6.5), 1, depth, [2, 3], '11', **solid)
draw_cube(ax, (14, 6.5), 1, depth, [2, 3, 7, 10], '15', **solid)

draw_cube(ax, (12, 5.5), 1, depth, [2, 3, 4], '10', **solid)
draw_cube(ax, (13, 5.5), 1, depth, [2, 3], '14', **solid)
draw_cube(ax, (14, 5.5), 1, depth, [2, 3, 7, 10], '18', **solid)

ax.text(5, 7.0, '+', size=12, ha='center', va='center')
ax.text(10.5, 7.0, '=', size=12, ha='center', va='center');
```

那么 `b -> (3, 1)` 怎么样？

在这种情况下，NumPy 将自动将 `b -> (3, 1)` 扩展为 `b -> (3, 3)`。

逐元素加法将产生一个 $3 \times 3$ 矩阵

```{code-cell} python3
b.shape = (3, 1)

a + b
```

以下是该广播操作的可视化表示：

```{code-cell} python3
---
tags: [hide-input]
---

fig = plt.figure(figsize=(5, 1), facecolor='w')
ax = plt.axes([0, 0, 1, 1], xticks=[], yticks=[], frameon=False)

# 第一个块
draw_cube(ax, (1, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '1', **solid)
draw_cube(ax, (2, 7.5), 1, depth, [1, 2, 3, 6, 9], '2', **solid)
draw_cube(ax, (3, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '3', **solid)

draw_cube(ax, (1, 6.5), 1, depth, [2, 3, 4], '4', **solid)
draw_cube(ax, (2, 6.5), 1, depth, [2, 3], '5', **solid)
draw_cube(ax, (3, 6.5), 1, depth, [2, 3, 7, 10], '6', **solid)

draw_cube(ax, (1, 5.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (2, 5.5), 1, depth, [2, 3], '8', **solid)
draw_cube(ax, (3, 5.5), 1, depth, [2, 3, 7, 10], '9', **solid)

# 第二个块
draw_cube(ax, (6, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 7, 9, 10], '3', **solid)
draw_cube(ax, (7, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '3', **dotted)
draw_cube(ax, (8, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '3', **dotted)

draw_cube(ax, (6, 6.5), 1, depth, [2, 3, 4, 7, 10], '6', **solid)
draw_cube(ax, (7, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)
draw_cube(ax, (8, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)

draw_cube(ax, (6, 5.5), 1, depth, [2, 3, 4, 7, 10], '9', **solid)
draw_cube(ax, (7, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)
draw_cube(ax, (8, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)

# 第三个块
draw_cube(ax, (12, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '4', **solid)
draw_cube(ax, (13, 7.5), 1, depth, [1, 2, 3, 6, 9], '5', **solid)
draw_cube(ax, (14, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '6', **solid)

draw_cube(ax, (12, 6.5), 1, depth, [2, 3, 4], '10', **solid)
draw_cube(ax, (13, 6.5), 1, depth, [2, 3], '11', **solid)
draw_cube(ax, (14, 6.5), 1, depth, [2, 3, 7, 10], '12', **solid)

draw_cube(ax, (12, 5.5), 1, depth, [2, 3, 4], '16', **solid)
draw_cube(ax, (13, 5.5), 1, depth, [2, 3], '17', **solid)
draw_cube(ax, (14, 5.5), 1, depth, [2, 3, 7, 10], '18', **solid)

ax.text(5, 7.0, '+', size=12, ha='center', va='center')
ax.text(10.5, 7.0, '=', size=12, ha='center', va='center');


```

在某些情况下，两个操作数都会被扩展。

当我们有 `a -> (3,)` 和 `b -> (3, 1)` 时，`a` 将被扩展为 `a -> (3, 3)`，`b` 将被扩展为 `b -> (3, 3)`。

在这种情况下，逐元素加法将产生一个 $3 \times 3$ 矩阵

```{code-cell} python3
a = np.array([3, 6, 9])
b = np.array([2, 3, 4])
b.shape = (3, 1)

a + b
```

以下是该广播操作的可视化表示：

```{code-cell} python3
---
tags: [hide-input]
---

# 绘制无边框的图形和坐标轴
fig = plt.figure(figsize=(5, 1), facecolor='w')
ax = plt.axes([0, 0, 1, 1], xticks=[], yticks=[], frameon=False)

# 第一个块
draw_cube(ax, (1, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '3', **solid)
draw_cube(ax, (2, 7.5), 1, depth, [1, 2, 3, 6, 9], '6', **solid)
draw_cube(ax, (3, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '9', **solid)

draw_cube(ax, (1, 6.5), 1, depth, range(2, 13), '3', **dotted)
draw_cube(ax, (2, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)
draw_cube(ax, (3, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '9', **dotted)

draw_cube(ax, (1, 5.5), 1, depth, [2, 3, 4, 7, 8, 10, 11, 12], '3', **dotted)
draw_cube(ax, (2, 5.5), 1, depth, [2, 3, 7, 10, 11], '6', **dotted)
draw_cube(ax, (3, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)

# 第二个块
draw_cube(ax, (6, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 7, 9, 10], '2', **solid)
draw_cube(ax, (7, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '2', **dotted)
draw_cube(ax, (8, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '2', **dotted)

draw_cube(ax, (6, 6.5), 1, depth, [2, 3, 4, 7, 10], '3', **solid)
draw_cube(ax, (7, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '3', **dotted)
draw_cube(ax, (8, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '3', **dotted)

draw_cube(ax, (6, 5.5), 1, depth, [2, 3, 4, 7, 10], '4', **solid)
draw_cube(ax, (7, 5.5), 1, depth, [2, 3, 7, 10, 11], '4', **dotted)
draw_cube(ax, (8, 5.5), 1, depth, [2, 3, 7, 10, 11], '4', **dotted)

# 第三个块
draw_cube(ax, (12, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '5', **solid)
draw_cube(ax, (13, 7.5), 1, depth, [1, 2, 3, 6, 9], '8', **solid)
draw_cube(ax, (14, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '11', **solid)

draw_cube(ax, (12, 6.5), 1, depth, [2, 3, 4], '6', **solid)
draw_cube(ax, (13, 6.5), 1, depth, [2, 3], '9', **solid)
draw_cube(ax, (14, 6.5), 1, depth, [2, 3, 7, 10], '12', **solid)

draw_cube(ax, (12, 5.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (13, 5.5), 1, depth, [2, 3], '10', **solid)
draw_cube(ax, (14, 5.5), 1, depth, [2, 3, 7, 10], '13', **solid)

ax.text(5, 7.0, '+', size=12, ha='center', va='center')
ax.text(10.5, 7.0, '=', size=12, ha='center', va='center');
```

虽然广播非常有用，但有时可能会令人困惑。

例如，让我们尝试将 `a -> (3, 2)` 和 `b -> (3,)` 相加。

```{code-cell} python3
---
tags: [raises-exception]
---
a = np.array(
      [[1, 2],
       [4, 5],
       [7, 8]])
b = np.array([3, 6, 9])

a + b
```

`ValueError` 告诉我们操作数无法一起广播。


以下是一个可视化表示，说明为什么该广播无法执行：

```{code-cell} python3
---
tags: [hide-input]
---
# 绘制无边框的图形和坐标轴
fig = plt.figure(figsize=(3, 1.3), facecolor='w')
ax = plt.axes([0, 0, 1, 1], xticks=[], yticks=[], frameon=False)

# 第一个块
draw_cube(ax, (1, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '1', **solid)
draw_cube(ax, (2, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '2', **solid)

draw_cube(ax, (1, 6.5), 1, depth, [2, 3, 4], '4', **solid)
draw_cube(ax, (2, 6.5), 1, depth, [2, 3, 7, 10], '5', **solid)

draw_cube(ax, (1, 5.5), 1, depth, [2, 3, 4], '7', **solid)
draw_cube(ax, (2, 5.5), 1, depth, [2, 3, 7, 10], '8', **solid)

# 第二个块
draw_cube(ax, (6, 7.5), 1, depth, [1, 2, 3, 4, 5, 6, 9], '3', **solid)
draw_cube(ax, (7, 7.5), 1, depth, [1, 2, 3, 6, 9], '6', **solid)
draw_cube(ax, (8, 7.5), 1, depth, [1, 2, 3, 6, 7, 9, 10], '9', **solid)

draw_cube(ax, (6, 6.5), 1, depth, range(2, 13), '3', **dotted)
draw_cube(ax, (7, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '6', **dotted)
draw_cube(ax, (8, 6.5), 1, depth, [2, 3, 6, 7, 9, 10, 11], '9', **dotted)

draw_cube(ax, (6, 5.5), 1, depth, [2, 3, 4, 7, 8, 10, 11, 12], '3', **dotted)
draw_cube(ax, (7, 5.5), 1, depth, [2, 3, 7, 10, 11], '6', **dotted)
draw_cube(ax, (8, 5.5), 1, depth, [2, 3, 7, 10, 11], '9', **dotted)


ax.text(4.5, 7.0, '+', size=12, ha='center', va='center')
ax.text(10, 7.0, '=', size=12, ha='center', va='center')
ax.text(11, 7.0, '?', size=16, ha='center', va='center');
```

我们可以看到 NumPy 无法将数组扩展为相同的大小。

这是因为当 `b` 从 `b -> (3,)` 扩展到 `b -> (3, 3)` 时，NumPy 无法将 `b` 与 `a -> (3, 2)` 匹配。

当我们移动到更高维度时，情况会变得更加复杂。

为了帮助我们理解，可以使用以下规则列表：

* *第一步*：当两个数组的维度不匹配时，NumPy 将通过在现有维度左侧添加维度来扩展维度较少的那个。
    - 例如，如果 `a -> (3, 3)` 且 `b -> (3,)`，则广播将在左侧添加一个维度，使得 `b -> (1, 3)`；
    - 如果 `a -> (2, 2, 2)` 且 `b -> (2, 2)`，则广播将在左侧添加一个维度，使得 `b -> (1, 2, 2)`；
    - 如果 `a -> (3, 2, 2)` 且 `b -> (2,)`，则广播将在左侧添加两个维度，使得 `b -> (1, 1, 2)`（你也可以将此过程视为经历了*第一步*两次）。


* *第二步*：当两个数组具有相同维度但不同形状时，NumPy 将尝试扩展形状索引为 1 的维度。
    - 例如，如果 `a -> (1, 3)` 且 `b -> (3, 1)`，则广播将扩展 `a` 和 `b` 中形状为 1 的维度，使得 `a -> (3, 3)` 且 `b -> (3, 3)`；
    - 如果 `a -> (2, 2, 2)` 且 `b -> (1, 2, 2)`，则广播将扩展 `b` 的第一个维度，使得 `b -> (2, 2, 2)`；
    - 如果 `a -> (3, 2, 2)` 且 `b -> (1, 1, 2)`，则广播将在所有形状为 1 的维度上扩展 `b`，使得 `b -> (3, 2, 2)`。

* *第三步*：经过第一步和第二步后，如果两个数组仍然不匹配，将引发 `ValueError`。例如，假设 `a -> (2, 2, 3)` 且 `b -> (2, 2)`
    - 通过*第一步*，`b` 将被扩展为 `b -> (1, 2, 2)`；
    - 通过*第二步*，`b` 将被扩展为 `b -> (2, 2, 2)`；
    - 我们可以看到经过前两步后它们仍然不匹配。因此，将引发 `ValueError`



## 可变性与数组复制

NumPy 数组是可变数据类型，类似于 Python 列表。

换句话说，它们的内容可以在初始化后在内存中被更改（变异）。

这很方便，但与 Python 的命名和引用模型结合时，可能会导致 NumPy 初学者犯错误。

在本节中，我们回顾一些关键问题。


### 可变性

我们在上面已经看到了可变性的例子。

以下是 NumPy 数组变异的另一个例子

```{code-cell} python3
a = np.array([42, 44])
a
```

```{code-cell} python3
a[-1] = 0  # 将最后一个元素改为 0
a
```

可变性导致以下行为（这可能会让 MATLAB 程序员感到震惊……）

```{code-cell} python3
a = np.random.randn(3)
a
```

```{code-cell} python3
b = a
b[0] = 0.0
a
```

发生的情况是我们通过修改 `b` 改变了 `a`。

名称 `b` 绑定到 `a`，成为该数组的另一个引用（Python 赋值模型在{doc}`课程后面 <python_advanced_features>`有更详细的描述）。

因此，它有同等权利对该数组进行更改。

这实际上是最合理的默认行为！

这意味着我们只传递数据指针，而不是复制数据。

复制在速度和内存方面都是昂贵的。

### 复制数组

当然，在需要时可以使 `b` 成为 `a` 的独立副本。

这可以使用 `np.copy` 来完成

```{code-cell} python3
a = np.random.randn(3)
a
```

```{code-cell} python3
b = np.copy(a)
b
```

现在 `b` 是一个独立副本（称为*深拷贝*）

```{code-cell} python3
b[:] = 1
b
```

```{code-cell} python3
a
```

注意对 `b` 的更改没有影响 `a`。




## 其他功能

让我们来看看 NumPy 的其他一些有用功能。


### 通用函数

```{index} single: NumPy; Vectorized Functions
```

NumPy 提供了标准函数 `log`、`exp`、`sin` 等的版本，这些版本对数组*逐元素*作用

```{code-cell} python3
z = np.array([1, 2, 3])
np.sin(z)
```

这消除了对显式逐元素循环的需求，例如

```{code-cell} python3
n = len(z)
y = np.empty(n)
for i in range(n):
    y[i] = np.sin(z[i])
```

因为它们对数组逐元素作用，这些函数有时被称为**向量化函数**。

在 NumPy 的术语中，它们也被称为 **ufuncs**，或**通用函数**。

如上所述，常规算术运算（`+`、`*` 等）也是逐元素工作的，将这些与 ufuncs 结合起来，可以得到非常大量的快速逐元素函数。

```{code-cell} python3
z
```

```{code-cell} python3
(1 / np.sqrt(2 * np.pi)) * np.exp(- 0.5 * z**2)
```

并非所有用户自定义函数都会逐元素作用。

例如，将下面定义的函数 `f` 传递给 NumPy 数组会导致 `ValueError`

```{code-cell} python3
def f(x):
    return 1 if x > 0 else 0
```

NumPy 函数 `np.where` 提供了一个向量化的替代方案：

```{code-cell} python3
x = np.random.randn(4)
x
```

```{code-cell} python3
np.where(x > 0, 1, 0)  # 如果 x > 0 为真则插入 1，否则插入 0
```

你也可以使用 `np.vectorize` 来向量化给定的函数

```{code-cell} python3
f = np.vectorize(f)
f(x)                # 传递与前一个例子中相同的向量 x
```

但是，这种方法并不总能获得与精心设计的向量化函数相同的速度。

（稍后我们将看到 JAX 有一个强大的 `np.vectorize` 版本，通常确实可以生成高效的代码。）


### 比较

```{index} single: NumPy; Comparisons
```

通常，对数组的比较是逐元素进行的

```{code-cell} python3
z = np.array([2, 3])
y = np.array([2, 3])
z == y
```

```{code-cell} python3
y[0] = 5
z == y
```

```{code-cell} python3
z != y
```

`>`、`<`、`>=` 和 `<=` 的情况类似。

我们也可以与标量进行比较

```{code-cell} python3
z = np.linspace(0, 10, 5)
z
```

```{code-cell} python3
z > 3
```

这对于*条件提取*特别有用

```{code-cell} python3
b = z > 3
b
```

```{code-cell} python3
z[b]
```

当然，我们可以——而且经常——在一个步骤中完成这些操作

```{code-cell} python3
z[z > 3]
```

### 子包

NumPy 通过其子包提供了一些与科学编程相关的附加功能。

我们已经看到了如何使用 np.random 生成随机变量

```{code-cell} python3
z = np.random.randn(10000)  # 生成标准正态随机数
y = np.random.binomial(10, 0.5, size=1000)    # 从 Bin(10, 0.5) 中抽取 1000 个样本
y.mean()
```

另一个常用的子包是 np.linalg

```{code-cell} python3
A = np.array([[1, 2], [3, 4]])

np.linalg.det(A)           # 计算行列式
```

```{code-cell} python3
np.linalg.inv(A)           # 计算逆矩阵
```

```{index} single: SciPy
```

```{index} single: Python; SciPy
```

这些功能的大部分也可在 [SciPy](https://scipy.org/) 中找到，SciPy 是建立在 NumPy 之上的模块集合。

我们将在{doc}`不久后 <scipy>`更详细地介绍 SciPy 版本。

有关 NumPy 中可用内容的完整列表，请参阅[此文档](https://numpy.org/doc/stable/reference/routines.html)。


### 隐式多线程

[之前](need_for_speed)我们讨论了通过多线程实现并行化的概念。

NumPy 在其大部分编译代码中尝试实现多线程。

让我们看一个例子来观察这一点。

下面这段代码计算大量随机生成矩阵的特征值。

运行需要几秒钟。

```{code-cell} python3
n = 20
m = 1000
for i in range(n):
    X = np.random.randn(m, m)
    λ = np.linalg.eigvals(X)
```

现在，让我们看看这段代码运行时机器上 htop 系统监视器的输出：

```{figure} /_static/lecture_specific/parallelization/htop_parallel_npmat.png
:scale: 80
```

我们可以看到 8 个 CPU 中有 4 个在全速运行。

这是因为 NumPy 的 `eigvals` 例程巧妙地将任务分割并分发到不同的线程。


## 练习


```{exercise-start}
:label: np_ex1
```

考虑多项式表达式

```{math}
:label: np_polynom

p(x) = a_0 + a_1 x + a_2 x^2 + \cdots a_N x^N = \sum_{n=0}^N a_n x^n
```

{ref}`之前 <pyess_ex2>`，你编写了一个简单函数 `p(x, coeff)` 来求解 {eq}`np_polynom`，但没有考虑效率。

现在编写一个执行相同任务的新函数，但使用 NumPy 数组和数组操作进行计算，而不是任何形式的 Python 循环。

（这样的功能已经在 `np.poly1d` 中实现，但为了练习目的，请不要使用该类）

```{hint}
:class: dropdown
使用 `np.cumprod()`
```
```{exercise-end}
```

```{solution-start} np_ex1
:class: dropdown
```

这段代码完成了任务

```{code-cell} python3
def p(x, coef):
    X = np.ones_like(coef)
    X[1:] = x
    y = np.cumprod(X)   # y = [1, x, x**2,...]
    return coef @ y
```

让我们测试它

```{code-cell} python3
x = 2
coef = np.linspace(2, 4, 3)
print(coef)
print(p(x, coef))
# 作为对比
q = np.poly1d(np.flip(coef))
print(q(x))
```

```{solution-end}
```


```{exercise-start}
:label: np_ex2
```

设 `q` 是长度为 `n` 且 `q.sum() == 1` 的 NumPy 数组。

假设 `q` 表示一个[概率质量函数](https://en.wikipedia.org/wiki/Probability_mass_function)。

我们希望生成一个离散随机变量 $x$，使得 $\mathbb P\{x = i\} = q_i$。

换句话说，`x` 取 `range(len(q))` 中的值，且 `x = i` 的概率为 `q[i]`。

标准（逆变换）算法如下：

* 将单位区间 $[0, 1]$ 分成 $n$ 个子区间 $I_0, I_1, \ldots, I_{n-1}$，使得 $I_i$ 的长度为 $q_i$。
* 在 $[0, 1]$ 上抽取一个均匀随机变量 $U$，返回满足 $U \in I_i$ 的 $i$。

抽到 $i$ 的概率是 $I_i$ 的长度，等于 $q_i$。

我们可以如下实现该算法

```{code-cell} python3
from random import uniform

def sample(q):
    a = 0.0
    U = uniform(0, 1)
    for i in range(len(q)):
        if a < U <= a + q[i]:
            return i
        a = a + q[i]
```

如果你不明白这是如何工作的，试着通过一个简单的例子来理解流程，例如 `q = [0.25, 0.75]`。
在纸上画出区间会有帮助。

你的练习是使用 NumPy 加速它，避免显式循环

```{hint}
:class: dropdown

使用 `np.searchsorted` 和 `np.cumsum`

```

如果可以的话，将功能实现为名为 `DiscreteRV` 的类，其中

* 类的实例数据是概率向量 `q`
* 该类有一个 `draw()` 方法，根据上述算法返回一个样本

如果可以的话，编写该方法使得 `draw(k)` 从 `q` 中返回 `k` 个样本。

```{exercise-end}
```

```{solution-start} np_ex2
:class: dropdown
```

以下是我们的初始解决方案：

```{code-cell} python3
from numpy import cumsum
from numpy.random import uniform

class DiscreteRV:
    """
    根据给定概率向量 q 生成离散随机变量的抽样数组。
    """

    def __init__(self, q):
        """
        参数 q 是一个 NumPy 数组或类似数组，非负且和为 1
        """
        self.q = q
        self.Q = cumsum(q)

    def draw(self, k=1):
        """
        从 q 中返回 k 个样本。对于每次抽样，值 i 以概率 q[i] 返回。
        """
        return self.Q.searchsorted(uniform(0, 1, size=k))
```

逻辑不是很直观，但如果你慢慢阅读，就会理解。

然而，这里有一个问题。

假设在创建 `discreteRV` 实例后修改了 `q`，例如

```{code-cell} python3
q = (0.1, 0.9)
d = DiscreteRV(q)
d.q = (0.5, 0.5)
```

问题是 `Q` 不会相应地改变，而 `Q` 是 `draw` 方法中使用的数据。

要处理这个问题，一个选项是每次调用 draw 方法时都计算 `Q`。

但这相对于一次性计算 `Q` 来说效率较低。

一个更好的选项是使用描述符。

[quantecon 库](https://github.com/QuantEcon/QuantEcon.py/tree/main/quantecon)中使用描述符的解决方案，其行为符合我们的要求，可以在[这里](https://github.com/QuantEcon/QuantEcon.py/blob/main/quantecon/discrete_rv.py)找到。

```{solution-end}
```


```{exercise}
:label: np_ex3

回顾我们{ref}`之前关于 <oop_ex1>`经验累积分布函数的讨论。

你的任务是：

1. 使用 NumPy 使 `__call__` 方法更高效。
1. 添加一个在 $[a, b]$ 上绘制经验累积分布函数的方法，其中 $a$ 和 $b$ 是方法参数。
```

```{solution-start} np_ex3
:class: dropdown
```

下面给出了一个示例解决方案。

本质上，我们只是取了来自 QuantEcon 的[这段代码](https://github.com/QuantEcon/QuantEcon.py/blob/main/quantecon/ecdf.py)并添加了绘图方法

```{code-cell} python3
"""
修改 QuantEcon 的 ecdf.py 以添加绘图方法

"""

class ECDF:
    """
    给定观测向量的一维经验分布函数。

    Parameters
    ----------
    observations : array_like
        观测数组

    Attributes
    ----------
    observations : array_like
        观测数组

    """

    def __init__(self, observations):
        self.observations = np.asarray(observations)

    def __call__(self, x):
        """
        在 x 处评估经验累积分布函数

        Parameters
        ----------
        x : scalar(float)
            评估经验累积分布函数的 x 值

        Returns
        -------
        scalar(float)
            样本中小于 x 的比例

        """
        return np.mean(self.observations <= x)

    def plot(self, ax, a=None, b=None):
        """
        在区间 [a, b] 上绘制经验累积分布函数。

        Parameters
        ----------
        a : scalar(float), optional(default=None)
            绘图区间的下端点
        b : scalar(float), optional(default=None)
            绘图区间的上端点

        """

        # === 如果未指定 [a, b]，则选择合理的区间 === #
        if a is None:
            a = self.observations.min() - self.observations.std()
        if b is None:
            b = self.observations.max() + self.observations.std()

        # === 生成图形 === #
        x_vals = np.linspace(a, b, num=100)
        f = np.vectorize(self.__call__)
        ax.plot(x_vals, f(x_vals))
        plt.show()
```

以下是使用示例

```{code-cell} python3
fig, ax = plt.subplots()
X = np.random.randn(1000)
F = ECDF(X)
F.plot(ax)
```

```{solution-end}
```


```{exercise-start}
:label: np_ex4
```

回顾 NumPy 中的[广播](broadcasting)可以帮助我们在不使用 `for` 循环的情况下对不同维度的数组执行逐元素操作。

在本练习中，尝试使用 `for` 循环来复现以下广播操作的结果。

**第一部分**：尝试使用 `for` 循环复现以下简单示例，并将你的结果与下面的广播操作进行比较。

```{code-cell} python3

np.random.seed(123)
x = np.random.randn(4, 4)
y = np.random.randn(4)
A = x / y
```

以下是输出结果

```{code-cell} python3
---
tags: [hide-output]
---
print(A)
```

**第二部分**：继续复现以下广播操作的结果。同时，比较广播和你实现的 `for` 循环的速度。

对于本练习的这一部分，你可以使用 `quantecon` 库中的 `tic`/`toc` 函数来计时执行。

让我们确保已安装该库。

```{code-cell} python3
:tags: [hide-output]
!pip install quantecon
```

现在我们可以导入 quantecon 包。

```{code-cell} python3

np.random.seed(123)
x = np.random.randn(1000, 100, 100)
y = np.random.randn(100)

with qe.Timer("广播操作"):
    B = x / y
```

以下是输出结果

```{code-cell} python3
---
tags: [hide-output]
---
print(B)
```

```{exercise-end}
```


```{solution-start} np_ex4
:class: dropdown
```

**第一部分解答**

```{code-cell} python3
np.random.seed(123)
x = np.random.randn(4, 4)
y = np.random.randn(4)

C = np.empty_like(x)
n = len(x)
for i in range(n):
    for j in range(n):
        C[i, j] = x[i, j] / y[j]
```

比较结果以检查你的答案

```{code-cell} python3
---
tags: [hide-output]
---
print(C)
```

你也可以使用 `array_equal()` 来检查你的答案

```{code-cell} python3
print(np.array_equal(A, C))
```


**第二部分解答**

```{code-cell} python3

np.random.seed(123)
x = np.random.randn(1000, 100, 100)
y = np.random.randn(100)

with qe.Timer("For 循环操作"):
    D = np.empty_like(x)
    d1, d2, d3 = x.shape
    for i in range(d1):
        for j in range(d2):
            for k in range(d3):
                D[i, j, k] = x[i, j, k] / y[k]
```

注意 `for` 循环比广播操作花费的时间长得多。

比较结果以检查你的答案

```{code-cell} python3
---
tags: [hide-output]
---
print(D)
```

```{code-cell} python3
print(np.array_equal(B, D))
```

```{solution-end}
```