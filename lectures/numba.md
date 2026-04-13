---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.7
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
translation:
  title: Numba
  headings:
    Overview: 概述
    Compiling Functions: 编译函数
    Compiling Functions::An Example: 示例
    Compiling Functions::How and When it Works: 工作原理与适用时机
    Type Inference: 类型推断
    Dangers and Limitations: 危险与局限
    Dangers and Limitations::Limitations: 局限性
    'Dangers and Limitations::A Gotcha: Global Variables': 一个陷阱：全局变量
    Dangers and Limitations::Caching Compiled Code: 缓存编译代码
    Multithreaded Loops in Numba: Numba 中的多线程循环
    Exercises: 练习
---

(numba_lecture)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Numba

除了 Anaconda 中已有的库之外，本讲座还需要以下库：

```{code-cell} ipython3
:tags: [hide-output]

!pip install quantecon
```

请同时确保您安装了最新版本的 Anaconda，因为旧版本是{doc}`常见错误来源 <troubleshooting>`。

让我们从一些导入开始：

```{code-cell} ipython3
import numpy as np
import quantecon as qe
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
import matplotlib.font_manager  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n
```

## 概述

在 {doc}`之前的讲座 <need_for_speed>` 中，我们讨论了向量化，这是一种通过将数组处理操作批量发送到高效底层代码来提高执行速度的方法。

然而，正如 {ref}`在那次讲座中所讨论的 <numba-p_c_vectorization>`，传统的向量化方案（例如在 MATLAB 和 NumPy 中）存在一些弱点。

* 对于复合数组操作，内存消耗极大
* 对某些算法无效甚至无法应用。

