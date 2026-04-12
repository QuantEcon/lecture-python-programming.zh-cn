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
  title: Matplotlib
  headings:
    Overview: 概述
    Overview::Matplotlib's Split Personality: Matplotlib 的双重性格
    The APIs: API 接口
    The APIs::The MATLAB-style API: MATLAB 风格 API
    The APIs::The Object-Oriented API: 面向对象 API
    The APIs::Tweaks: 调整
    More Features: 更多特性
    More Features::Multiple Plots on One Axis: 在同一坐标轴上绘制多个图形
    More Features::Multiple Subplots: 多个子图
    More Features::3D Plots: 三维图形
    More Features::A Customizing Function: 自定义函数
    More Features::Style Sheets: 样式表
    Further Reading: 延伸阅读
    Exercises: 练习
---

(matplotlib)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`Matplotlib <single: Matplotlib>`

```{index} single: Python; Matplotlib
```

## 概述

在这些讲座中，我们已经使用 [Matplotlib](https://matplotlib.org/) 生成了相当多的图形。

Matplotlib 是一个出色的图形库，专为科学计算设计，具有以下特点：

* 高质量的二维和三维图形
* 支持所有常见格式（PDF、PNG 等）的输出
* LaTeX 集成
* 对演示效果各方面的精细控制
* 动画等功能

### Matplotlib 的双重性格

Matplotlib 的独特之处在于它提供了两种不同的绘图接口。

其一是简单的 MATLAB 风格 API（应用程序编程接口），这是为了帮助从 MATLAB 转来的用户能够快速上手而编写的。

另一种是更具"Python 风格"的面向对象 API。

出于下文所述的原因，我们建议您使用第二种 API。

但首先，让我们来讨论两者之间的区别。

## API 接口

```{index} single: Matplotlib; Simple API
```

### MATLAB 风格 API

以下是您可能在入门教程中找到的简单示例

```{code-cell} ipython
import matplotlib.pyplot as plt
import numpy as np
import matplotlib as mpl  # i18n
import matplotlib.font_manager  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n

x = np.linspace(0, 10, 200)
y = np.sin(x)

plt.plot(x, y, 'b-', linewidth=2)
plt.show()
```

这种方式简单方便，但也有一定局限性，且不够 Python 化。

例如，在函数调用中，许多对象被创建并传递，而程序员对此并不知情。

Python 程序员倾向于更明确的编程风格（在代码块中运行 `import this` 并查看第二行）。

这引导我们使用另一种替代方案——面向对象的 Matplotlib API。

### 面向对象 API

以下是使用面向对象 API 对应前述图形的代码

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'b-', linewidth=2)
plt.show()
```

这里，调用 `fig, ax = plt.subplots()` 返回一个元组，其中

* `fig` 是一个 `Figure` 实例——就像一块空白画布。
* `ax` 是一个 `AxesSubplot` 实例——可以将其看作绘图的框架。

`plot()` 函数实际上是 `ax` 的一个方法。

虽然需要多写一些代码，但对象的更明确使用让我们拥有更好的控制能力。

随着我们的深入，这一点将变得更加清晰。

### 调整

这里我们将线条改为红色并添加了图例

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'r-', linewidth=2, label='正弦函数', alpha=0.6)
ax.legend()
plt.show()
```

我们还使用了 `alpha` 使线条略微透明——这使其看起来更平滑。

图例的位置可以通过将 `ax.legend()` 替换为 `ax.legend(loc='upper center')` 来改变。

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'r-', linewidth=2, label='正弦函数', alpha=0.6)
ax.legend(loc='upper center')
plt.show()
```

如果一切配置正确，添加 LaTeX 是轻而易举的

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'r-', linewidth=2, label=r'$y=\sin(x)$', alpha=0.6)
ax.legend(loc='upper center')
plt.show()
```

控制刻度、添加标题等操作也同样简单直接

```{code-cell} python3
fig, ax = plt.subplots()
ax.plot(x, y, 'r-', linewidth=2, label=r'$y=\sin(x)$', alpha=0.6)
ax.legend(loc='upper center')
ax.set_yticks([-1, 0, 1])
ax.set_title('测试图')
plt.show()
```

## 更多特性

Matplotlib 拥有大量的函数和特性，您可以随着需要逐渐发现它们。

我们仅介绍其中几个。

### 在同一坐标轴上绘制多个图形

```{index} single: Matplotlib; Multiple Plots on One Axis
```

在同一坐标轴上生成多个图形是很直接的。

以下示例随机生成三个正态密度曲线，并添加各自均值的标签

