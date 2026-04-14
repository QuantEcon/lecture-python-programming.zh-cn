---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.2
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
translation:
  title: JAX
  headings:
    JAX as a NumPy Replacement: JAX 作为 NumPy 的替代品
    JAX as a NumPy Replacement::Similarities: 相似之处
    JAX as a NumPy Replacement::Differences: 差异
    JAX as a NumPy Replacement::Differences::Speed!: 速度！
    JAX as a NumPy Replacement::Differences::Speed!::With NumPy: 使用 NumPy
    JAX as a NumPy Replacement::Differences::Speed!::With JAX: 使用 JAX
    JAX as a NumPy Replacement::Differences::Size Experiment: 大小实验
    JAX as a NumPy Replacement::Differences::Precision: 精度
    JAX as a NumPy Replacement::Differences::Immutability: 不可变性
    JAX as a NumPy Replacement::Differences::A Workaround: 变通方法
    Functional Programming: 函数式编程
    Functional Programming::Pure functions: 纯函数
    Functional Programming::Examples -- Pure and Impure: 示例——纯函数与非纯函数
    Functional Programming::Why Functional Programming?: 为什么要函数式编程？
    Random numbers: 随机数
    Random numbers::NumPy / MATLAB Approach: NumPy / MATLAB 方法
    Random numbers::JAX: JAX
    Random numbers::Benefits: 优势
    JIT Compilation: JIT 编译
    JIT Compilation::With NumPy: 使用 NumPy
    JIT Compilation::With JAX: 使用 JAX
    JIT Compilation::Compiling the Whole Function: 编译整个函数
    JIT Compilation::How JIT compilation works: JIT 编译的工作原理
    JIT Compilation::Compiling non-pure functions: 编译非纯函数
    Exercises: 练习
---

# JAX

