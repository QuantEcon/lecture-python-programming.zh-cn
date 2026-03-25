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
    Pure Python is slow: 纯 Python 速度慢
    Pure Python is slow::High vs low level code: 高级语言与低级语言
    Pure Python is slow::Where are the bottlenecks?: 瓶颈在哪里？
    Pure Python is slow::Where are the bottlenecks?::Dynamic typing: 动态类型
    Pure Python is slow::Where are the bottlenecks?::Static types: 静态类型
    Pure Python is slow::Data Access: 数据访问
    Pure Python is slow::Data Access::Summing with Compiled Code: 使用编译代码求和
    Pure Python is slow::Data Access::Summing in Pure Python: 在纯 Python 中求和
    Pure Python is slow::Summary: 总结
    Accelerating Python: 加速 Python
    'Accelerating Python::{index}`Vectorization <single: Vectorization>`': '{index}`向量化 <single: Vectorization>`'
    Accelerating Python::Vectorization vs for pure Python loops: 向量化 vs 纯 Python 循环
    Accelerating Python::JIT compilers: JIT 编译器
    Parallelization: 并行化
    Parallelization::Parallelization on CPUs: CPU 上的并行化
    Parallelization::Parallelization on CPUs::Multiprocessing: 多进程
    Parallelization::Parallelization on CPUs::Multithreading: 多线程
    Parallelization::Parallelization on CPUs::Advantages and Disadvantages: 优缺点
    Parallelization::Hardware Accelerators: 硬件加速器
    Parallelization::Hardware Accelerators::GPUs and TPUs: GPU 和 TPU
    Parallelization::Hardware Accelerators::Why TPUs/GPUs Matter: 为何 TPU/GPU 至关重要
    Parallelization::Single GPUs vs GPU Servers: 单 GPU 与 GPU 服务器
    Parallelization::Single GPUs vs GPU Servers::Single GPU Systems: 单 GPU 系统
    Parallelization::Single GPUs vs GPU Servers::Multi-GPU Servers: 多 GPU 服务器
    Parallelization::Summary: 总结
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
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n
import random
```


## 主要科学库

让我们简要回顾一下 Python 的科学库。


### 为什么需要它们？

我们使用科学库的一个原因是它们实现了我们想要使用的程序。

* 数值积分、插值、线性代数、求根等。

例如，使用现有的求根程序通常比从头编写一个新程序更好。

（对于标准算法，如果社区能够统一使用一套由专家编写、并经用户调优以尽可能快速和健壮的共同实现，效率将达到最高！）

但这并不是我们使用 Python 科学库的唯一原因。

另一个原因是纯 Python 速度不够快。

因此，我们需要那些专门用于加速 Python 代码执行的库。

它们通过两种策略来实现这一目标：

1. 使用编译器将类 Python 语句转换为针对单一逻辑线程的快速机器码，以及
2. 在多个"工作单元"（例如 CPU、GPU 内部的各个线程）之间并行化任务。

我们将在本讲座及本系列剩余讲座中广泛讨论这些思想。


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


## 纯 Python 速度慢

如上所述，科学库的一大吸引力在于更快的执行速度。

我们将讨论科学库如何帮助我们加速代码。

对于这个主题，如果我们理解是什么导致了执行速度慢，将会很有帮助。


### 高级语言与低级语言

像 Python 这样的高级语言是为人类优化的。

这意味着程序员可以将许多细节留给运行时环境处理，例如：

* 指定变量类型
* 内存分配与释放
* 等等。

此外，纯 Python 由一个[解释器](https://en.wikipedia.org/wiki/Interpreter_(computing))运行，该解释器逐条语句地执行代码。

这使得 Python 灵活、交互性强、易于编写、易于阅读，并且相对容易调试。

另一方面，Python 的标准实现（称为 CPython）无法与 C 或 Fortran 等编译语言的速度相媲美。


### 瓶颈在哪里？

为什么会这样呢？


#### 动态类型

```{index} single: Dynamic Typing
```

考虑这个 Python 操作：

```{code-cell} python3
a, b = 10, 10
a + b
```

即使对于这个简单的操作，Python 解释器也有相当多的工作要做。

例如，在语句 `a + b` 中，解释器必须知道调用哪个操作。

如果 `a` 和 `b` 是字符串，那么 `a + b` 需要字符串连接：

```{code-cell} python3
a, b = 'foo', 'bar'
a + b
```

如果 `a` 和 `b` 是列表，那么 `a + b` 需要列表连接：

```{code-cell} python3
a, b = ['foo'], ['bar']
a + b
```

（我们说运算符 `+` 是*重载的*——它的动作取决于它所作用的对象的类型。）

因此，在执行 `a + b` 时，Python 必须首先检查对象的类型，然后调用正确的操作。

这涉及到不可忽视的开销。

如果我们在一个紧密的循环中反复执行此表达式，这种不可忽视的开销就会变成巨大的开销。


#### 静态类型

```{index} single: Static Types
```

编译语言通过显式的静态类型来避免这些开销。

例如，考虑以下 C 代码，它对从 1 到 10 的整数求和：

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

变量 `i` 和 `sum` 被明确声明为整数。

此外，当我们写出 `int i` 这样的语句时，我们是在向编译器承诺，在程序执行的整个过程中，`i` 将*始终*是一个整数。

因此，表达式 `sum + i` 中加法的含义是完全明确的。

无需类型检查，因此没有额外开销。


### 数据访问

高级语言速度慢的另一个原因是数据访问。

为了说明这一点，让我们考虑对一些数据求和的问题——比如，一组整数。

#### 使用编译代码求和

在 C 或 Fortran 中，这些整数通常存储在数组中，数组是一种用于存储同类数据的简单数据结构。

这样的数组存储在单个连续的内存块中：

* 在现代计算机中，内存地址分配给每个字节（1字节 = 8位）。
* 例如，一个 64 位整数存储在 8 字节的内存中。
* $n$ 个这样的整数组成的数组占据 $8n$ 个*连续*的内存槽。

此外，编译器通过程序员的声明得知数据类型。

* 在本例中为 64 位整数。

因此，每个连续的数据点都可以通过在内存空间中向前移动一个已知且固定的量来访问。

* 在本例中为 8 字节。

#### 在纯 Python 中求和

Python 在一定程度上试图复制这些思想。

例如，在标准 Python 实现（CPython）中，列表元素被放置在某种意义上连续的内存位置。

然而，这些列表元素更像是指向数据的指针，而非实际数据。

因此，访问数据值本身仍然存在开销。

这对速度是一个相当大的拖累。

事实上，内存流量通常是导致执行缓慢的主要因素。



### 总结

上面的讨论是否意味着我们应该对所有事情都切换到 C 或 Fortran？

答案是：绝对不是！

对于任何给定的程序，相对而言只有少数几行代码对时间要求严苛。

因此，用 Python 这样的高生产力语言编写大部分代码要高效得多。

此外，即使对于那些*确实*对时间要求严苛的代码行，我们现在也可以通过使用 Python 的科学库，达到甚至超过从 C 或 Fortran 编译的二进制文件的速度。

在这一点上，我们强调，在过去几年中，加速代码基本上已经与并行化同义。

这个任务最好留给专门的编译器！

某些 Python 库在并行化科学代码方面具有出色的能力——我们将在后续内容中进一步讨论。




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

硬件制造商增加了每台机器中嵌入的核心数量（物理 CPU）。

对于程序员来说，挑战在于通过同时运行多个进程（即并行）来充分利用这些多个 CPU。

这在科学编程中尤为重要，因为科学编程需要处理：

* 大量数据，以及
* CPU 密集型的模拟和其他计算。

下面我们讨论科学计算中的并行化，重点关注：

1. Python 中最佳的并行化工具，以及
1. 这些工具如何应用于定量经济学问题。


### CPU 上的并行化

让我们回顾一下科学计算中常用的两种主要 CPU 并行化方式，并讨论它们的优缺点。


#### 多进程

多进程是指使用多个处理器并发执行多个进程。

在这个语境中，**进程**是一系列指令（即一个程序）。

多进程可以在一台拥有多个 CPU 的机器上进行，也可以在通过网络连接的多台机器上进行。

在后一种情况下，这些机器的集合通常称为**集群**。

在多进程中，每个进程都有自己的内存空间，尽管物理内存芯片可能是共享的。

#### 多线程

多线程与多进程类似，不同之处在于，在执行期间，所有线程共享同一内存空间。

由于一些[遗留的设计特性](https://wiki.python.org/moin/GlobalInterpreterLock)，原生 Python 难以实现多线程。

但这对于 NumPy 和 Numba 等科学库来说并不是限制。

从这些库导入的函数和经过 JIT 编译的代码在低级执行环境中运行，Python 的遗留限制在那里并不适用。

#### 优缺点

多线程更加轻量，因为大多数系统资源和内存资源由线程共享。

此外，多个线程访问共享内存池这一事实对于数值编程来说极为方便。

另一方面，多进程更灵活，可以分布在集群中。

对于我们在这些讲座中所做的绝大多数工作，多线程就足够了。


### 硬件加速器

虽然拥有多核的 CPU 已成为并行计算的标准，但随着专用硬件加速器的兴起，发生了更为深刻的变化。

这些加速器专门为科学计算、机器学习和数据科学中出现的高度并行计算而设计。

#### GPU 和 TPU

两种最重要的硬件加速器类型是：

* **GPU**（图形处理单元）和
* **TPU**（张量处理单元）。

GPU 最初是为渲染图形而设计的，这需要同时对许多像素执行相同的操作。

```{figure} /_static/lecture_specific/need_for_speed/geforce.png
:scale: 40
```

科学家和工程师意识到，这种架构——许多简单的处理器并行工作——非常适合科学计算任务。

TPU 是较新的发展，由 Google 专门为机器学习工作负载设计。

与 GPU 一样，TPU 擅长并行执行大量矩阵运算。


#### 为何 TPU/GPU 至关重要

使用硬件加速器带来的性能提升可能是惊人的。

例如，一块现代 GPU 可以包含数千个小型处理核心，而 CPU 中通常只有 8 到 64 个核心。

当一个问题可以表示为对数据数组进行许多独立操作时，GPU 的速度可以比 CPU 快几个数量级。

这对科学计算尤为重要，因为许多算法天然地映射到 GPU 的并行架构上。


### 单 GPU 与 GPU 服务器

访问 GPU 资源有两种常见方式：

#### 单 GPU 系统

许多工作站和笔记本电脑现在配备了性能强劲的 GPU，或者可以安装 GPU。

单块现代 GPU 可以显著加速许多科学计算任务。

对于个人研究人员和小型项目，单块 GPU 通常就足够了。

现代 Python 库（如本系列讲座中广泛讨论的 JAX）可以以最少的代码改动自动检测并使用可用的 GPU。


#### 多 GPU 服务器

对于规模更大的问题，包含多个 GPU（通常每台服务器 4-8 个 GPU）的服务器越来越普遍。

```{figure} /_static/lecture_specific/need_for_speed/dgx.png
:scale: 40
```


借助适当的软件，计算可以分布在多个 GPU 上，无论是在单台服务器内还是跨多台服务器。

这使研究人员能够处理在单个 GPU 或 CPU 上不可行的问题。


### 总结

GPU 计算正变得越来越容易获取，尤其是在 Python 中。

一些 Python 科学库（如 JAX）现在支持 GPU 加速，对现有代码的改动极少。

我们将在后续讲座中更详细地探讨 GPU 计算，并将其应用于一系列经济学应用。