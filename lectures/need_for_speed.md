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
  title: 科学计算中的 Python
  headings:
    Overview: 概述
    Major Scientific Libraries: 主要科学库
    Major Scientific Libraries::Why do we need them?: 为什么需要它们？
    Major Scientific Libraries::Python's Scientific Ecosystem: Python 的科学生态系统
    Why is Pure Python Slow?: 为什么纯 Python 速度较慢？
    Why is Pure Python Slow?::Type Checking: 类型检查
    Why is Pure Python Slow?::Type Checking::Dynamic typing: 动态类型
    Why is Pure Python Slow?::Type Checking::Static types: 静态类型
    Why is Pure Python Slow?::Data Access: 数据访问
    Why is Pure Python Slow?::Data Access::Summing with Compiled Code: 使用编译代码求和
    Why is Pure Python Slow?::Data Access::Summing in Pure Python: 用纯 Python 求和
    Why is Pure Python Slow?::Summary: 总结
    Accelerating Python: 加速 Python
    Accelerating Python::Vectorization: 向量化
    Accelerating Python::Vectorization vs pure Python loops: 向量化 vs 纯 Python 循环
    Accelerating Python::JIT compilers: JIT 编译器
    Parallelization: 并行化
    Parallelization::Parallelization on CPUs: CPU 上的并行化
    Parallelization::Parallelization on CPUs::Multithreading: 多线程
    Parallelization::Parallelization on CPUs::Multiprocessing: 多进程
    Parallelization::Parallelization on CPUs::Which Should We Use?: 应该选择哪种方式？
    Parallelization::Hardware Accelerators: 硬件加速器
    Parallelization::Accessing GPU Resources: 访问 GPU 资源
---

(speed)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# 科学计算中的 Python

```{epigraph}
"我们应该忘记那些小的效率提升，大约97%的时间都是如此：过早优化是万恶之源。" -- Donald Knuth
```

## 概述

可以说，Python 是科学计算领域最流行的编程语言。

这得益于以下几点：

* 语言本身易于理解和表达，
* 丰富的高质量科学库，
* 该语言及其库均为开源，
* Python 在数据科学、机器学习和人工智能领域发挥着核心作用。

在之前的讲座中，我们使用了一些科学 Python 库，包括 NumPy 和 Matplotlib。

然而，我们的主要关注点是 Python 核心语言，而非这些库。

现在我们将目光转向科学库，并对其进行深入探讨。

在这节入门讲座中，我们将讨论以下主题：

1. 科学 Python 生态系统的主要组成部分是什么？
1. 它们是如何相互配合的？
1. 随着时间的推移，情况是如何变化的？

除了 Anaconda 中已有的内容之外，本讲座还需要以下安装包：

```{code-cell} ipython
---
tags: [hide-output]
---
!pip install quantecon
```

让我们从一些导入开始：

```{code-cell} ipython
import numpy as np
import quantecon as qe
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
import matplotlib.font_manager  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n
import random
```

## 主要科学库

让我们简要回顾一下 Python 的科学库。

### 为什么需要它们？

我们需要 Python 科学库的两个原因：

1. Python 体量小
2. Python 速度慢

**Python 体量小**

核心 Python 在设计上体量小——这有助于优化、可访问性和维护。

科学库提供了我们不想——也可能不应该——自己编写的程序：

* 数值积分、插值、线性代数、求根等。

**Python 速度慢**

我们需要科学库的另一个原因是纯 Python 相对较慢。

科学库通过三种主要策略加速执行：

1. 向量化：提供编译好的机器码以及使该代码可访问的接口
1. JIT 编译：将类 Python 语句在运行时转换为快速机器码的编译器
2. 并行化：将任务分配到多个线程/CPU/GPU/TPU 上

我们将在下文深入讨论这些思想。

### Python 的科学生态系统

在 QuantEcon，我们最常使用的科学库是：

