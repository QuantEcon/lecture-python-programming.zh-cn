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
  title: 关于本系列讲座
  headings:
    Overview: 概述
    Overview::Can't I Just Use LLMs?: 我就不能直接使用大语言模型吗？
    Overview::Isn't MATLAB Better?: MATLAB 不是更好吗？
    Introducing Python: Python 简介
    Introducing Python::Common Uses: 常见用途
    Introducing Python::Relative Popularity: 相对流行度
    Introducing Python::Features: 特性
    Introducing Python::Syntax and Design: 语法与设计
    Introducing Python::The AI Connection: 与人工智能的关联
    Scientific Programming with Python: 使用 Python 进行科学编程
    Scientific Programming with Python::NumPy: NumPy
    Scientific Programming with Python::NumPy Alternatives: NumPy 的替代方案
    Scientific Programming with Python::SciPy: SciPy
    Scientific Programming with Python::Graphics: 图形可视化
    Scientific Programming with Python::Networks and Graphs: 网络与图
    Scientific Programming with Python::Other Scientific Libraries: 其他科学库
---

(about_py)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

```{index} single: python
```

# 关于本系列讲座

```{epigraph}
"Python 已经足够强大，我们不再需要转向 R 了。抱歉，R 的用户们。我曾经也是你们中的一员，但我们不再需要转向 R 了。" -- Chris Wiggins
```

## 概述

本系列讲座将教您使用 Python 进行科学计算，重点关注经济学和金融领域。

本系列面向 Python 初学者，但有经验的用户也能在后续讲座中找到有用的内容。

在本讲座中，我们将

* 介绍 Python，
* 展示其部分功能，
* 解释为什么 Python 是我们最喜欢的科学计算语言，以及
* 为您指引下一步的学习方向。

您**不需要**理解本讲座中看到的所有内容——我们将在后续讲座系列中逐步深入讲解细节。


### 我就不能直接使用大语言模型吗？

不行！

当然，在人工智能时代，人们很容易认为我们不需要再学习如何编程了。

是的，我们有时也喜欢偷懒。

此外，我们也承认人工智能是程序员出色的生产力工具。

但人工智能无法可靠地解决它从未见过的新问题。

您需要成为架构师和监督者——而要完成这些任务，您需要能够阅读、编写和理解计算机代码。

话虽如此，一个好的大语言模型是学习本系列讲座的有用伴侣——试着把本系列中的一些代码复制粘贴过去，并请求它进行解释。


### MATLAB 不是更好吗？

不，不，绝对不是。

