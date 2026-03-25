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
  title: SciPy
  headings:
    Overview: 概述
    '{index}`SciPy <single: SciPy>` versus {index}`NumPy <single: NumPy>`': '{index}`SciPy <single: SciPy>` 与 {index}`NumPy <single: NumPy>` 的比较'
    Statistics: 统计
    Statistics::Random Variables and Distributions: 随机变量与分布
    Statistics::Alternative Syntax: 替代语法
    Statistics::Other Goodies in scipy.stats: scipy.stats 中的其他功能
    Roots and Fixed Points: 根与不动点
    'Roots and Fixed Points::{index}`Bisection <single: Bisection>`': '{index}`二分法 <single: Bisection>`'
    'Roots and Fixed Points::The {index}`Newton-Raphson Method <single: Newton-Raphson Method>`': '{index}`牛顿-拉弗森法 <single: Newton-Raphson Method>`'
    Roots and Fixed Points::Hybrid Methods: 混合方法
    Roots and Fixed Points::Multivariate Root-Finding: 多元求根
    Roots and Fixed Points::Fixed Points: 不动点
    '{index}`Optimization <single: Optimization>`': '{index}`优化 <single: Optimization>`'
    '{index}`Optimization <single: Optimization>`::Multivariate Optimization': 多元优化
    '{index}`Integration <single: Integration>`': '{index}`积分 <single: Integration>`'
    '{index}`Linear Algebra <single: Linear Algebra>`': '{index}`线性代数 <single: Linear Algebra>`'
    Exercises: 练习
---

(sp)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`SciPy <single: SciPy>`

```{index} single: Python; SciPy
```

除了 Anaconda 中已有的内容之外，本讲座还需要以下库：

```{code-cell} ipython3
:tags: [hide-output]

!pip install --upgrade quantecon
```

我们使用以下导入语句。

```{code-cell} ipython3
import numpy as np
import quantecon as qe
```

## 概述