* [NumPy](https://numpy.org/)
* [SciPy](https://scipy.org/)
* [Matplotlib](https://matplotlib.org/)
* [JAX](https://github.com/jax-ml/jax)
* [Pandas](https://pandas.pydata.org/)
* [Numba](https://numba.pydata.org/)

它们之间的关系如下：

* NumPy 通过提供基本的数组数据类型（可以理解为向量和矩阵）以及对这些数组进行操作的函数（例如矩阵乘法）奠定了基础。
* SciPy 在 NumPy 的基础上，添加了科学中常用的数值方法（插值、优化、求根等）。
* Matplotlib 用于生成图形，重点是绘制存储在 NumPy 数组中的数据。
* JAX 包含类似于 NumPy 的数组处理操作、自动微分、以并行化为核心的即时编译器，以及与 GPU 等硬件加速器的自动集成。
* Pandas 提供用于操作数据的类型和函数。
* Numba 提供一个即时编译器，与 NumPy 配合良好，有助于加速 Python 代码。

我们将在本系列讲座中广泛讨论所有这些库。

## 为什么纯 Python 速度较慢？

如上所述，用纯 Python 编写的数值计算代码相对较慢。

让我们尝试理解是什么导致了执行速度缓慢。

### 类型检查

纯 Python 操作中的一个开销来源是类型检查。

让我们尝试理解其中的问题。

#### 动态类型

```{index} single: Dynamic Typing
```

考虑以下 Python 操作

```{code-cell} python3
a, b = 10, 10
a + b
```

即使对于这个简单的操作，Python 解释器也需要做相当多的工作。

例如，在语句 `a + b` 中，解释器必须知道应该调用哪种操作。

如果 `a` 和 `b` 是字符串，那么 `a + b` 需要执行字符串拼接

```{code-cell} python3
a, b = 'foo', 'bar'
a + b
```

如果 `a` 和 `b` 是列表，那么 `a + b` 需要执行列表拼接

```{code-cell} python3
a, b = ['foo'], ['bar']
a + b
```

因此，当执行 `a + b` 时，Python 必须首先检查对象的类型，然后再调用正确的操作。

这带来了额外的开销。

如果我们在一个紧凑的循环中反复执行此表达式，开销将变得很大。

#### 静态类型

```{index} single: Static Types
```

编译型语言通过显式的静态类型来避免这些开销。

例如，考虑以下 C 代码，它计算从 1 到 10 的整数之和

```{code-block} c
:class: no-execute

#include <stdio.h>

int main(void) {
    int i;
    int sum = 0;
    for (i = 1; i <= 10; i++) {
        sum = sum + i;
    }
    printf("sum = %d\n", sum);
    return 0;
}
```

变量 `i` 和 `sum` 被显式声明为整数类型。

此外，当我们写下 `int i` 这样的语句时，我们是在向编译器承诺：在整个程序执行过程中，`i` 将*始终*是一个整数。

因此，表达式 `sum + i` 中加法的含义是完全明确的。

无需进行类型检查，也就不存在额外开销。

### 数据访问

高级语言速度较慢的另一个原因是数据访问方式。

为了说明这一点，让我们考虑对某些数据求和的问题——比如，一组整数的集合。

#### 使用编译代码求和

在 C 或 Fortran 中，整数数组存储在一块连续的内存空间中

* 例如，一个 64 位整数占用 8 个字节的内存。
* 由 $n$ 个这样的整数组成的数组占用 $8n$ 个连续字节。

此外，数据类型在编译时是已知的。

因此，每个连续的数据点都可以通过在内存空间中向前移动一个已知且固定的量来访问。

#### 用纯 Python 求和

Python 在一定程度上尝试复现这些思路。

例如，在标准 Python 实现（CPython）中，列表元素被放置在某种意义上连续的内存位置。

然而，这些列表元素更像是指向数据的指针，而不是实际的数据本身。

因此，访问数据值本身仍然存在额外开销。

这种开销是导致执行速度缓慢的主要原因之一。

### 总结

上述讨论是否意味着我们应该将所有工作都切换到 C 或 Fortran？

答案是：绝对不是！

对于任何给定的程序，真正对时间敏感的代码行数相对较少。

因此，用 Python 这样的高生产力语言编写大部分代码效率要高得多。

此外，即使对于那些*确实*对时间敏感的代码行，我们现在也可以通过使用 Python 的科学计算库来达到甚至超越由 C 或 Fortran 编译的二进制文件的性能。

在这一点上，我们强调，在过去几年中，代码加速在本质上已经与并行化画上了等号。

这项任务最好留给专门的编译器来完成！

## 加速 Python

在本节中，我们将介绍三种加速 Python 代码的相关技术。

这里我们将重点关注基本思想。

稍后我们将研究具体的库以及它们如何实现这些思想。

### {index}`向量化 <single: Vectorization>`

```{index} single: Python; Vectorization
```

避免内存流量和类型检查的一种方法是[数组编程](https://en.wikipedia.org/wiki/Array_programming)。

许多经济学家通常将数组编程称为"向量化"。

```{note}
在计算机科学中，这个术语有[稍微不同的含义](https://en.wikipedia.org/wiki/Automatic_vectorization)。
```

其核心思想是将数组处理操作批量发送到预编译的高效本地机器码。

机器码本身通常由经过仔细优化的 C 或 Fortran 编译而来。

例如，在高级语言中工作时，对大矩阵求逆的操作可以委托给高效的机器码，该机器码为此目的预先编译，并作为包的一部分提供给用户。

其核心优势在于：

1. 类型检查是*按数组*进行的，而不是按元素进行的，以及
1. 包含相同数据类型元素的数组在内存访问方面是高效的。

向量化的思想可以追溯到 MATLAB，MATLAB 大量使用向量化。


```{figure} /_static/lecture_specific/need_for_speed/matlab.png
```

NumPy 使用类似的模型，灵感来源于 MATLAB

### 向量化 vs 纯 Python 循环

让我们做一个快速的速度比较，来说明向量化如何加速代码。

以下是一些非向量化的代码，它使用原生 Python 循环来生成、平方并求和大量随机变量：

```{code-cell} python3
n = 1_000_000
```

```{code-cell} python3
with qe.Timer():
    y = 0      # 将累加并存储总和
    for i in range(n):
        x = random.uniform(0, 1)
        y += x**2
```

以下向量化代码使用 NumPy（我们将很快深入研究）来实现同样的功能：

```{code-cell} ipython
with qe.Timer():
    x = np.random.uniform(0, 1, n)
    y = np.sum(x**2)
```

如您所见，第二个代码块运行速度快得多。

它将循环分解为三个基本操作：

1. 生成 `n` 个均匀分布的随机数
1. 对它们求平方
1. 对它们求和

这些操作作为批量算子发送到经过优化的机器码。




(numba-p_c_vectorization)=
### JIT 编译器

在最好的情况下，向量化可以产生快速、简洁的代码。

然而，它也并非没有缺点。

一个问题是它可能非常消耗内存。

这是因为向量化在产生最终计算结果之前，往往会创建许多中间数组。

另一个问题是并非所有算法都可以向量化。

由于这些问题，大多数高性能计算正在从传统向量化转向使用[即时编译器](https://en.wikipedia.org/wiki/Just-in-time_compilation)。

在本系列后续讲座中，我们将学习现代 Python 库如何利用即时编译器生成快速、高效、并行化的机器码。

## 并行化

近年来，CPU 时钟速度（即单条逻辑链的运行速度）的增长已大幅放缓。

芯片设计师和计算机程序员通过寻求一条不同的路径来应对这一放缓：并行化。

这涉及到：

1. 增加每台机器中嵌入的 CPU 数量
1. 连接 GPU 和 TPU 等硬件加速器

对于程序员来说，挑战在于充分利用这些硬件，同时运行多个进程（即并行）。

下面我们讨论科学计算中的并行化，重点关注：

1. Python 中的并行化工具，以及
1. 这些工具如何应用于定量经济学问题。

### CPU 上的并行化

让我们回顾一下科学计算中常用的两种主要 CPU 并行化方式，并讨论它们的优缺点。

#### 多线程

多线程是指在单个进程中运行多个执行线程。

所有线程共享同一内存空间，因此它们可以在不复制数据的情况下对同一数组进行读写。

例如，当对大型数组的数值操作在现代笔记本电脑上运行时，工作负载可以分配到机器的多个 CPU 核心上，每个核心处理数组的一部分。

```{note}
由于一些[遗留的设计特性](https://wiki.python.org/moin/GlobalInterpreterLock)，原生 Python 难以实现多线程。
但这对于 NumPy 和 Numba 等科学库来说并不是限制。
从这些库导入的函数和经过 JIT 编译的代码在低级执行环境中运行，Python 的遗留限制在那里并不适用。
```

#### 多进程

多进程是指运行多个独立进程，每个进程都有自己独立的内存空间。

由于内存不共享，进程之间通过相互传递数据进行通信。

多进程可以在单台机器上运行，也可以分布在通过网络连接的多台机器（集群）上。

#### 应该选择哪种方式？

对于单台机器上的数值工作，通常首选多线程——它更加轻量，且共享内存模型非常便利。

当需要扩展到单台机器之外时，多进程就变得重要了。

对于我们在这些讲座中所做的绝大多数工作，多线程就足够了。

### 硬件加速器

并行化的一个更为重要的来源来自专用硬件加速器，尤其是 **GPU**（图形处理单元）。

GPU 最初是为渲染图形而设计的，这需要同时对许多像素执行相同的操作。

```{figure} /_static/lecture_specific/need_for_speed/geforce.png
:scale: 40
```

这种架构——数千个简单核心对不同数据点执行相同指令——恰好非常适合科学计算。

```{note}
**核心**是芯片中的一个独立处理单元——一种可以自主执行指令的电路。CPU 通常拥有少量强大的核心，每个核心都能处理复杂的操作序列。而 GPU 则集成了数千个更小、更简单的核心，每个核心专门执行基本的算术运算。GPU 的强大之处在于让所有这些核心同时处理同一问题的不同部分。
```

当一个计算可以表示为对大型数据数组进行独立操作时，GPU 的速度可以比 CPU 快几个数量级。

由 Google 为机器学习设计的 **TPU**（张量处理单元）遵循类似的理念，专门针对大规模并行矩阵运算进行优化。

### 访问 GPU 资源

许多工作站和笔记本电脑现在都配备了性能强劲的 GPU，对于个人研究项目来说，单块现代 GPU 通常就足够了。

现代 Python 库（如本系列讲座中广泛讨论的 JAX）可以以最少的代码改动自动检测并使用可用的 GPU。

对于规模更大的问题，包含多个 GPU（通常每台机器 4--8 个 GPU）的服务器越来越普遍。

```{figure} /_static/lecture_specific/need_for_speed/dgx.png
:scale: 40
```

借助适当的软件，计算可以分布在多个 GPU 上，无论是在单台服务器内还是跨集群。

我们将在后续讲座中更详细地探讨 GPU 计算，并将其应用于一系列经济学应用。