Nirvana 很伟大（而 Soundgarden [更胜一筹](https://www.youtube.com/watch?v=3mbBbFH9fAg&list=RD3mbBbFH9fAg)），但是是时候告别 90 年代了。

对于大多数现代问题，Python 的科学库现在已经远远超过了 MATLAB 的能力。

这在深度学习和强化学习等快速发展的领域尤为明显。

此外，所有主流大语言模型编写 Python 代码的能力都远超编写 MATLAB 代码的能力。

我们将在本系列讲座中以及后续关于 [JAX](https://jax.quantecon.org/intro.html) 的系列中讨论 Python 库的相对优势。



## Python 简介

[Python](https://www.python.org) 是一种通用编程语言，由 [Guido van Rossum](https://en.wikipedia.org/wiki/Guido_van_Rossum) 于 1989 年构想创建。

Python 是免费且[开源](https://en.wikipedia.org/wiki/Open_source)的，其开发由 [Python Software Foundation](https://www.python.org/psf-landing/) 协调推进。

这一点非常重要，因为它

* 为我们节省了费用，
* 意味着 Python 由用户社区而非营利性企业控制，以及
* 鼓励可重复性研究和[开放科学](https://en.wikipedia.org/wiki/Open_science)。


### 常见用途

{index}`Python <single: Python; common uses>` 是一种通用语言，几乎在所有应用领域都有使用，包括

* 人工智能与计算机科学
* 其他科学计算
* 通信
* 网络开发
* CGI 与图形用户界面
* 游戏开发
* 资源规划
* 多媒体
* 等等

它被众多大型科技公司广泛使用和支持，包括

* [Google](https://www.google.com/)
* [OpenAI](https://openai.com/)
* [Netflix](https://www.netflix.com/)
* [Meta](https://opensource.fb.com/)
* [Amazon](https://www.amazon.com/)
* [Reddit](https://www.reddit.com/)
* 等等


### 相对流行度

Python 是[最流行的编程语言](https://www.tiobe.com/tiobe-index/)之一——如果不是最流行的话。

Python 库，如 [pandas](https://pandas.pydata.org/) 和 [Polars](https://pola.rs/)，正在取代 Excel 和 VBA 等熟悉的工具，成为金融和银行领域的必备技能。

此外，Python 在科学界——尤其是与人工智能相关的领域——也极为流行。

例如，以下来自 Stack Overflow Trends 的图表展示了单个 Python 深度学习库（[PyTorch](https://pytorch.org/)）在过去几年中的流行度增长情况。


```{figure} /_static/lecture_specific/about_py/pytorch_vs_matlab.png
```
PyTorch 只是 Python 众多深度学习和人工智能库之一。



### 特性

Python 是一种[高级语言](https://en.wikipedia.org/wiki/High-level_programming_language)，这意味着它相对容易阅读、编写和调试。

它有一个相对较小的核心语言，易于学习。

该核心由许多库支持，可根据需要进行学习。

Python 灵活而务实，支持多种编程风格（过程式、面向对象、函数式等）。


### 语法与设计

```{index} single: Python; syntax and design
```

Python 受欢迎的原因之一是其简洁优雅的设计。

为了感受这一点，让我们来看一个例子。

以下代码是用 [Java](https://en.wikipedia.org/wiki/Java_(programming_language)) 而非 Python 编写的。

您**不需要**阅读和理解这段代码！


```{code-block} java

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class CSVReader {
    public static void main(String[] args) {
        String filePath = "data.csv"; 
        String line;
        String splitBy = ",";
        int columnIndex = 1; 
        double sum = 0;
        int count = 0;

        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            while ((line = br.readLine()) != null) {
                String[] values = line.split(splitBy);
                if (values.length > columnIndex) {
                    try {
                        double value = Double.parseDouble(
                            values[columnIndex]
                        );
                        sum += value;
                        count++;
                    } catch (NumberFormatException e) {
                        System.out.println(
                            "Skipping non-numeric value: " + 
                            values[columnIndex]
                        );
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        if (count > 0) {
            double average = sum / count;
            System.out.println(
                "Average of the second column: " + average
            );
        } else {
            System.out.println(
                "No valid numeric data found in the second column."
            );
        }
    }
}

```

这段 Java 代码打开一个名为 `data.csv` 的假想文件，并计算第二列值的平均数。

以下是完成相同功能的 Python 代码。

即使您尚不了解 Python，也能看出这段代码要简洁易读得多。

```{code-cell} python3
:tags: [skip-execution]

import csv

total, count = 0, 0
with open('data.csv', mode='r') as file:
    reader = csv.reader(file)
    for row in reader:
        try:
            total += float(row[1])
            count += 1
        except (ValueError, IndexError):
            pass
print(f"Average: {total / count if count else 'No valid data'}")

```



### 与人工智能的关联

人工智能正在接管许多目前由人类执行的任务，就像过去几个世纪其他形式的机械所做的那样。

此外，Python 在推动人工智能和机器学习的发展中扮演着举足轻重的角色。

这意味着科技公司正在大力投资开发极其强大的 Python 库。

即使您不打算从事人工智能和机器学习方向，您也可以从学习使用这些库中受益，将其应用于经济学、金融学和其他科学领域的项目中。

本系列讲座将解释如何做到这一点。


## 使用 Python 进行科学编程

```{index} single: scientific programming
```

我们已经讨论了 Python 对于人工智能、机器学习和数据科学的重要性。

Python 也是以下领域的主导力量之一：

* 天文学
* 化学
* 计算生物学
* 气象学
* 自然语言处理
* 等等

Python 在经济学、金融学以及运筹学等相邻领域的应用也在不断增长——这些领域此前主要由 MATLAB／Excel／STATA／C／Fortran 主导。

本节将简要展示一些 Python 用于通用科学编程的示例。


### NumPy

```{index} single: scientific programming; numeric
```

科学计算中最重要的部分之一是处理数据。

数据通常存储在矩阵、向量和数组中。

我们可以用纯 Python 创建一个简单的数字数组，如下所示：

```{code-cell} python3
a = [-3.14, 0, 3.14]                    # 一个 Python 列表
a
```

这个数组非常小，因此使用纯 Python 处理没有问题。

但当我们在实际程序中需要处理更大的数组时，我们需要更高的效率和更多的工具。

为此，我们需要使用处理数组的库。

对于 Python 而言，最重要的矩阵和数组处理库是 [NumPy](https://numpy.org/) 库。

例如，让我们构建一个包含 100 个元素的 NumPy 数组：

```{code-cell} python3
import numpy as np                     # 加载库
import matplotlib as mpl  # i18n
import matplotlib.font_manager  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n

a = np.linspace(-np.pi, np.pi, 100)    # 创建从 -π 到 π 的均匀网格
a
```

现在让我们通过对数组应用函数来变换它。

```{code-cell} python3
b = np.cos(a)                          # 对 a 的每个元素应用余弦函数
c = np.sin(a)                          # 对 a 的每个元素应用正弦函数
```

现在我们可以轻松计算 `b` 和 `c` 的内积。

```{code-cell} python3
b @ c
```

我们还可以完成许多其他任务，例如

* 计算数组的均值和方差
* 构建矩阵并求解线性方程组
* 生成用于模拟的随机数组，等等

我们将在后续讲座中深入介绍 NumPy 的详细内容。


### NumPy 的替代方案

虽然 NumPy 仍然是 Python 中数组处理的王者，但现在出现了一些重要的竞争者。

[JAX](https://github.com/jax-ml/jax)、[Pytorch](https://pytorch.org/) 和 [CuPy](https://cupy.dev/) 等库也拥有内置的数组类型和数组操作，速度非常快且效率极高。

事实上，这些库在利用并行化和快速硬件方面更具优势，我们将在本系列后续内容中加以说明。

但是，您仍应首先学习 NumPy，因为

* NumPy 更简单，能提供坚实的基础，以及
* JAX 等库直接扩展了 NumPy 的功能，因此在您已经掌握 NumPy 的情况下更容易学习。

本系列讲座将为您提供关于 NumPy 的深入背景知识。

### SciPy

[SciPy](https://scipy.org/) 库建立在 NumPy 之上，提供了额外的功能。

(tuple_unpacking_example)=
例如，让我们计算 $\int_{-2}^2 \phi(z) dz$，其中 $\phi$ 是标准正态密度函数。

```{code-cell} python3
from scipy.stats import norm
from scipy.integrate import quad

ϕ = norm()
value, error = quad(ϕ.pdf, -2, 2)  # 使用高斯求积法进行积分
value
```

SciPy 包含许多标准例程，用于

* [线性代数](https://docs.scipy.org/doc/scipy/reference/linalg.html)
* [积分](https://docs.scipy.org/doc/scipy/reference/integrate.html)
* [插值](https://docs.scipy.org/doc/scipy/reference/interpolate.html)
* [优化](https://docs.scipy.org/doc/scipy/reference/optimize.html)
* [分布与统计技术](https://docs.scipy.org/doc/scipy/reference/stats.html)
* [信号处理](https://docs.scipy.org/doc/scipy/reference/signal.html)

请在[此处](https://docs.scipy.org/doc/scipy/reference/index.html)查看全部功能。

稍后我们将更详细地讨论 SciPy。


### 图形可视化

```{index} single: Matplotlib
```

Python 的一大优势是数据可视化。

用于创建图形和图表的最流行、最全面的 Python 库是 [Matplotlib](https://matplotlib.org/)，其功能包括

* 折线图、直方图、等高线图、3D 图形、条形图等
* 多种格式输出（PDF、PNG、EPS 等）
* LaTeX 集成

嵌入 LaTeX 注释的二维绘图示例

```{figure} /_static/lecture_specific/about_py/qs.png
:scale: 75
```

等高线图示例

```{figure} /_static/lecture_specific/about_py/bn_density1.png
:scale: 70
```

三维图形示例

```{figure} /_static/lecture_specific/about_py/career_vf.png
```

更多示例可在 [Matplotlib 缩略图库](https://matplotlib.org/stable/gallery/index.html)中找到。

其他图形库包括

* [Plotly](https://plotly.com/python/)
* [seaborn](https://seaborn.pydata.org/) — matplotlib 的高级接口
* [Altair](https://altair-viz.github.io/)
* [Bokeh](https://docs.bokeh.org/en/latest/)

您可以访问 [Python Graph Gallery](https://python-graph-gallery.com/) 查看使用各种库绘制的更多示例图表。


### 网络与图

[网络](https://networks.quantecon.org/)研究正在成为经济学、金融学和其他领域科学工作的重要组成部分。

例如，我们有兴趣研究

* 生产网络
* 银行和金融机构网络
* 友谊与社交网络
* 等等

Python 拥有许多用于研究网络和图的库。

```{index} single: NetworkX
```

一个广为人知的例子是 [NetworkX](https://networkx.org/)。

其功能包括（还有很多其他功能）：

* 用于分析网络的标准图算法
* 绘图例程

以下是一些示例代码，用于生成并绘制一个随机图，节点颜色由从中心节点出发的最短路径长度决定。

```{code-cell} ipython
import networkx as nx
import matplotlib.pyplot as plt
np.random.seed(1234)

# 生成随机图
p = dict((i, (np.random.uniform(0, 1), np.random.uniform(0, 1)))
         for i in range(200))
g = nx.random_geometric_graph(200, 0.12, pos=p)
pos = nx.get_node_attributes(g, 'pos')

# 找到最靠近中心点 (0.5, 0.5) 的节点
dists = [(x - 0.5)**2 + (y - 0.5)**2 for x, y in list(pos.values())]
ncenter = np.argmin(dists)

# 绘制图形，按从中心节点出发的路径长度着色
p = nx.single_source_shortest_path_length(g, ncenter)
plt.figure()
nx.draw_networkx_edges(g, pos, alpha=0.4)
nx.draw_networkx_nodes(g,
                       pos,
                       nodelist=list(p.keys()),
                       node_size=120, alpha=0.5,
                       node_color=list(p.values()),
                       cmap=plt.cm.jet_r)
plt.show()
```


### 其他科学库

如上所述，Python 有数以千计的科学库。

有些库很小，只完成非常特定的任务。

有些库则在代码行数以及来自程序员和科技公司的投入方面都非常庞大。

以下是一些未在上文提及的 Python 重要科学库的简短列表。

* [SymPy](https://www.sympy.org/) — 用于符号代数，包括极限、导数和积分
* [statsmodels](https://www.statsmodels.org/) — 用于统计例程
* [scikit-learn](https://scikit-learn.org/) — 用于机器学习
* [Keras](https://keras.io/) — 用于机器学习
* [Pyro](https://pyro.ai/) 和 [PyStan](https://pystan.readthedocs.io/en/latest/) — 用于贝叶斯数据分析
* [GeoPandas](https://geopandas.org/en/stable/) — 用于空间数据分析
* [Dask](https://docs.dask.org/en/stable/) — 用于并行化
* [Numba](https://numba.pydata.org/) — 使 Python 以原生机器码的速度运行
* [CVXPY](https://www.cvxpy.org/) — 用于凸优化
* [scikit-image](https://scikit-image.org/) 和 [OpenCV](https://opencv.org/) — 用于处理和分析图像数据
* [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) — 用于从 HTML 和 XML 文件中提取数据


在本系列讲座中，我们将学习如何使用这些库中的许多库来完成经济学和金融学中的科学计算任务。