本讲座简要介绍 [Google JAX](https://github.com/jax-ml/jax)。

```{include} _admonition/gpu.md
```

JAX 是一个高性能科学计算库，提供以下功能：

* 类似 [NumPy](https://en.wikipedia.org/wiki/NumPy) 的接口，可以在 CPU 和 GPU 上自动并行化，
* 一个即时编译器，用于加速大量数值运算，以及
* [自动微分](https://en.wikipedia.org/wiki/Automatic_differentiation)。

JAX 也在日益维护和提供 [更多专业化的科学计算例程](https://docs.jax.dev/en/latest/jax.scipy.html)，例如那些最初在 [SciPy](https://en.wikipedia.org/wiki/SciPy) 中找到的例程。

除了 Anaconda 中已有的内容外，本讲座还需要以下库：

```{code-cell} ipython3
:tags: [hide-output]

!pip install jax quantecon
```

我们将使用以下导入：

```{code-cell} ipython3
import jax
import jax.numpy as jnp
import matplotlib.pyplot as plt
import numpy as np
import quantecon as qe
```

注意我们导入了 `jax.numpy as jnp`，它提供了类似 NumPy 的接口。

## JAX 作为 NumPy 的替代品

JAX 的一个吸引人之处在于，它的数组处理操作在尽可能的情况下遵循 NumPy API。

这意味着在许多情况下，我们可以将 JAX 作为 NumPy 的直接替代品使用。

让我们来看看 JAX 和 NumPy 之间的异同。

### 相似之处

以下是使用 `jnp` 进行的一些标准数组操作：

```{code-cell} ipython3
a = jnp.asarray((1.0, 3.2, -1.5))
```

```{code-cell} ipython3
print(a)
```

```{code-cell} ipython3
print(jnp.sum(a))
```

```{code-cell} ipython3
print(jnp.dot(a, a))
```

然而，数组对象 `a` 并不是 NumPy 数组：

```{code-cell} ipython3
a
```

```{code-cell} ipython3
type(a)
```

即使是数组上的标量值映射也会返回 JAX 数组，而不是标量！

```{code-cell} ipython3
jnp.sum(a)
```

### 差异

现在让我们来看看 JAX 和 NumPy 数组操作之间的一些差异。

(jax_speed)=
#### 速度！

假设我们想在许多点上计算余弦函数。

```{code-cell}
n = 50_000_000
x = np.linspace(0, 10, n)
```

##### 使用 NumPy

让我们先用 NumPy 试试：

```{code-cell}
with qe.Timer():
    # First NumPy timing
    y = np.cos(x)
```

再来一次。

```{code-cell}
with qe.Timer():
    # Second NumPy timing
    y = np.cos(x)
```

这里

* NumPy 使用预编译的二进制文件对浮点数数组应用余弦函数
* 该二进制文件在本地机器的 CPU 上运行

##### 使用 JAX

现在让我们用 JAX 试试。

```{code-cell}
x = jnp.linspace(0, 10, n)
```

让我们对相同的过程计时。

```{code-cell}
with qe.Timer():
    # First run
    y = jnp.cos(x)
    # Hold the interpreter until the array operation finishes
    jax.block_until_ready(y);
```

```{note}
这里，为了测量实际速度，我们使用 `block_until_ready` 方法来阻塞解释器，直到计算结果返回。

这是必要的，因为 JAX 使用异步调度，允许 Python 解释器在数值计算之前运行。

对于非计时代码，可以删除包含 `block_until_ready` 的那一行。
```

再来计时一次。

```{code-cell}
with qe.Timer():
    # Second run
    y = jnp.cos(x)
    # Hold interpreter 
    jax.block_until_ready(y);
```

在 GPU 上，此代码的运行速度远快于其 NumPy 等效代码。

此外，通常第二次运行比第一次更快，这是由于 JIT 编译的缘故。

这是因为即使是像 `jnp.cos` 这样的内置函数也是经过 JIT 编译的——第一次运行包含了编译时间。

为什么 JAX 要对像 `jnp.cos` 这样的内置函数进行 JIT 编译，而不是像 NumPy 那样直接提供预编译版本？

原因是 JIT 编译器希望针对所使用数组的*大小*（以及数据类型）进行专门优化。

大小对于生成优化代码很重要，因为高效的并行化需要将任务大小与可用硬件相匹配。

#### 大小实验

我们可以通过更改输入大小并观察运行时间来验证 JAX 针对数组大小进行专门化的说法。

```{code-cell}
x = jnp.linspace(0, 10, n + 1)
```

```{code-cell}
with qe.Timer():
    # First run
    y = jnp.cos(x)
    # Hold interpreter
    jax.block_until_ready(y);
```

```{code-cell}
with qe.Timer():
    # Second run
    y = jnp.cos(x)
    # Hold interpreter
    jax.block_until_ready(y);
```

运行时间先增加后减少（这在 GPU 上会更明显）。

这与上面的讨论一致——更改数组大小后的第一次运行显示了编译开销。

关于 JIT 编译的进一步讨论见下文。

#### 精度

NumPy 和 JAX 之间的另一个差异是 JAX 默认使用 32 位浮点数。

这是因为 JAX 经常用于 GPU 计算，而大多数 GPU 计算使用 32 位浮点数。

使用 32 位浮点数可以在精度损失很小的情况下带来显著的速度提升。

然而，对于某些计算，精度至关重要。

在这些情况下，可以通过以下命令强制使用 64 位浮点数：

```{code-cell} ipython3
jax.config.update("jax_enable_x64", True)
```

让我们验证这是否有效：

```{code-cell} ipython3
jnp.ones(3)
```

#### 不可变性

作为 NumPy 的替代品，一个更显著的差异是数组被视为**不可变的**。

例如，在 NumPy 中我们可以这样写：

```{code-cell} ipython3
a = np.linspace(0, 1, 3)
a
```

然后在内存中修改数据：

```{code-cell} ipython3
a[0] = 1
a
```

在 JAX 中，这会失败！

```{code-cell} ipython3
a = jnp.linspace(0, 1, 3)
a
```

```{code-cell} ipython3
try:
    a[0] = 1
except Exception as e:
    print(e)
```

JAX 的设计者选择将数组设为不可变的，因为 JAX 使用函数式编程风格，我们将在下面讨论这一点。

#### 变通方法

我们注意到 JAX 确实提供了一种替代原地数组修改的方式，使用 [`at` 方法](https://docs.jax.dev/en/latest/_autosummary/jax.numpy.ndarray.at.html)。

```{code-cell} ipython3
a = jnp.linspace(0, 1, 3)
```

应用 `at[0].set(1)` 会返回一个新的 `a` 的副本，其中第一个元素被设置为 1：

```{code-cell} ipython3
a = a.at[0].set(1)
a
```

显然，使用 `at` 有一些缺点：

* 语法繁琐，且
* 每次更改单个值时，我们都希望避免在内存中创建新数组！

因此，在大多数情况下，我们尽量避免使用这种语法。

（尽管它在 JIT 编译的函数中实际上可以很高效——但现在先把这个放在一边。）

## 函数式编程

来自 JAX 的文档：

*当在意大利乡间漫步时，当地人会毫不犹豫地告诉你 JAX 有"una anima di pura programmazione funzionale"（纯函数式编程的灵魂）。*

换句话说，JAX 假设采用 [函数式编程](https://en.wikipedia.org/wiki/Functional_programming) 风格。

### 纯函数

最主要的含义是 JAX 函数应该是纯函数。

[纯函数](https://en.wikipedia.org/wiki/Pure_function)具有以下特征：

1. *确定性*
2. *无副作用*

[确定性](https://en.wikipedia.org/wiki/Deterministic_algorithm)意味着：

* 相同输入 $\implies$ 相同输出
* 输出不依赖于全局状态

特别地，纯函数在使用相同输入调用时将始终返回相同的结果。

[无副作用](https://en.wikipedia.org/wiki/Side_effect_(computer_science))意味着函数：

* 不会改变全局状态
* 不会修改传递给函数的数据（不可变数据）

### 示例——纯函数与非纯函数

以下是一个*非纯*函数的示例：

```{code-cell} ipython3
tax_rate = 0.1

def add_tax(prices):
    for i, price in enumerate(prices):
        prices[i] = price * (1 + tax_rate)

prices = [10.0, 20.0]
add_tax(prices)
prices
```

这个函数不是纯函数，因为：

* 副作用——它修改了全局变量 `prices`
* 非确定性——对全局变量 `tax_rate` 的更改会修改函数输出，即使使用相同的输入数组 `prices`。

以下是一个*纯*版本：

```{code-cell} ipython3

def add_tax_pure(prices, tax_rate):
    new_prices = [price * (1 + tax_rate) for price in prices]
    return new_prices

tax_rate = 0.1
prices = (10.0, 20.0)
after_tax_prices = add_tax_pure(prices, tax_rate)
after_tax_prices
```

这是纯函数，因为：

* 所有依赖关系通过函数参数显式表达
* 不修改任何外部状态

### 为什么要函数式编程？

在 QuantEcon，我们热爱纯函数，因为它们：

* 有助于测试：每个函数可以独立运行
* 促进确定性行为，从而提高可重复性
* 防止由于修改共享状态而产生的错误

JAX 编译器热爱纯函数和函数式编程，因为：

* 数据依赖关系是显式的，有助于优化复杂计算
* 纯函数更易于微分（自动微分）
* 纯函数更易于并行化和优化（不依赖于共享的可变状态）

另一种理解方式如下：

JAX 将函数表示为计算图，然后对其进行编译或变换（例如，微分）。

这些计算图描述了给定的一组输入如何被转换为输出。

JAX 的计算图在构造上是纯粹的。

JAX 使用函数式编程风格，以便用户构建的函数能够直接映射到 JAX 所支持的图论表示中。

## 随机数

JAX 中的随机数生成与 NumPy 或 MATLAB 中的模式有很大不同。

### NumPy / MATLAB 方法

在 NumPy / MATLAB 中，生成通过维护隐藏的全局状态来工作。

```{code-cell} ipython3
np.random.seed(42)
print(np.random.randn(2))   
```

每次我们调用随机函数时，隐藏状态都会被更新：

```{code-cell} ipython3
print(np.random.randn(2)) 
```

这个函数*不是纯函数*，因为：

* 它是非确定性的：相同的输入，不同的输出
* 它有副作用：它修改了全局随机数生成器状态

这在并行化下是危险的——必须仔细控制每个线程中发生的事情。

### JAX

在 JAX 中，随机数生成器的状态被显式控制。

首先我们生成一个密钥，它为随机数生成器提供种子。

```{code-cell} ipython3
seed = 1234
key = jax.random.key(seed)
```

现在我们可以使用密钥生成一些随机数：

```{code-cell} ipython3
x = jax.random.normal(key, (3, 3))
x
```

如果我们再次使用相同的密钥，我们会以相同的种子初始化，因此随机数是相同的：

```{code-cell} ipython3
jax.random.normal(key, (3, 3))
```

要生成（准）独立的抽取，一种选择是"分裂"现有密钥：

```{code-cell} ipython3
key, subkey = jax.random.split(key)
```

```{code-cell} ipython3
jax.random.normal(key, (3, 3))
```

```{code-cell} ipython3
jax.random.normal(subkey, (3, 3))
```

下图说明了 `split` 如何从单个根密钥生成密钥树，每个密钥生成独立的随机抽取。

```{code-cell} ipython3
:tags: [hide-input]

fig, ax = plt.subplots(figsize=(8, 4))
ax.set_xlim(-0.5, 6.5)
ax.set_ylim(-0.5, 3.5)
ax.set_aspect('equal')
ax.axis('off')

box_style = dict(boxstyle="round,pad=0.3", facecolor="white",
                 edgecolor="black", linewidth=1.5)
box_used = dict(boxstyle="round,pad=0.3", facecolor="#d4edda",
                edgecolor="black", linewidth=1.5)

# Root key
ax.text(3, 3, "key₀", ha='center', va='center', fontsize=11,
        bbox=box_style)

# Level 1
ax.annotate("", xy=(1.5, 2), xytext=(3, 2.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.annotate("", xy=(4.5, 2), xytext=(3, 2.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.text(1.5, 2, "key₁", ha='center', va='center', fontsize=11,
        bbox=box_style)
ax.text(4.5, 2, "subkey₁", ha='center', va='center', fontsize=11,
        bbox=box_used)
ax.text(5.7, 2, "→ draw", ha='left', va='center', fontsize=10,
        color='green')

# Label the split
ax.text(2, 2.65, "split", ha='center', va='center', fontsize=9,
        fontstyle='italic', color='gray')

# Level 2
ax.annotate("", xy=(0.5, 1), xytext=(1.5, 1.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.annotate("", xy=(2.5, 1), xytext=(1.5, 1.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.text(0.5, 1, "key₂", ha='center', va='center', fontsize=11,
        bbox=box_style)
ax.text(2.5, 1, "subkey₂", ha='center', va='center', fontsize=11,
        bbox=box_used)
ax.text(3.7, 1, "→ draw", ha='left', va='center', fontsize=10,
        color='green')

ax.text(0.7, 1.65, "split", ha='center', va='center', fontsize=9,
        fontstyle='italic', color='gray')

# Level 3
ax.annotate("", xy=(0, 0), xytext=(0.5, 0.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.annotate("", xy=(1.5, 0), xytext=(0.5, 0.7),
            arrowprops=dict(arrowstyle="->", lw=1.5))
ax.text(0, 0, "key₃", ha='center', va='center', fontsize=11,
        bbox=box_style)
ax.text(1.5, 0, "subkey₃", ha='center', va='center', fontsize=11,
        bbox=box_used)
ax.text(2.7, 0, "→ draw", ha='left', va='center', fontsize=10,
        color='green')
ax.text(0, 0.65, "split", ha='center', va='center', fontsize=9,
        fontstyle='italic', color='gray')

ax.text(3, -0.5, "⋮", ha='center', va='center', fontsize=14)

ax.set_title("PRNG 密钥拆分树", fontsize=13, pad=10)
plt.tight_layout()
plt.show()
```

对于 NumPy 或 Matlab 用户来说，这种语法看起来很不寻常——但当我们进入并行编程时，就会很有意义。

下面的函数使用 `split` 生成 `k` 个（准）独立的随机 `n x n` 矩阵。

```{code-cell} ipython3
def gen_random_matrices(
        key,   # JAX key for random numbers
        n=2,   # Matrices will be n x n
        k=3    # Number of matrices to generate
    ):
    matrices = []
    for _ in range(k):
        key, subkey = jax.random.split(key)
        A = jax.random.uniform(subkey, (n, n))
        matrices.append(A)
    return matrices
```

```{code-cell} ipython3
seed = 42
key = jax.random.key(seed)
gen_random_matrices(key)
```

这个函数是*纯函数*

* 确定性：相同的输入，相同的输出
* 无副作用：没有隐藏状态被修改

### 优势

如上所述，这种显式性是有价值的：

* 可复现性：通过重用密钥轻松重现结果
* 并行化：控制每个独立线程中发生的事情
* 调试：没有隐藏状态使代码更容易测试
* JIT 兼容性：编译器可以更积极地优化纯函数

## JIT 编译

JAX 的即时（JIT）编译器通过生成随任务大小和硬件变化的高效机器码来加速执行。

我们在 {ref}`上文 <jax_speed>` 中已经看到了 JAX 的 JIT 编译器结合并行硬件的强大之处，当时我们对一个大数组应用了 `cos` 函数。

这里我们研究针对更复杂函数的 JIT 编译。

### 使用 NumPy

我们先用 NumPy 试试：

```{code-cell}
def f(x):
    y = np.cos(2 * x**2) + np.sqrt(np.abs(x)) + 2 * np.sin(x**4) - x**2
    return y
```

让我们用较大的 `x` 运行：

```{code-cell}
n = 50_000_000
x = np.linspace(0, 10, n)
```

```{code-cell}
with qe.Timer():
    # Time NumPy code
    y = f(x)
```

**即时执行**模型

* 每个操作在遇到时立即执行，在下一个操作开始之前将其结果实体化。

缺点

* 并行化程度极低
* 内存占用大——产生许多中间数组
* 大量内存读写

### 使用 JAX

作为第一步，我们将整个代码中的 `np` 替换为 `jnp`：

```{code-cell}
def f(x):
    y = jnp.cos(2 * x**2) + jnp.sqrt(jnp.abs(x)) + 2 * jnp.sin(x**4) - x**2
    return y


x = jnp.linspace(0, 10, n)
```

现在让我们计时。

```{code-cell}
with qe.Timer():
    # First call
    y = f(x)
    # Hold interpreter
    jax.block_until_ready(y);
```

```{code-cell}
with qe.Timer():
    # Second call
    y = f(x)
    # Hold interpreter
    jax.block_until_ready(y);
```

结果与 `cos` 示例类似——JAX 更快，尤其是在 JIT 编译后的第二次运行中。

这是因为单个数组操作在 GPU 上并行化了。

但我们仍然在使用即时执行模式

* 由于中间数组导致大量内存占用
* 大量内存读写

此外，在 GPU 上还启动了许多独立的内核。

### 编译整个函数

幸运的是，使用 JAX，我们还有另一个技巧——我们可以对整个函数进行 JIT 编译，而不仅仅是单个操作。

编译器将所有数组操作融合到单个优化内核中。

让我们用函数 `f` 来试试这个：

```{code-cell}
f_jax = jax.jit(f)
```

```{code-cell}
with qe.Timer():
    # First run
    y = f_jax(x)
    # Hold interpreter
    jax.block_until_ready(y);
```

```{code-cell}
with qe.Timer():
    # Second run
    y = f_jax(x)
    # Hold interpreter
    jax.block_until_ready(y);
```

运行时间再次改善——现在是因为我们融合了所有操作

* 基于整个计算序列的激进优化
* 消除对硬件加速器的多次调用

内存占用也大幅降低——不再创建中间数组。

顺便提一下，当针对 JIT 编译器的函数时，更常见的语法是：

```{code-cell} ipython3
@jax.jit
def f(x):
    pass # put function body here
```

### JIT 编译的工作原理

当我们对一个函数应用 `jax.jit` 时，JAX 会对其进行*追踪*：它不会立即执行操作，而是将操作序列记录为计算图，并将该图交给 [XLA](https://openxla.org/xla) 编译器。

XLA 随后将这些操作融合并优化为针对可用硬件（CPU、GPU 或 TPU）定制的单个编译内核。

对 JIT 编译函数的第一次调用会产生编译开销，但对于具有相同输入形状和类型的后续调用，将重用缓存的编译代码并以全速运行。

### 编译非纯函数

虽然 JAX 在编译非纯函数时通常不会抛出错误，但执行会变得不可预测！

以下是一个示例：

```{code-cell} ipython3
a = 1  # global

@jax.jit
def f(x):
    return a + x
```

```{code-cell} ipython3
x = jnp.ones(2)
```

```{code-cell} ipython3
f(x)
```

在上面的代码中，全局值 `a=1` 被融入了 JIT 编译的函数中。

即使我们更改 `a`，只要调用的是相同的编译版本，`f` 的输出也不会受到影响。

```{code-cell} ipython3
a = 42
```

```{code-cell} ipython3
f(x)
```

更改输入的维度会触发函数的重新编译，此时 `a` 值的变化才会生效：

```{code-cell} ipython3
x = jnp.ones(3)
```

```{code-cell} ipython3
f(x)
```

这个故事的寓意：使用 JAX 时请编写纯函数！

## 练习


```{exercise-start}
:label: jax_intro_ex2
```

在关于 Numba 的{doc}`讲座 <numba>`的练习部分，我们{ref}`使用蒙特卡洛方法为欧式看涨期权定价 <numba_ex4>`。

该代码通过基于 Numba 的多线程进行了加速。

尝试使用所有相同的参数为 JAX 编写此操作的版本。



```{exercise-end}
```


```{solution-start} jax_intro_ex2
:class: dropdown
```
以下是一种解法：

```{code-cell} ipython3
M = 10_000_000

n, β, K = 20, 0.99, 100
μ, ρ, ν, S0, h0 = 0.0001, 0.1, 0.001, 10, 0

@jax.jit
def compute_call_price_jax(β=β,
                           μ=μ,
                           S0=S0,
                           h0=h0,
                           K=K,
                           n=n,
                           ρ=ρ,
                           ν=ν,
                           M=M,
                           key=jax.random.key(1)):

    s = jnp.full(M, np.log(S0))
    h = jnp.full(M, h0)

    def update(i, loop_state):
        s, h, key = loop_state
        key, subkey = jax.random.split(key)
        Z = jax.random.normal(subkey, (2, M))
        s = s + μ + jnp.exp(h) * Z[0, :]
        h = ρ * h + ν * Z[1, :]
        new_loop_state = s, h, key
        return new_loop_state

    initial_loop_state = s, h, key
    final_loop_state = jax.lax.fori_loop(0, n, update, initial_loop_state)
    s, h, key = final_loop_state

    expectation = jnp.mean(jnp.maximum(jnp.exp(s) - K, 0))

    return β**n * expectation
```

```{note}
我们使用 `jax.lax.fori_loop` 代替 Python 的 `for` 循环。
这允许 JAX 在不展开循环的情况下高效地编译循环，
从而显著减少大数组的编译时间。
```

让我们运行一次以编译它：

```{code-cell} ipython3
with qe.Timer():
    compute_call_price_jax().block_until_ready()
```

现在让我们计时：

```{code-cell} ipython3
with qe.Timer():
    compute_call_price_jax().block_until_ready()
```

```{solution-end}
```