绕过这些问题的一种方法是使用 [Numba](https://numba.pydata.org/)，这是一个面向数值计算的 Python **即时（JIT）编译器**。

Numba 在运行时将函数编译为本地机器码指令。

编译成功后，Numba 的性能将与低级语言生成的机器码相当。

此外，Numba 还可以完成其他有用的技巧，例如 {ref}`多线程 <multithreading>` 或通过 `numba.cuda` 与 GPU 进行交互。

Numba 的 JIT 编译器在许多方面与 Julia 中的 JIT 编译器类似。

主要区别在于它的目标更为保守，只尝试编译语言的一个较小子集。

虽然这听起来像是一个缺陷，但在某些方面却是一个优势。

Numba 精简、易用，且非常擅长它所做的事情。

本讲座将介绍核心思路。

(numba_link)=
## {index}`编译函数 <single: Compiling Functions>`

```{index} single: Python; Numba
```


(quad_map_eg)=
### 示例

让我们考虑一个难以向量化的问题（即难以交给数组处理操作来完成）。

该问题涉及通过二次映射生成轨迹

$$
    x_{t+1} = \alpha x_t (1 - x_t)
$$

在下文中，我们设置

```{code-cell} ipython3
α = 4.0
```

以下是从 $x_0 = 0.1$ 出发的典型轨迹图，横轴为 $t$

```{code-cell} ipython3
def qm(x0, n):
    x = np.empty(n+1)
    x[0] = x0
    for t in range(n):
      x[t+1] = α * x[t] * (1 - x[t])
    return x

x = qm(0.1, 250)
fig, ax = plt.subplots()
ax.plot(x, 'b-', lw=2, alpha=0.8)
ax.set_xlabel('$t$', fontsize=12)
ax.set_ylabel('$x_{t}$', fontsize = 12)
plt.show()
```

要使用 Numba 加速函数 `qm`，第一步是

```{code-cell} ipython3
from numba import jit

qm_numba = jit(qm)
```

函数 `qm_numba` 是 `qm` 的一个版本，它被"定向"用于即时编译（JIT-compilation）。

我们将在稍后解释这意味着什么。

让我们对这两个版本的相同函数调用进行计时和比较，首先从原始函数 `qm` 开始：

```{code-cell} ipython3
n = 10_000_000

with qe.Timer() as timer1:
    qm(0.1, int(n))
time1 = timer1.elapsed
```

现在让我们尝试 qm_numba

```{code-cell} ipython3
with qe.Timer() as timer2:
    qm_numba(0.1, int(n))
time2 = timer2.elapsed
```

这已经是非常大的速度提升。

事实上，下一次及之后的每次运行都会更快，因为函数已经被编译并保存在内存中：

(qm_numba_result)=

```{code-cell} ipython3
with qe.Timer() as timer3:
    qm_numba(0.1, int(n))
time3 = timer3.elapsed
```

```{code-cell} ipython3
time1 / time3  # 计算加速比
```

### 工作原理与适用场景

Numba 尝试使用 [LLVM Project](https://llvm.org/) 提供的基础设施生成快速的机器代码。

它通过动态推断类型信息来实现这一目标。

（有关类型的讨论，请参阅我们 {doc}`早期关于科学计算的讲座 <need_for_speed>`。）

基本思路如下：

* Python 非常灵活，因此我们可以用多种类型调用函数 qm。
    * 例如，`x0` 可以是 NumPy 数组或列表，`n` 可以是整数或浮点数，等等。
* 这使得*预*编译函数（即在运行时之前编译）变得困难。
* 然而，当我们实际调用函数时，例如运行 `qm(0.5, 10)`，`x0` 和 `n` 的类型就变得明确了。
* 此外，一旦输入类型已知，`qm` 中*其他变量*的类型*便可以被推断出来*。
* 因此，Numba 和其他 JIT 编译器的策略是等待这一时刻，然后再编译函数。

这就是所谓的"即时"编译（just-in-time compilation）。

请注意，如果你调用 `qm(0.5, 10)` 后紧接着调用 `qm(0.9, 20)`，编译只在第一次调用时发生。

这是因为编译后的代码会被缓存并在需要时重复使用。

这就是为什么在上述代码中，`time3` 小于 `time2`。

这是因为编译后的代码会被缓存并在需要时重复使用。

这就是为什么在上面的代码中，`time3` 小于 `time2`。

```{admonition} 备注
在实践中，我们不直接写 `qm_numba = jit(qm)`，而是使用*装饰器*语法，在函数定义前加上 `@jit`。这等价于在定义之后添加 `qm = jit(qm)`。我们在本讲座的其余部分都使用这种语法。（有关装饰器的更多内容，请参见 {doc}`python_advanced_features`。）
```

在实践中，这通常使用另一种*装饰器*语法来完成。

（我们在 {doc}`单独的讲座 <python_advanced_features>` 中讨论装饰器，但在此阶段您可以跳过细节。）

具体来说，要将函数定向为 JIT 编译，我们可以在函数定义前放置 `@jit`。

以下是 `qm` 的写法

```{code-cell} ipython3
@jit
def qm(x0, n):
    x = np.empty(n+1)
    x[0] = x0
    for t in range(n):
        x[t+1] = α * x[t] * (1 - x[t])
    return x
```

这等价于在函数定义后添加 `qm = jit(qm)`。

以下代码现在使用 JIT 编译版本：

```{code-cell} ipython3
with qe.Timer(precision=4):
    qm(0.1, 100_000)
```

```{code-cell} ipython3
with qe.Timer(precision=4):
    qm(0.1, 100_000)
```

Numba 还为装饰器提供了几个参数以加速计算和缓存函数——请参阅 [这里](https://numba.readthedocs.io/en/stable/user/performance-tips.html)。

## 类型推断

成功的类型推断是 JIT 编译的关键部分。

可以想象，对于简单的 Python 对象（例如，浮点数和整数等简单标量数据类型），推断类型更为容易。

Numba 也与 NumPy 数组配合良好，因为它们具有明确定义的类型。

在理想情况下，Numba 可以推断出所有必要的类型信息。

这使它能够生成本地机器码，而无需调用 Python 运行时环境。

当 Numba 无法推断所有类型信息时，它将引发错误。

例如，在下面的示例中，Numba 在编译 `iterate` 时无法确定函数 `g` 的类型

```{code-cell} ipython3
@jit
def iterate(f, x0, n):
    x = x0
    for t in range(n):
        x = f(x)
    return x

# 未经 jit 编译
def g(x):
    return np.cos(x) - 2 * np.sin(x)

# 这段代码会抛出错误
try:
    iterate(g, 0.5, 100)
except Exception as e:
    print(e)
```

我们可以通过编译 `g` 来轻松修复这个错误。

```{code-cell} ipython3
@jit
def g(x):
    return np.cos(x) - 2 * np.sin(x)

iterate(g, 0.5, 100)
```

## 危险与局限

让我们补充一些注意事项。

### 局限性

正如我们所见，Numba 需要推断所有变量的类型信息以生成快速的机器级指令。

对于较大的例程或使用外部库的例程，此过程很容易失败。

因此，最好专注于加速代码中小而关键的片段。

这将比在 Python 程序中大量使用 `@jit` 语句带来更好的性能。

### 一个陷阱：全局变量

以下是使用 Numba 时需要注意的另一件事。

考虑以下示例

```{code-cell} ipython3
a = 1

@jit
def add_a(x):
    return a + x

print(add_a(10))
```

```{code-cell} ipython3
a = 2

print(add_a(10))
```

注意，更改全局变量对函数返回值没有任何影响。

当 Numba 为函数编译机器码时，它将全局变量视为常量，以确保类型稳定性。

### 缓存编译代码

默认情况下，Numba 在每次启动新的 Python 会话时都会重新编译函数。

为了避免这种开销，可以向装饰器传递 `cache=True`：

```{code-cell} ipython3
@jit(cache=True)
def qm(x0, n):
    x = np.empty(n+1)
    x[0] = x0
    for t in range(n):
        x[t+1] = α * x[t] * (1 - x[t])
    return x
```

这会将编译后的代码存储在磁盘上，以便后续会话可以跳过编译步骤。

(multithreading)=
## Numba 中的多线程循环

除了 JIT 编译之外，Numba 还为 CPU 和 GPU 上的并行计算提供了支持。

Numba 中在 CPU 上进行并行化的关键工具是 `prange` 函数，它告诉 Numba 在可用的核心上并行执行循环迭代。

为了说明，让我们首先看一个简单的单线程（即非并行化）代码片段。

该代码通过以下规则模拟家庭财富 $w_t$ 的更新

$$
w_{t+1} = R_{t+1} s w_t + y_{t+1}
$$

其中

* $R$ 是资产的总回报率
* $s$ 是家庭的储蓄率，以及
* $y$ 是劳动收入。

我们将 $R$ 和 $y$ 均建模为来自对数正态分布的独立抽样。

以下是代码：

```{code-cell} ipython3
from numba import jit

@jit
def h(w, r=0.1, s=0.3, v1=0.1, v2=1.0):
    """
    Updates household wealth.
    """

    # 抽取冲击
    R = np.exp(v1 * np.random.randn()) * (1 + r)
    y = np.exp(v2 * np.random.randn())

    # 更新财富
    w = R * s * w + y
    return w
```

让我们看看在此规则下财富如何演变。

```{code-cell} ipython3
fig, ax = plt.subplots()

T = 100
w = np.empty(T)
w[0] = 5
for t in range(T-1):
    w[t+1] = h(w[t])

ax.plot(w)
ax.set_xlabel('$t$', fontsize=12)
ax.set_ylabel('$w_{t}$', fontsize=12)
plt.show()
```

现在，假设我们有一个庞大的家庭群体，并且想知道中位财富将是多少。

这个问题很难用纸笔求解，因此我们将使用模拟：

1. 向前模拟大量家庭
2. 计算中位财富

以下是代码：

```{code-cell} ipython3
@jit
def compute_long_run_median(w0=1, T=1000, num_reps=50_000):

    obs = np.empty(num_reps)
    for i in range(num_reps):
        w = w0
        for t in range(T):
            w = h(w)
        obs[i] = w

    return np.median(obs)
```

让我们看看运行速度：

```{code-cell} ipython3
with qe.Timer():
    compute_long_run_median()
```

为了加速这个过程，我们将通过多线程对其进行并行化。

为此，我们添加 `parallel=True` 标志并将 `range` 更改为 `prange`：

```{code-cell} ipython3
from numba import prange

@jit(parallel=True)
def compute_long_run_median_parallel(w0=1, T=1000, num_reps=50_000):

    obs = np.empty(num_reps)
    for i in prange(num_reps):
        w = w0
        for t in range(T):
            w = h(w)
        obs[i] = w

    return np.median(obs)
```

让我们看看计时结果：

```{code-cell} ipython3
with qe.Timer():
    compute_long_run_median_parallel()
```

速度提升非常显著。

注意，我们是跨家庭进行并行化，而非跨时间——单个家庭跨时期的更新本质上是顺序的。

## 练习

```{exercise}
:label: speed_ex1

{ref}`之前 <pbe_ex5>` 我们考虑了如何用蒙特卡洛方法近似 $\pi$。

在这里使用相同的思路，但使用 Numba 使代码高效。

当样本量较大时，比较有无 Numba 的速度。
```

```{solution-start} speed_ex1
:class: dropdown
```

以下是一种解法：

```{code-cell} ipython3
@jit
def calculate_pi(n=1_000_000):
    count = 0
    for i in range(n):
        u, v = np.random.uniform(0, 1), np.random.uniform(0, 1)
        d = np.sqrt((u - 0.5)**2 + (v - 0.5)**2)
        if d < 0.5:
            count += 1

    area_estimate = count / n
    return area_estimate * 4  # 除以半径的平方
```

现在让我们看看运行速度：

```{code-cell} ipython3
with qe.Timer():
    calculate_pi()
```

```{code-cell} ipython3
with qe.Timer():
    calculate_pi()
```

如果我们通过删除 `@jit` 来关闭 JIT 编译，代码在我们的机器上大约需要慢 150 倍。

因此，通过添加四个字符，我们获得了 2 个数量级的速度提升。

```{solution-end}
```

```{exercise-start}
:label: speed_ex2
```

在 [Python 定量经济学入门](https://intro.quantecon.org/intro.html) 讲座系列中，您可以学习到关于有限状态马尔可夫链的所有知识。

现在，让我们专注于模拟一个非常简单的此类链的示例。

假设一种资产的回报波动率可以处于两种状态之一——高或低。

跨状态的转移概率如下所示

```{image} /_static/lecture_specific/sci_libs/nfs_ex1.png
:align: center
```

例如，设周期长度为一天，假设当前状态为高。

从图中我们可以看出，明天的状态将是：

* 以 0.8 的概率为高
* 以 0.2 的概率为低

您的任务是根据此规则模拟每日波动率状态序列。

将序列长度设为 `n = 1_000_000`，并从高状态开始。

实现一个纯 Python 版本和一个 Numba 版本，并比较速度。

为了测试您的代码，评估链停留在低状态的时间比例。

如果您的代码正确，该比例应约为 2/3。

```{hint}
:class: dropdown

* 将低状态表示为 0，高状态表示为 1。
* 如果您想在 NumPy 数组中存储整数，然后应用 JIT 编译，请使用 `x = np.empty(n, dtype=np.int64)`。

```

```{exercise-end}
```

```{solution-start} speed_ex2
:class: dropdown
```

我们设

- 0 表示"低"
- 1 表示"高"

```{code-cell} ipython3
p, q = 0.1, 0.2  # 分别为离开低状态和高状态的概率
```

以下是函数的纯 Python 版本

```{code-cell} ipython3
def compute_series(n):
    x = np.empty(n, dtype=np.int64)
    x[0] = 1  # 从状态 1 开始
    U = np.random.uniform(0, 1, size=n)
    for t in range(1, n):
        current_x = x[t-1]
        if current_x == 0:
            x[t] = U[t] < p
        else:
            x[t] = U[t] > q
    return x
```

让我们运行这段代码，并检查处于低状态的时间比例约为 0.666

```{code-cell} ipython3
n = 1_000_000
x = compute_series(n)
print(np.mean(x == 0))  # x 处于状态 0 的时间比例
```

这是（近似）正确的输出。

现在让我们计时：

```{code-cell} ipython3
with qe.Timer():
    compute_series(n)
```

接下来，让我们实现一个 Numba 版本，这很容易

```{code-cell} ipython3
compute_series_numba = jit(compute_series)
```

让我们检查是否仍然得到正确的数字

```{code-cell} ipython3
x = compute_series_numba(n)
print(np.mean(x == 0))
```

让我们看看时间

```{code-cell} ipython3
with qe.Timer():
    compute_series_numba(n)
```

对于一行代码来说，这是一个不错的速度提升！

```{solution-end}
```

```{exercise}
:label: numba_ex3

在 {ref}`之前的练习 <speed_ex1>` 中，我们使用 Numba 加速了通过蒙特卡洛方法计算常数 $\pi$ 的工作。

现在尝试添加并行化，看看是否能获得进一步的速度提升。

这里您不应该期望获得巨大的提升，因为虽然有许多独立的任务（抽取点并测试是否在圆内），但每个任务的执行时间都很短。

一般来说，当要并行化的各个任务相对于总执行时间非常小时，并行化效果较差。

这是由于将所有这些小任务分散到多个 CPU 上所带来的开销。

尽管如此，使用合适的硬件，在本练习中仍然可以获得不可忽视的速度提升。

对于蒙特卡洛模拟的规模，请使用一个较大的值，例如 `n = 100_000_000`。
```

```{solution-start} numba_ex3
:class: dropdown
```

以下是一种解法：

```{code-cell} ipython3
@jit(parallel=True)
def calculate_pi(n=1_000_000):
    count = 0
    for i in prange(n):
        u, v = np.random.uniform(0, 1), np.random.uniform(0, 1)
        d = np.sqrt((u - 0.5)**2 + (v - 0.5)**2)
        if d < 0.5:
            count += 1

    area_estimate = count / n
    return area_estimate * 4  # 除以半径的平方
```

现在让我们看看运行速度：

```{code-cell} ipython3
with qe.Timer():
    calculate_pi()
```

```{code-cell} ipython3
with qe.Timer():
    calculate_pi()
```

通过打开和关闭并行化（在 `@jit` 注解中选择 `True` 或 `False`），我们可以测试多线程在 JIT 编译之上提供的速度增益。

在我们的工作站上，我们发现并行化将执行速度提高了 2 到 3 倍。

（如果您在本地执行，您将得到不同的数字，主要取决于您机器上的 CPU 数量。）

```{solution-end}
```


```{exercise}
:label: numba_ex4

在 {doc}`我们关于 SciPy 的讲座 <scipy>` 中，我们讨论了在标的股票价格具有简单且众所周知的分布的情况下，如何为看涨期权定价。

这里我们讨论一个更现实的情境。

我们回顾一下，期权的价格满足

$$
P = \beta^n \mathbb E \max\{ S_n - K, 0 \}
$$

其中

1. $\beta$ 是贴现因子，
2. $n$ 是到期日，
2. $K$ 是行权价，以及
3. $\{S_t\}$ 是标的资产在每个时刻 $t$ 的价格。

假设 `n, β, K = 20, 0.99, 100`。

假设股票价格满足

$$
\ln \frac{S_{t+1}}{S_t} = \mu + \sigma_t \xi_{t+1}
$$

其中

$$
    \sigma_t = \exp(h_t),
    \quad
        h_{t+1} = \rho h_t + \nu \eta_{t+1}
$$

这里 $\{\xi_t\}$ 和 $\{\eta_t\}$ 是独立同分布的标准正态随机变量。

（这是一个**随机波动率**模型，其中波动率 $\sigma_t$ 随时间变化。）

使用默认值 `μ, ρ, ν, S0, h0 = 0.0001, 0.1, 0.001, 10, 0`。

（这里 `S0` 是 $S_0$，`h0` 是 $h_0$。）

通过生成 $M$ 条路径 $s_0, \ldots, s_n$，计算蒙特卡洛估计值

$$
    \hat P_M
    := \beta^n \mathbb E \max\{ S_n - K, 0 \}
    \approx
    \frac{1}{M} \sum_{m=1}^M \max \{S_n^m - K, 0 \}
$$

即价格，应用 Numba 和并行化。

```


```{solution-start} numba_ex4
:class: dropdown
```


令 $s_t := \ln S_t$，价格动态变为

$$
s_{t+1} = s_t + \mu + \exp(h_t) \xi_{t+1}
$$

利用这一事实，解可以写成如下形式。


```{code-cell} ipython3
M = 10_000_000

n, β, K = 20, 0.99, 100
μ, ρ, ν, S0, h0 = 0.0001, 0.1, 0.001, 10, 0

@jit(parallel=True)
def compute_call_price_parallel(β=β,
                                μ=μ,
                                S0=S0,
                                h0=h0,
                                K=K,
                                n=n,
                                ρ=ρ,
                                ν=ν,
                                M=M):
    current_sum = 0.0
    # 对每条样本路径
    for m in prange(M):
        s = np.log(S0)
        h = h0
        # 向前模拟
        for t in range(n):
            s = s + μ + np.exp(h) * np.random.randn()
            h = ρ * h + ν * np.random.randn()
        # 将 max{S_n - K, 0} 的值累加到 current_sum
        current_sum += max(np.exp(s) - K, 0)

    return β**n * current_sum / M
```

尝试在 `parallel=True` 和 `parallel=False` 之间切换，并注意运行时间的差异。

如果您使用的是具有多个 CPU 的机器，差异应该很显著。

```{solution-end}
```