```{code-cell} python3
from scipy.stats import norm
from random import uniform

fig, ax = plt.subplots()
x = np.linspace(-4, 4, 150)
for i in range(3):
    m, s = uniform(-1, 1), uniform(1, 2)
    y = norm.pdf(x, loc=m, scale=s)
    current_label = rf'$\mu = {m:.2}$'
    ax.plot(x, y, linewidth=2, alpha=0.6, label=current_label)
ax.legend()
plt.show()
```

### 多个子图

```{index} single: Matplotlib; Subplots
```

有时我们希望在一个图形中包含多个子图。

以下示例生成 6 个直方图

```{code-cell} python3
num_rows, num_cols = 3, 2
fig, axes = plt.subplots(num_rows, num_cols, figsize=(10, 12))
for i in range(num_rows):
    for j in range(num_cols):
        m, s = uniform(-1, 1), uniform(1, 2)
        x = norm.rvs(loc=m, scale=s, size=100)
        axes[i, j].hist(x, alpha=0.6, bins=20)
        t = rf'$\mu = {m:.2}, \quad \sigma = {s:.2}$'
        axes[i, j].set(title=t, xticks=[-4, 0, 4], yticks=[])
plt.show()
```

### 三维图形

```{index} single: Matplotlib; 3D Plots
```

Matplotlib 在三维图形方面表现出色——以下是一个示例

```{code-cell} python3
from mpl_toolkits.mplot3d.axes3d import Axes3D
from matplotlib import cm


def f(x, y):
    return np.cos(x**2 + y**2) / (1 + x**2 + y**2)

xgrid = np.linspace(-3, 3, 50)
ygrid = xgrid
x, y = np.meshgrid(xgrid, ygrid)

fig = plt.figure(figsize=(10, 6))
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(x,
                y,
                f(x, y),
                rstride=2, cstride=2,
                cmap=cm.jet,
                alpha=0.7,
                linewidth=0.25)
ax.set_zlim(-0.5, 1.0)
plt.show()
```

### 自定义函数

也许您会发现一组您经常使用的自定义设置。

假设我们通常希望坐标轴过原点，并显示网格线。

