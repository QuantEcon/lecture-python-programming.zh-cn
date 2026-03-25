---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
translation:
  title: SymPy
  headings:
    Overview: 概述
    Getting Started: 入门
    Symbolic algebra: 符号代数
    Symbolic algebra::Symbols: 符号
    Symbolic algebra::Expressions: 表达式
    Symbolic algebra::Equations: 方程
    'Symbolic algebra::Equations::Example: fixed point computation': 示例：不动点计算
    Symbolic algebra::Inequalities and logic: 不等式与逻辑
    Symbolic algebra::Series: 级数
    'Symbolic algebra::Series::Example: bank deposits': 示例：银行存款
    'Symbolic algebra::Series::Example: discrete random variable': 示例：离散随机变量
    Symbolic Calculus: 符号微积分
    Symbolic Calculus::Limits: 极限
    Symbolic Calculus::Derivatives: 导数
    Symbolic Calculus::Integrals: 积分
    Plotting: 绘图
    'Application: Two-person Exchange Economy': 应用：两人交换经济
    Exercises: 练习
---

(sympy)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`SymPy <single: SymPy>`

```{index} single: Python; SymPy
```

## 概述

与处理数值的数值库不同，[SymPy](https://www.sympy.org/en/index.html) 专注于直接操作数学符号和表达式。

SymPy 提供了[多种功能](https://www.sympy.org/en/features.html)，包括：

- 符号表达式
- 方程求解
- 化简
- 微积分
- 矩阵
- 离散数学等

这些功能使 SymPy 成为 Mathematica 等专有符号计算软件的热门开源替代品。

在本讲座中，我们将探索 SymPy 的一些功能，并演示如何使用基本的 SymPy 函数来求解经济模型。

## 入门

首先导入库并初始化符号输出的打印器

```{code-cell} ipython3
from sympy import *
from sympy.plotting import plot, plot3d_parametric_line, plot3d
from sympy.solvers.inequalities import reduce_rational_inequalities
from sympy.stats import Poisson, Exponential, Binomial, density, moment, E, cdf

import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n

# 启用 mathjax 打印器
init_printing(use_latex='mathjax')
```

## 符号代数

### 符号

首先初始化一些符号以供使用

```{code-cell} ipython3
x, y, z = symbols('x y z')
```

符号是 SymPy 中符号计算的基本单元。

### 表达式

现在我们可以使用符号 `x`、`y` 和 `z` 来构建表达式和方程。

这里我们先构建一个简单的表达式

```{code-cell} ipython3
expr = (x+y) ** 2
expr
```

我们可以用 `expand` 函数展开这个表达式

```{code-cell} ipython3
expand_expr = expand(expr)
expand_expr
```

并用 `factor` 函数将其分解回因式分解的形式

```{code-cell} ipython3
factor(expand_expr)
```

我们可以求解这个表达式

```{code-cell} ipython3
solve(expr)
```

注意，这等价于求解以下关于 `x` 的方程

$$
(x + y)^2 = 0 
$$

```{note}
[Solvers](https://docs.sympy.org/latest/modules/solvers/index.html) 是一个重要的模块，包含求解不同类型方程的工具。

SymPy 中提供了多种求解器，具体取决于问题的性质。
```

### 方程

SymPy 提供了多个函数来操作方程。

让我们用之前定义的表达式建立一个方程

```{code-cell} ipython3
eq = Eq(expr, 0)
eq
```

对 $x$ 求解这个方程，结果与直接求解表达式相同

```{code-cell} ipython3
solve(eq, x)
```

SymPy 可以处理有多个解的方程

```{code-cell} ipython3
eq = Eq(expr, 1)
solve(eq, x)
```

`solve` 函数还可以将多个方程组合在一起，求解方程组

```{code-cell} ipython3
eq2 = Eq(x, y)
eq2
```

```{code-cell} ipython3
solve([eq, eq2], [x, y])
```

我们也可以通过将 $x$ 替换为 $y$ 来求 $y$ 的值

```{code-cell} ipython3
expr_sub = expr.subs(x, y)
expr_sub
```

```{code-cell} ipython3
solve(Eq(expr_sub, 1))
```

下面是另一个包含符号 `x` 以及函数 `sin`、`cos` 和 `tan` 的方程示例，使用 `Eq` 函数构建

```{code-cell} ipython3
# 创建一个方程
eq = Eq(cos(x) / (tan(x)/sin(x)), 0)
eq
```

现在我们使用 `simplify` 函数化简这个方程

```{code-cell} ipython3
# 化简一个表达式
simplified_expr = simplify(eq)
simplified_expr
```

再次使用 `solve` 函数求解这个方程

```{code-cell} ipython3
# 求解方程
sol = solve(eq, x)
sol
```

SymPy 还可以处理涉及三角函数和复数的更复杂的方程。

我们使用[欧拉公式](https://en.wikipedia.org/wiki/Euler%27s_formula)来演示这一点

```{code-cell} ipython3
# 'I' 代表虚数单位 i
euler = cos(x) + I*sin(x)
euler
```

```{code-cell} ipython3
simplify(euler)
```

如果你感兴趣，我们建议你阅读关于[三角函数和复数](https://python.quantecon.org/complex_and_trig.html)的讲座。

#### 示例：不动点计算

不动点计算在经济学和金融学中被广泛应用。

这里我们求解索洛-斯旺增长动态的不动点：

$$
k_{t+1}=s f\left(k_t\right)+(1-\delta) k_t, \quad t=0,1, \ldots
$$

其中 $k_t$ 是资本存量，$f$ 是生产函数，$\delta$ 是折旧率。

我们感兴趣的是计算这一动态的不动点，即满足 $k_{t+1} = k_t$ 的 $k$ 值。

在 $f(k) = Ak^\alpha$ 的条件下，我们可以用纸笔推导出唯一不动点 $k^*$：

$$
k^*:=\left(\frac{s A}{\delta}\right)^{1 /(1-\alpha)}
$$ 

这可以在 SymPy 中轻松计算

```{code-cell} ipython3
A, s, k, α, δ = symbols('A s k^* α δ')
```

现在我们求解不动点 $k^*$

$$
k^* = sA(k^*)^{\alpha}+(1-\delta) k^*
$$

```{code-cell} ipython3
# 定义索洛-斯旺增长动态
solow = Eq(s*A*k**α + (1-δ)*k, k)
solow
```

```{code-cell} ipython3
solve(solow, k)
```

### 不等式与逻辑

SymPy 还允许用户定义不等式和集合运算符，并提供了丰富的[操作](https://docs.sympy.org/latest/modules/solvers/inequalities.html)。

```{code-cell} ipython3
reduce_inequalities([2*x + 5*y <= 30, 4*x + 2*y <= 20], [x])
```

```{code-cell} ipython3
And(2*x + 5*y <= 30, x > 0)
```

### 级数

级数在经济学和统计学中被广泛使用，从资产定价到离散随机变量的期望。

我们可以使用 `Sum` 函数和 `Indexed` 符号构建简单的求和级数

```{code-cell} ipython3
x, y, i, j = symbols("x y i j")
sum_xy = Sum(Indexed('x', i)*Indexed('y', j), 
            (i, 0, 3),
            (j, 0, 3))
sum_xy
```

为了计算这个求和，我们可以对公式进行 [`lambdify`](https://docs.sympy.org/latest/modules/utilities/lambdify.html#sympy.utilities.lambdify.lambdify) 处理。

经过 lambdify 处理的表达式可以接受 $x$ 和 $y$ 的数值输入并计算结果

```{code-cell} ipython3
sum_xy = lambdify([x, y], sum_xy)
grid = np.arange(0, 4, 1)
sum_xy(grid, grid)
```

#### 示例：银行存款

假设一家银行在时间 $t$ 的存款为 $D_0$。

它将 $(1-r)$ 的存款贷出，并保留 $r$ 比例的现金作为准备金。

其在无限时间范围内的存款可以写成

$$
\sum_{i=0}^\infty (1-r)^i D_0
$$

让我们计算时间 $t$ 的存款

```{code-cell} ipython3
D = symbols('D_0')
r = Symbol('r', positive=True)
Dt = Sum('(1 - r)^i * D_0', (i, 0, oo))
Dt
```

我们可以调用 `doit` 方法来计算该级数

```{code-cell} ipython3
Dt.doit()
```

化简上述表达式得到

```{code-cell} ipython3
simplify(Dt.doit())
```

这与[等比数列](https://intro.quantecon.org/geom_series.html#example-the-money-multiplier-in-fractional-reserve-banking)讲座中关于部分准备金银行制度货币乘数的解一致。


#### 示例：离散随机变量

在下面的例子中，我们计算离散随机变量的期望。

让我们定义一个服从[泊松分布](https://en.wikipedia.org/wiki/Poisson_distribution)的离散随机变量 $X$：

$$
f(x) = \frac{\lambda^x e^{-\lambda}}{x!}, \quad x = 0, 1, 2, \ldots
$$

```{code-cell} ipython3
λ = symbols('lambda')

# 将符号 x 细化为正整数
x = Symbol('x', integer=True, positive=True)
pmf = λ**x * exp(-λ) / factorial(x)
pmf
```

我们可以验证所有可能取值的概率之和是否等于 $1$：

$$
\sum_{x=0}^{\infty} f(x) = 1
$$

```{code-cell} ipython3
sum_pmf = Sum(pmf, (x, 0, oo))
sum_pmf.doit()
```

该分布的期望为：

$$
E(X) = \sum_{x=0}^{\infty} x f(x) 
$$

```{code-cell} ipython3
fx = Sum(x*pmf, (x, 0, oo))
fx.doit()
```

SymPy 包含一个名为 [`Stats`](https://docs.sympy.org/latest/modules/stats.html) 的统计子模块。

`Stats` 提供了内置分布和概率分布函数。

上述计算也可以使用 `Stats` 模块中的期望函数 `E` 简化为一行代码

```{code-cell} ipython3
λ = Symbol("λ", positive = True)

# 使用 sympy.stats.Poisson() 方法
X = Poisson("x", λ)
E(X)
```

## 符号微积分

SymPy 允许我们执行各种微积分运算，例如极限、求导和积分。


### 极限

我们可以使用 `limit` 函数计算给定表达式的极限

```{code-cell} ipython3
# 定义一个表达式
f = x**2 / (x-1)

# 计算极限
lim = limit(f, x, 0)
lim
```

### 导数

我们可以使用 `diff` 函数对任意 SymPy 表达式求导

```{code-cell} ipython3
# 对函数关于 x 求导
df = diff(f, x)
df
```

### 积分

我们可以使用 `integrate` 函数计算定积分和不定积分

```{code-cell} ipython3
# 计算不定积分
indef_int = integrate(df, x)
indef_int
```

让我们使用这个函数来计算[指数分布](https://en.wikipedia.org/wiki/Exponential_distribution)的矩生成函数，其概率密度函数为：

$$
f(x) = \lambda e^{-\lambda x}, \quad x \ge 0
$$

```{code-cell} ipython3
λ = Symbol('lambda', positive=True)
x = Symbol('x', positive=True)
pdf = λ * exp(-λ*x)
pdf
```

```{code-cell} ipython3
t = Symbol('t', positive=True)
moment_t = integrate(exp(t*x) * pdf, (x, 0, oo))
simplify(moment_t)
```

注意，我们也可以使用 `Stats` 模块来计算矩

```{code-cell} ipython3
X = Exponential(x, λ)
```

```{code-cell} ipython3
moment(X, 1)
```

```{code-cell} ipython3
E(X**t)
```

使用 `integrate` 函数，我们可以推导出 $\lambda = 0.5$ 时指数分布的累积密度函数

```{code-cell} ipython3
λ_pdf = pdf.subs(λ, 1/2)
λ_pdf
```

```{code-cell} ipython3
integrate(λ_pdf, (x, 0, 4))
```

使用 `Stats` 模块中的 `cdf` 函数可以得到相同的结果

```{code-cell} ipython3
cdf(X, 1/2)
```

```{code-cell} ipython3
# 代入 z 的值
λ_cdf = cdf(X, 1/2)(4)
λ_cdf
```

```{code-cell} ipython3
# 代入 λ
λ_cdf.subs({λ: 1/2})
```

## 绘图

SymPy 提供了强大的绘图功能。

首先我们使用 `plot` 函数绘制一个简单的函数

```{code-cell} ipython3
f = sin(2 * sin(2 * sin(2 * sin(x))))
p = plot(f, (x, -10, 10), show=False)
p.title = '简单图形'
p.show()
```

与 Matplotlib 类似，SymPy 提供了自定义图形的接口

```{code-cell} ipython3
plot_f = plot(f, (x, -10, 10), 
              xlabel='', ylabel='', 
              legend = True, show = False)
plot_f[0].label = 'f(x)'
df = diff(f)
plot_df = plot(df, (x, -10, 10), 
            legend = True, show = False)
plot_df[0].label = 'f\'(x)'
plot_f.append(plot_df[0])
plot_f.show()
```

它还支持绘制隐函数和不等式的可视化

```{code-cell} ipython3
p = plot_implicit(Eq((1/x + 1/y)**2, 1))
```

```{code-cell} ipython3
p = plot_implicit(And(2*x + 5*y <= 30, 4*x + 2*y >= 20),
                     (x, -1, 10), (y, -10, 10))
```

以及三维空间中的可视化

```{code-cell} ipython3
p = plot3d(cos(2*x + y), zlabel='')
```

## 应用：两人交换经济

设想一个纯交换经济，有两个人（$a$ 和 $b$）和两种商品，以比例表示（$x$ 和 $y$）。

他们可以根据各自的偏好相互交换商品。

假设消费者的效用函数为

$$
u_a(x, y) = x^{\alpha} y^{1-\alpha}
$$

$$
u_b(x, y) = (1 - x)^{\beta} (1 - y)^{1-\beta}
$$

其中 $\alpha, \beta \in (0, 1)$。

首先定义符号和效用函数

```{code-cell} ipython3
# 定义符号和效用函数
x, y, α, β = symbols('x, y, α, β')
u_a = x**α * y**(1-α)
u_b = (1 - x)**β * (1 - y)**(1 - β)
```

```{code-cell} ipython3
u_a
```

```{code-cell} ipython3
u_b
```

我们感兴趣的是商品 $x$ 和 $y$ 的帕累托最优配置。

注意，当在给定另一方配置的情况下，某一方的配置最优时，该点即为帕累托有效点。

用边际效用来表示：

$$
\frac{\frac{\partial u_a}{\partial x}}{\frac{\partial u_a}{\partial y}} = \frac{\frac{\partial u_b}{\partial x}}{\frac{\partial u_b}{\partial y}}
$$

```{code-cell} ipython3
# 当在给定另一方配置的情况下某一方的配置最优时，该点为帕累托有效点

pareto = Eq(diff(u_a, x)/diff(u_a, y), 
            diff(u_b, x)/diff(u_b, y))
pareto
```

```{code-cell} ipython3
# 求解方程
sol = solve(pareto, y)[0]
sol
```

让我们使用 SymPy 计算 $\alpha = \beta = 0.5$ 时经济的帕累托最优配置（契约曲线）

```{code-cell} ipython3
# 代入 α = 0.5 和 β = 0.5
sol.subs({α: 0.5, β: 0.5})
```

我们可以使用这个结果来可视化不同参数下的更多契约曲线

```{code-cell} ipython3
# 绘制一系列 α 和 β 的取值
params = [{α: 0.5, β: 0.5}, 
          {α: 0.1, β: 0.9},
          {α: 0.1, β: 0.8},
          {α: 0.8, β: 0.9},
          {α: 0.4, β: 0.8}, 
          {α: 0.8, β: 0.1},
          {α: 0.9, β: 0.8},
          {α: 0.8, β: 0.4},
          {α: 0.9, β: 0.1}]

p = plot(xlabel='x', ylabel='y', show=False)

for param in params:
    p_add = plot(sol.subs(param), (x, 0, 1), 
                 show=False)
    p.append(p_add[0])
p.show()
```

我们邀请你调整参数，观察契约曲线如何变化，并思考以下两个问题：

- 你能想到用 `numpy` 绘制相同图形的方法吗？
- 编写一个 `numpy` 实现会有多困难？

## 练习

```{exercise}
:label: sympy_ex1

洛必达法则指出，对于两个函数 $f(x)$ 和 $g(x)$，如果 $\lim_{x \to a} f(x) = \lim_{x \to a} g(x) = 0$ 或 $\pm \infty$，则

$$
\lim_{x \to a} \frac{f(x)}{g(x)} = \lim_{x \to a} \frac{f'(x)}{g'(x)}
$$

使用 SymPy 验证以下函数的洛必达法则

$$
f(x) = \frac{y^x - 1}{x}
$$

当 $x$ 趋近于 $0$ 时
```

```{solution-start} sympy_ex1
:class: dropdown
```

首先定义函数

```{code-cell} ipython3
f_upper = y**x - 1
f_lower = x
f = f_upper/f_lower
f
```

SymPy 足够智能，可以直接求解这个极限

```{code-cell} ipython3
lim = limit(f, x, 0)
lim
```

我们对比洛必达法则给出的结果

```{code-cell} ipython3
lim = limit(diff(f_upper, x)/
            diff(f_lower, x), x, 0)
lim
```

```{solution-end}
```

```{exercise}
:label: sympy_ex2

[最大似然估计（MLE）](https://python.quantecon.org/mle.html) 是一种估计统计模型参数的方法。

它通常涉及最大化对数似然函数并求解一阶导数。

二项分布的概率质量函数为

$$
f(x; n, θ) = \frac{n!}{x!(n-x)!}θ^x(1-θ)^{n-x}
$$

其中 $n$ 是试验次数，$x$ 是成功次数。

假设我们观测到一系列二元结果，在 $n$ 次试验中有 $x$ 次成功。

使用 SymPy 计算 $θ$ 的最大似然估计
```

```{solution-start} sympy_ex2
:class: dropdown
```

首先定义二项分布

```{code-cell} ipython3
n, x, θ = symbols('n x θ')

binomial_factor = (factorial(n)) / (factorial(x)*factorial(n-r))
binomial_factor
```

```{code-cell} ipython3
bino_dist = binomial_factor * ((θ**x)*(1-θ)**(n-x))
bino_dist
```

现在计算对数似然函数并求解结果

```{code-cell} ipython3
log_bino_dist = log(bino_dist)
```

```{code-cell} ipython3
log_bino_diff = simplify(diff(log_bino_dist, θ))
log_bino_diff
```

```{code-cell} ipython3
solve(Eq(log_bino_diff, 0), θ)[0]
```

```{solution-end}
```