[SciPy](https://scipy.org/) 构建在 NumPy 之上，为科学编程提供了常用工具，例如

* [线性代数](https://docs.scipy.org/doc/scipy/reference/linalg.html)
* [数值积分](https://docs.scipy.org/doc/scipy/reference/integrate.html)
* [插值](https://docs.scipy.org/doc/scipy/reference/interpolate.html)
* [优化](https://docs.scipy.org/doc/scipy/reference/optimize.html)
* [分布与随机数生成](https://docs.scipy.org/doc/scipy/reference/stats.html)
* [信号处理](https://docs.scipy.org/doc/scipy/reference/signal.html)
* 等等

与 NumPy 一样，SciPy 稳定、成熟且被广泛使用。

许多 SciPy 程序是对工业标准 Fortran 库（如 [LAPACK](https://en.wikipedia.org/wiki/LAPACK)、[BLAS](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms) 等）的薄封装。

实际上没有必要将 SciPy 作为一个整体来"学习"。

更常见的做法是大致了解库中有什么，然后在需要时查阅[文档](https://docs.scipy.org/doc/scipy/reference/index.html)。

在本讲座中，我们的目标仅是重点介绍该包中一些有用的部分。

## {index}`SciPy <single: SciPy>` 与 {index}`NumPy <single: NumPy>` 的比较

SciPy 是一个包含各种工具的包，这些工具构建在 NumPy 之上，使用其数组数据类型和相关功能。

````{note} 
在较旧版本的 SciPy（`scipy < 0.15.1`）中，导入该包时也会将 NumPy 符号导入全局命名空间，从 SciPy 初始化文件的以下摘录可以看出：

```python
from numpy import *
from numpy.random import rand, randn
from numpy.fft import fft, ifft
from numpy.lib.scimath import *
```

然而，更好的做法是显式使用 NumPy 功能。

```python
import numpy as np

a = np.identity(3)
```

较新版本的 SciPy（1.15+）不再自动导入 NumPy 符号。
````

SciPy 中有用的部分是其子包中的功能

* `scipy.optimize`、`scipy.integrate`、`scipy.stats` 等。

让我们来探索一些主要的子包。

## 统计

```{index} single: SciPy; Statistics
```

`scipy.stats` 子包提供了

* 大量的随机变量对象（密度函数、累积分布、随机抽样等）
* 一些估计方法
* 一些统计检验

### 随机变量与分布

回想一下，`numpy.random` 提供了生成随机变量的函数

```{code-cell} python3
np.random.beta(5, 5, size=3)
```

当 `a, b = 5, 5` 时，这从具有以下密度函数的分布中生成一个样本

```{math}
:label: betadist2

f(x; a, b) = \frac{x^{(a - 1)} (1 - x)^{(b - 1)}}
    {\int_0^1 u^{(a - 1)} (1 - u)^{(b - 1)} du}
    \qquad (0 \leq x \leq 1)
```

有时我们需要访问密度函数本身，或者累积分布函数、分位数等。

为此，我们可以使用 `scipy.stats`，它在一个统一的接口中提供了所有这些功能以及随机数生成。

以下是一个使用示例

```{code-cell} ipython
from scipy.stats import beta
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n

q = beta(5, 5)      # Beta(a, b)，其中 a = b = 5
obs = q.rvs(2000)   # 2000 个观测值
grid = np.linspace(0.01, 0.99, 100)

fig, ax = plt.subplots()
ax.hist(obs, bins=40, density=True)
ax.plot(grid, q.pdf(grid), 'k-', linewidth=2)
plt.show()
```

表示该分布的对象 `q` 还有其他有用的方法，包括

```{code-cell} python3
q.cdf(0.4)      # 累积分布函数
```

```{code-cell} python3
q.ppf(0.8)      # 分位数（逆累积分布函数）函数
```

```{code-cell} python3
q.mean()
```

创建这些表示分布的对象（类型为 `rv_frozen`）的一般语法为

> `name = scipy.stats.distribution_name(shape_parameters, loc=c, scale=d)`

其中 `distribution_name` 是 [scipy.stats](https://docs.scipy.org/doc/scipy/reference/stats.html) 中的一个分布名称。

`loc` 和 `scale` 参数将原始随机变量 $X$ 变换为 $Y = c + d X$。

### 替代语法

上述方法有另一种调用方式。

例如，生成上图的代码可以替换为

```{code-cell} python3
obs = beta.rvs(5, 5, size=2000)
grid = np.linspace(0.01, 0.99, 100)

fig, ax = plt.subplots()
ax.hist(obs, bins=40, density=True)
ax.plot(grid, beta.pdf(grid, 5, 5), 'k-', linewidth=2)
plt.show()
```

### scipy.stats 中的其他功能

`scipy.stats` 中有各种统计函数。

例如，`scipy.stats.linregress` 实现了简单线性回归

```{code-cell} python3
from scipy.stats import linregress

x = np.random.randn(200)
y = 2 * x + 0.1 * np.random.randn(200)
gradient, intercept, r_value, p_value, std_err = linregress(x, y)
gradient, intercept
```

要查看完整列表，请参阅[文档](https://docs.scipy.org/doc/scipy/reference/stats.html#statistical-functions-scipy-stats)。

## 根与不动点

实函数 $f$ 在 $[a,b]$ 上的**根**或**零点**是满足 $f(x)=0$ 的 $x \in [a, b]$。

例如，如果我们绘制函数

```{math}
:label: root_f

f(x) = \sin(4 (x - 1/4)) + x + x^{20} - 1
```

其中 $x \in [0,1]$，我们得到

```{code-cell} python3
f = lambda x: np.sin(4 * (x - 1/4)) + x + x**20 - 1
x = np.linspace(0, 1, 100)

fig, ax = plt.subplots()
ax.plot(x, f(x), label='$f(x)$')
ax.axhline(ls='--', c='k')
ax.set_xlabel('$x$', fontsize=12)
ax.set_ylabel('$f(x)$', fontsize=12)
ax.legend(fontsize=12)
plt.show()
```

唯一的根大约为 0.408。

让我们考虑一些求根的数值方法。

### {index}`二分法 <single: Bisection>`

```{index} single: SciPy; Bisection
```

数值求根最常见的算法之一是*二分法*。

为了理解这个思路，回忆一下一个众所周知的游戏：

* 玩家 A 心里想一个 1 到 100 之间的秘密数字
* 玩家 B 询问它是否小于 50
    * 如果是，B 询问它是否小于 25
    * 如果不是，B 询问它是否小于 75

如此反复。

这就是二分法。

以下是该算法在 Python 中的一个简单实现。

它适用于所有行为足够良好的单调递增连续函数，且满足 $f(a) < 0 < f(b)$

(bisect_func)=
```{code-cell} python3
def bisect(f, a, b, tol=10e-5):
    """
    实现二分法求根算法，假设 f 是 [a, b] 上的
    实值函数，满足 f(a) < 0 < f(b)。
    """
    lower, upper = a, b

    while upper - lower > tol:
        middle = 0.5 * (upper + lower)
        if f(middle) > 0:   # 根在 lower 和 middle 之间
            lower, upper = lower, middle
        else:               # 根在 middle 和 upper 之间
            lower, upper = middle, upper

    return 0.5 * (upper + lower)
```

让我们用 {eq}`root_f` 中定义的函数 $f$ 来测试它

```{code-cell} python3
bisect(f, 0, 1)
```

不出所料，SciPy 提供了自己的二分函数。

让我们用 {eq}`root_f` 中定义的同一函数 $f$ 来测试它

```{code-cell} python3
from scipy.optimize import bisect

bisect(f, 0, 1)
```

### {index}`牛顿-拉弗森法 <single: Newton-Raphson Method>`

```{index} single: SciPy; Newton-Raphson Method
```

另一种非常常见的求根算法是[牛顿-拉弗森法](https://en.wikipedia.org/wiki/Newton%27s_method)。

在 SciPy 中，该算法由 `scipy.optimize.newton` 实现。

与二分法不同，牛顿-拉弗森法使用局部斜率信息来尝试提高收敛速度。

让我们用上面定义的同一函数 $f$ 来研究这个问题。

在合适的初始条件下，我们得到收敛：

```{code-cell} python3
from scipy.optimize import newton

newton(f, 0.2)   # 在初始条件 x = 0.2 处开始搜索
```

但其他初始条件会导致不收敛：

```{code-cell} python3
newton(f, 0.7)   # 在 x = 0.7 处开始搜索
```

### 混合方法

数值方法的一个基本原则如下：

* 如果您对给定问题有具体的了解，您可能可以利用它来提高效率。
* 如果没有，那么算法的选择涉及速度和鲁棒性之间的权衡。

在实践中，大多数用于求根、优化和不动点的默认算法都使用*混合*方法。

这些方法通常将快速方法与鲁棒方法结合起来，方式如下：

1. 尝试使用快速方法
1. 检查诊断信息
1. 如果诊断信息不好，则切换到更鲁棒的算法

在 `scipy.optimize` 中，函数 `brentq` 就是这样一种混合方法，也是一个好的默认选择

```{code-cell} python3
from scipy.optimize import brentq

brentq(f, 0, 1)
```

这里找到了正确的解，速度比二分法更快：

```{code-cell} ipython
with qe.Timer(unit="milliseconds"):
    brentq(f, 0, 1)
```

```{code-cell} ipython
with qe.Timer(unit="milliseconds"):
    bisect(f, 0, 1)
```

### 多元求根

```{index} single: SciPy; Multivariate Root-Finding
```

使用 `scipy.optimize.fsolve`，这是 MINPACK 中混合方法的一个封装。

详情请参阅[文档](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.fsolve.html)。

### 不动点

实函数 $f$ 在 $[a,b]$ 上的**不动点**是满足 $f(x)=x$ 的 $x \in [a, b]$。

```{index} single: SciPy; Fixed Points
```

SciPy 也有一个用于寻找（标量）不动点的函数

```{code-cell} python3
from scipy.optimize import fixed_point

fixed_point(lambda x: x**2, 10.0)  # 10.0 是初始猜测值
```

如果结果不理想，您随时可以切换回 `brentq` 求根器，因为函数 $f$ 的不动点是 $g(x) := x - f(x)$ 的根。

## {index}`优化 <single: Optimization>`

```{index} single: SciPy; Optimization
```

大多数数值包只提供*最小化*函数。

最大化可以通过以下方式实现：域 $D$ 上函数 $f$ 的最大化点等同于 $-f$ 在 $D$ 上的最小化点。

最小化与求根密切相关：对于光滑函数，内部最优点对应于一阶导数的根。

上述速度/鲁棒性权衡在数值优化中同样存在。

除非您有一些先验信息可以利用，否则通常最好使用混合方法。

对于有约束的单变量（即标量）最小化，一个好的混合选项是 `fminbound`

```{code-cell} python3
from scipy.optimize import fminbound

fminbound(lambda x: x**2, -1, 2)  # 在 [-1, 2] 中搜索
```

### 多元优化

```{index} single: Optimization; Multivariate
```

多元局部优化器包括 `minimize`、`fmin`、`fmin_powell`、`fmin_cg`、`fmin_bfgs` 和 `fmin_ncg`。

有约束的多元局部优化器包括 `fmin_l_bfgs_b`、`fmin_tnc`、`fmin_cobyla`。

详情请参阅[文档](https://docs.scipy.org/doc/scipy/reference/optimize.html)。

## {index}`积分 <single: Integration>`

```{index} single: SciPy; Integration
```

大多数数值积分方法通过计算近似多项式的积分来工作。

所产生的误差取决于多项式对被积函数的拟合程度，而这又取决于被积函数的"规则程度"。

在 SciPy 中，数值积分的相关模块是 `scipy.integrate`。

单变量积分的一个好的默认选择是 `quad`

```{code-cell} python3
from scipy.integrate import quad

integral, error = quad(lambda x: x**2, 0, 1)
integral
```

实际上，`quad` 是 Fortran 库 QUADPACK 中一个非常标准的数值积分程序的接口。

它使用[克伦肖-柯蒂斯求积](https://en.wikipedia.org/wiki/Clenshaw-Curtis_quadrature)，基于切比雪夫多项式展开。

单变量积分还有其他选项——一个有用的选项是 `fixed_quad`，它速度快，因此在 `for` 循环内效果很好。

还有用于多元积分的函数。

详情请参阅[文档](https://docs.scipy.org/doc/scipy/reference/integrate.html)。

## {index}`线性代数 <single: Linear Algebra>`

```{index} single: SciPy; Linear Algebra
```

我们看到 NumPy 提供了一个名为 `linalg` 的线性代数模块。

SciPy 也提供了一个同名的线性代数模块。

后者不是前者的严格超集，但总体上具有更多功能。

我们留给您自行探索[可用程序集](https://docs.scipy.org/doc/scipy/reference/linalg.html)。

## 练习

前几个练习涉及在风险中性假设下为欧式看涨期权定价。价格满足

$$
P = \beta^n \mathbb E \max\{ S_n - K, 0 \}
$$

其中

1. $\beta$ 是贴现因子，
2. $n$ 是到期日，
2. $K$ 是行权价，以及
3. $\{S_t\}$ 是标的资产在每个时间 $t$ 的价格。

例如，如果看涨期权是以行权价 $K$ 购买亚马逊股票，则持有者有权（但没有义务）在 $n$ 天后以价格 $K$ 购买亚马逊 1 股股票。

因此，收益为 $\max\{S_n - K, 0\}$

价格是收益的期望值，贴现到当前价值。


```{exercise-start}
:label: sp_ex01
```

假设 $S_n$ 服从参数为 $\mu$ 和 $\sigma$ 的[对数正态](https://en.wikipedia.org/wiki/Log-normal_distribution)分布。设 $f$ 表示该分布的密度函数。则

$$
P = \beta^n \int_0^\infty \max\{x - K, 0\} f(x) dx
$$

在区间 $[0, 400]$ 上绘制函数

$$
g(x) = \beta^n  \max\{x - K, 0\} f(x)
$$ 

当 `μ, σ, β, n, K = 4, 0.25, 0.99, 10, 40` 时。

```{hint}
:class: dropdown

从 `scipy.stats` 中可以导入 `lognorm`，然后使用 `lognorm.pdf(x, σ, scale=np.exp(μ))` 来获得密度函数 $f$。
```

```{exercise-end}
```

```{solution-start} sp_ex01
:class: dropdown
```

以下是一种可能的解答

```{code-cell} ipython3
from scipy.integrate import quad
from scipy.stats import lognorm

μ, σ, β, n, K = 4, 0.25, 0.99, 10, 40

def g(x):
    return β**n * np.maximum(x - K, 0) * lognorm.pdf(x, σ, scale=np.exp(μ))

x_grid = np.linspace(0, 400, 1000)
y_grid = g(x_grid) 

fig, ax = plt.subplots()
ax.plot(x_grid, y_grid, label="$g$")
ax.legend()
plt.show()
```

```{solution-end}
```

```{exercise}
:label: sp_ex02

为了得到期权价格，使用 `scipy.integrate` 中的 `quad` 对该函数进行数值积分。

```

```{solution-start} sp_ex02
:class: dropdown
```

```{code-cell} ipython3
P, error = quad(g, 0, 1_000)
print(f"The numerical integration based option price is {P:.3f}")
```

```{solution-end}
```

```{exercise}
:label: sp_ex03

尝试使用蒙特卡洛方法来计算期权价格中的期望项，而不是使用 `quad`，从而得到类似的结果。

特别地，利用以下事实：如果 $S_n^1, \ldots, S_n^M$ 是从上述指定的对数正态分布中独立抽取的样本，那么根据大数定律，

$$ \mathbb E \max\{ S_n - K, 0 \} 
    \approx
    \frac{1}{M} \sum_{m=1}^M \max \{S_n^m - K, 0 \}
    $$
    
设 `M = 10_000_000`

```

```{solution-start} sp_ex03
:class: dropdown
```

以下是一种解答：

```{code-cell} ipython3
M = 10_000_000
S = np.exp(μ + σ * np.random.randn(M))
return_draws = np.maximum(S - K, 0)
P = β**n * np.mean(return_draws) 
print(f"The Monte Carlo option price is {P:3f}")
```


```{solution-end}
```



```{exercise}
:label: sp_ex1

在{ref}`本讲座 <functions>`中，我们讨论了{ref}`递归函数调用 <recursive_functions>`的概念。

尝试编写上面{ref}`描述的 <bisect_func>`自制二分函数的递归实现。

用函数 {eq}`root_f` 对其进行测试。
```

```{solution-start} sp_ex1
:class: dropdown
```

以下是一个合理的解答：


```{code-cell} python3
def bisect(f, a, b, tol=10e-5):
    """
    实现二分法求根算法，假设 f 是 [a, b] 上的
    实值函数，满足 f(a) < 0 < f(b)。
    """
    lower, upper = a, b
    if upper - lower < tol:
        return 0.5 * (upper + lower)
    else:
        middle = 0.5 * (upper + lower)
        print(f'Current mid point = {middle}')
        if f(middle) > 0:   # 意味着根在 lower 和 middle 之间
            return bisect(f, lower, middle)
        else:               # 意味着根在 middle 和 upper 之间
            return bisect(f, middle, upper)
```

我们可以如下测试它

```{code-cell} python3
f = lambda x: np.sin(4 * (x - 0.25)) + x + x**20 - 1
bisect(f, 0, 1)
```

```{solution-end}
```