以下是 [Matthew Doty](https://github.com/xcthulhu) 提供的一个很好的示例，展示了如何使用面向对象 API 构建一个实现这些更改的自定义 `subplots` 函数。

仔细阅读代码，看看您是否能理解其中的逻辑

```{code-cell} python3
def subplots():
    "自定义坐标轴过原点的子图"
    fig, ax = plt.subplots()

    # 将坐标轴设置为过原点
    for spine in ['left', 'bottom']:
        ax.spines[spine].set_position('zero')
    for spine in ['right', 'top']:
        ax.spines[spine].set_color('none')

    ax.grid()
    return fig, ax


fig, ax = subplots()  # 调用本地版本，而非 plt.subplots()
x = np.linspace(-2, 10, 200)
y = np.sin(x)
ax.plot(x, y, 'r-', linewidth=2, label='正弦函数', alpha=0.6)
ax.legend(loc='lower right')
plt.show()
```

自定义的 `subplots` 函数

1. 在内部调用标准的 `plt.subplots` 函数以生成 `fig, ax` 对，
1. 对 `ax` 进行所需的自定义，
1. 将 `fig, ax` 对传回调用代码。

### 样式表

Matplotlib 的另一个有用特性是[样式表](https://matplotlib.org/stable/gallery/style_sheets/style_sheets_reference.html)。

我们可以使用样式表来创建具有统一风格的图形。

我们可以通过打印属性 `plt.style.available` 来查找可用样式列表

```{code-cell} python3
print(plt.style.available)
```

现在我们可以使用 `plt.style.use()` 方法来设置样式表。

让我们编写一个函数，该函数接受样式表的名称并使用该样式绘制不同的图形

```{code-cell} python3

def draw_graphs(style='default'):

    # 设置样式表
    plt.style.use(style)

    fig, axes = plt.subplots(nrows=1, ncols=4, figsize=(10, 3))
    x = np.linspace(-13, 13, 150)

    # 设置随机种子以复现随机抽取结果
    np.random.seed(9)

    for i in range(3):

        # 从均匀分布中抽取均值和标准差
        m, s = np.random.uniform(-8, 8), np.random.uniform(2, 2.5)

        # 生成正态密度图
        y = norm.pdf(x, loc=m, scale=s)
        axes[0].plot(x, y, linewidth=3, alpha=0.7)

        # 从正态分布中创建具有随机 X 和 Y 值的散点图
        rnormX = norm.rvs(loc=m, scale=s, size=150)
        rnormY = norm.rvs(loc=m, scale=s, size=150)
        axes[1].plot(rnormX, rnormY, ls='none', marker='o', alpha=0.7)

        # 使用随机 X 值创建直方图
        axes[2].hist(rnormX, alpha=0.7)

        # 以及使用随机 Y 值的折线图
        axes[3].plot(x, rnormY, linewidth=2, alpha=0.7)

    style_name = style.split('-')[0]
    plt.suptitle(f'样式：{style_name}', fontsize=13)
    plt.show()

```

让我们来看看一些样式的效果。

首先，我们使用样式表 `seaborn` 绘制图形

```{code-cell} python3
draw_graphs(style='seaborn-v0_8')
```

我们可以使用 `grayscale` 来去除图形中的颜色

```{code-cell} python3
draw_graphs(style='grayscale')
```

以下是 `ggplot` 的效果

```{code-cell} python3
draw_graphs(style='ggplot')
```

我们还可以使用 `dark_background` 样式

```{code-cell} python3
draw_graphs(style='dark_background')
```

您可以使用该函数尝试列表中的其他样式。

如果您感兴趣，甚至可以创建自己的样式表。

您的样式表参数存储在类似字典的变量 `plt.rcParams` 中

```{code-cell} python3
---
tags: [hide-output]
---
 
print(plt.rcParams.keys())

```

您可以为样式表设置许多参数。

通过以下方式设置样式表的参数：

1. 创建您自己的 [`matplotlibrc` 文件](https://matplotlib.org/stable/users/explain/customizing.html)，或
2. 更新存储在类似字典变量 `plt.rcParams` 中的值

让我们使用第二种方法更改叠加密度线的样式

```{code-cell} python3
from cycler import cycler

# 设置为默认样式表
plt.style.use('default')

# 您可以使用键更新单个值：

# 将字体样式设置为斜体
plt.rcParams['font.style'] = 'italic'

# 更新线宽
plt.rcParams['lines.linewidth'] = 2


# 您也可以使用 update() 方法一次更新多个值：

parameters = {

    # 更改默认图形大小
    'figure.figsize': (5, 4),

    # 添加水平网格线
    'axes.grid': True,
    'axes.grid.axis': 'y',

    # 更新密度线的颜色
    'axes.prop_cycle': cycler('color', 
                            ['dimgray', 'slategrey', 'darkgray'])
}

plt.rcParams.update(parameters)


```

```{note} 

这些设置是`全局`的。

在 `.rcParams` 中更改参数后生成的任何图形都将受到该设置的影响。

```

```{code-cell} python3
fig, ax = plt.subplots()
x = np.linspace(-4, 4, 150)
for i in range(3):
    m, s = uniform(-1, 1), uniform(1, 2)
    y = norm.pdf(x, loc=m, scale=s)
    current_label = rf'$\mu = {m:.2}$'
    ax.plot(x, y, linewidth=2, alpha=0.6, label=current_label)
ax.legend()
plt.show()
```

再次应用 `default` 样式表将您的样式恢复为默认值

```{code-cell} python3

plt.style.use('default')

# 重置默认图形大小
plt.rcParams['figure.figsize'] = (10, 6)

```

## 延伸阅读

* [Matplotlib 图库](https://matplotlib.org/stable/gallery/index.html) 提供了许多示例。
* Nicolas Rougier、Mike Muller 和 Gael Varoquaux 编写的一份精彩的 [Matplotlib 教程](https://scipy-lectures.org/intro/matplotlib/index.html)。
* [mpltools](https://tonysyu.github.io/mpltools/index.html) 允许在图形样式之间轻松切换。
* [Seaborn](https://github.com/mwaskom/seaborn) 简化了 Matplotlib 中常见统计图形的绘制。

## 练习

```{exercise-start}
:label: mpl_ex1
```

在区间 $[0, 5]$ 上，对 `np.linspace(0, 2, 10)` 中的每个 $\theta$，绘制函数

$$
f(x) = \cos(\pi \theta x) \exp(-x)
$$

将所有曲线放在同一图形中。

输出结果应如下所示

```{image} /_static/lecture_specific/matplotlib/matplotlib_ex1.png
:scale: 130
:align: center
```

```{exercise-end}
```

```{solution-start} mpl_ex1
:class: dropdown
```

以下是一种解法

```{code-cell} ipython3
def f(x, θ):
    return np.cos(np.pi * θ * x ) * np.exp(- x)

θ_vals = np.linspace(0, 2, 10)
x = np.linspace(0, 5, 200)
fig, ax = plt.subplots()

for θ in θ_vals:
    ax.plot(x, f(x, θ))

plt.show()
```

```{solution-end}
```