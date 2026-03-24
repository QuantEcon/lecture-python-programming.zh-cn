---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
heading-map:
  More Language Features: 更多语言特性
  Overview: 概述
  Iterables and iterators: 可迭代对象与迭代器
  Iterables and iterators::Iterators: 迭代器
  Iterables and iterators::Iterators in for loops: for 循环中的迭代器
  Iterables and iterators::Iterables: 可迭代对象
  Iterables and iterators::Iterators and built-ins: 迭代器与内置函数
  '`*` and `**` operators': '`*` 和 `**` 运算符'
  '`*` and `**` operators::Unpacking arguments': 解包参数
  '`*` and `**` operators::Arbitrary arguments': 任意数量的参数
  Type hints: 类型提示
  Type hints::Basic syntax: 基本语法
  Type hints::Common types: 常用类型
  Type hints::Hints don't enforce types: 提示不强制类型
  Type hints::Why use type hints?: 为什么使用类型提示？
  Type hints::Type hints in scientific Python: 科学 Python 中的类型提示
  Decorators and descriptors: 装饰器与描述符
  Decorators and descriptors::Decorators: 装饰器
  Decorators and descriptors::Decorators::An example: 一个示例
  Decorators and descriptors::Decorators::Enter decorators: 引入装饰器
  Decorators and descriptors::Descriptors: 描述符
  Decorators and descriptors::Descriptors::A solution: 解决方案
  Decorators and descriptors::Descriptors::How it works: 工作原理
  Decorators and descriptors::Descriptors::Decorators and properties: 装饰器与属性
  Generators: 生成器
  Generators::Generator expressions: 生成器表达式
  Generators::Generator functions: 生成器函数
  Generators::Generator functions::Example 1: 示例 1
  Generators::Generator functions::Example 2: 示例 2
  Generators::Advantages of iterators: 迭代器的优势
  Exercises: 练习
---

(python_advanced_features)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# 更多语言特性

## 概述

关于这最后一讲，我们的建议是*第一次阅读时跳过它*，除非你迫切想要阅读。

它在这里有两个用途：

1. 作为参考，以便我们在需要时可以链接回来，以及
1. 供那些已经完成了许多应用练习、现在想要深入了解 Python 语言的人使用。

本讲涵盖了多种主题，包括迭代器、类型提示、装饰器和描述符，以及生成器。

## 可迭代对象与迭代器

```{index} single: Python; Iteration
```

我们{ref}`已经提到过 <iterating_version_1>` Python 中的迭代。

现在让我们更仔细地了解它的工作原理，重点关注 Python 中 `for` 循环的实现。

(iterators)=

### 迭代器

```{index} single: Python; Iterators
```

迭代器是一种统一的接口，用于逐步遍历集合中的元素。

这里我们将讨论如何使用迭代器——稍后我们将学习如何构建自己的迭代器。

正式地说，*迭代器*是一个具有 `__next__` 方法的对象。

例如，文件对象就是迭代器。

为了验证这一点，让我们再来看看{ref}`美国城市数据 <us_cities_data>`，
该数据在以下单元格中被写入当前工作目录：

```{code-cell} ipython
%%file us_cities.txt
new york: 8244910
los angeles: 3819702
chicago: 2707120
houston: 2145146
philadelphia: 1536471
phoenix: 1469471
san antonio: 1359758
san diego: 1326179
dallas: 1223229
```

```{code-cell} python3
f = open('us_cities.txt')
f.__next__()
```

```{code-cell} python3
f.__next__()
```

我们看到文件对象确实有一个 `__next__` 方法，调用该方法会返回文件中的下一行。

`next` 方法也可以通过内置函数 `next()` 访问，
该函数直接调用此方法：

```{code-cell} python3
next(f)
```

`enumerate()` 返回的对象也是迭代器：

```{code-cell} python3
e = enumerate(['foo', 'bar'])
next(e)
```

```{code-cell} python3
next(e)
```

`csv` 模块中的 reader 对象也是迭代器。

让我们创建一个小的 csv 文件，其中包含日经指数的数据：

```{code-cell} ipython
%%file test_table.csv
Date,Open,High,Low,Close,Volume,Adj Close
2009-05-21,9280.35,9286.35,9189.92,9264.15,133200,9264.15
2009-05-20,9372.72,9399.40,9311.61,9344.64,143200,9344.64
2009-05-19,9172.56,9326.75,9166.97,9290.29,167000,9290.29
2009-05-18,9167.05,9167.82,8997.74,9038.69,147800,9038.69
2009-05-15,9150.21,9272.08,9140.90,9265.02,172000,9265.02
2009-05-14,9212.30,9223.77,9052.41,9093.73,169400,9093.73
2009-05-13,9305.79,9379.47,9278.89,9340.49,176000,9340.49
2009-05-12,9358.25,9389.61,9298.61,9298.61,188400,9298.61
2009-05-11,9460.72,9503.91,9342.75,9451.98,230800,9451.98
2009-05-08,9351.40,9464.43,9349.57,9432.83,220200,9432.83
```

```{code-cell} python3
from csv import reader

f = open('test_table.csv', 'r')
nikkei_data = reader(f)
next(nikkei_data)
```

```{code-cell} python3
next(nikkei_data)
```

### for 循环中的迭代器

```{index} single: Python; Iterators
```

所有迭代器都可以放在 `for` 循环语句中 `in` 关键字的右侧。

实际上，这正是 `for` 循环的工作方式：如果我们写

```{code-block} python3
:class: no-execute

for x in iterator:
    <code block>
```

那么解释器会

* 调用 `iterator.___next___()` 并将 `x` 绑定到结果
* 执行代码块
* 重复直到发生 `StopIteration` 错误

所以现在你明白了这种看起来很神奇的语法是如何工作的：

```{code-block} python3
:class: no-execute

f = open('somefile.txt', 'r')
for line in f:
    # do something
```

解释器只是不断地

1. 调用 `f.__next__()` 并将 `line` 绑定到结果
1. 执行循环体

这一过程持续到发生 `StopIteration` 错误为止。

### 可迭代对象

```{index} single: Python; Iterables
```

你已经知道我们可以将 Python 列表放在 `for` 循环中 `in` 的右侧：

```{code-cell} python3
for i in ['spam', 'eggs']:
    print(i)
```

那么这是否意味着列表就是迭代器呢？

答案是否定的：

```{code-cell} python3
x = ['foo', 'bar']
type(x)
```

```{code-cell} python3
---
tags: [raises-exception]
---
next(x)
```

那么为什么我们可以在 `for` 循环中遍历列表呢？

原因是列表是*可迭代的*（而不是迭代器）。

正式地说，如果一个对象可以使用内置函数 `iter()` 转换为迭代器，则该对象是可迭代的。

列表就是这样一种对象：

```{code-cell} python3
x = ['foo', 'bar']
type(x)
```

```{code-cell} python3
y = iter(x)
type(y)
```

```{code-cell} python3
next(y)
```

```{code-cell} python3
next(y)
```

```{code-cell} python3
---
tags: [raises-exception]
---
next(y)
```

许多其他对象也是可迭代的，例如字典和元组。

当然，并非所有对象都是可迭代的：

```{code-cell} python3
---
tags: [raises-exception]
---
iter(42)
```

总结我们对 `for` 循环的讨论：

* `for` 循环适用于迭代器或可迭代对象。
* 在第二种情况下，可迭代对象在循环开始之前被转换为迭代器。

### 迭代器与内置函数

```{index} single: Python; Iterators
```

一些作用于序列的内置函数也适用于可迭代对象：

* `max()`、`min()`、`sum()`、`all()`、`any()`

例如：

```{code-cell} python3
x = [10, -10]
max(x)
```

```{code-cell} python3
y = iter(x)
type(y)
```

```{code-cell} python3
max(y)
```

关于迭代器，有一点需要记住，那就是它们会因使用而耗尽：

```{code-cell} python3
x = [10, -10]
y = iter(x)
max(y)
```

```{code-cell} python3
---
tags: [raises-exception]
---
max(y)
```

## `*` 和 `**` 运算符

`*` 和 `**` 是方便且广泛使用的工具，用于解包列表和元组，以及允许用户定义可以接受任意数量参数的函数。

在本节中，我们将探讨如何使用它们并区分它们的使用场景。

### 解包参数

当我们操作一个参数列表时，我们通常需要在将参数传递给函数时，将列表的内容提取为单独的参数，而不是作为一个集合。

幸运的是，`*` 运算符可以帮助我们在函数调用中将列表和元组解包为[*位置参数*](pos_args)。

为了具体说明，请看以下示例：

不使用 `*` 时，`print` 函数打印一个列表：

```{code-cell} python3
l1 = ['a', 'b', 'c']

print(l1)
```

而使用 `*` 时，`print` 函数打印单个元素，因为 `*` 将列表解包为单独的参数：

```{code-cell} python3
print(*l1)
```

使用 `*` 将列表解包为位置参数，等同于在调用函数时单独定义它们：

```{code-cell} python3
print('a', 'b', 'c')
```

但是，如果我们想要再次重用它们，`*` 运算符会更加方便：

```{code-cell} python3
l1.append('d')

print(*l1)
```

类似地，`**` 也用于解包参数。

区别在于 `**` 将*字典*解包为*关键字参数*。

当我们有许多想要重用的关键字参数时，通常会使用 `**`。

例如，假设我们想使用相同的图形设置绘制多个图表，这可能涉及反复设置许多图形参数，这些参数通常使用关键字参数来定义。

在这种情况下，我们可以使用字典来存储这些参数，并在需要时使用 `**` 将字典解包为关键字参数。

让我们一起通过一个简单的示例，来区分 `*` 和 `**` 的用法：

```{code-cell} python3
import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n

# 设置图框和子图
fig, ax = plt.subplots(2, 1)
plt.subplots_adjust(hspace=0.7)

# 创建一个生成合成数据的函数
def generate_data(β_0, β_1, σ=30, n=100):
    x_values = np.arange(0, n, 1)
    y_values = β_0 + β_1 * x_values + np.random.normal(size=n, scale=σ)
    return x_values, y_values

# 将折线和图例的关键字参数存储在字典中
line_kargs = {'lw': 1.5, 'alpha': 0.7}
legend_kargs = {'bbox_to_anchor': (0., 1.02, 1., .102), 
                'loc': 3, 
                'ncol': 4,
                'mode': 'expand', 
                'prop': {'size': 7}}

β_0s = [10, 20, 30]
β_1s = [1, 2, 3]

# 使用 for 循环绘制折线
def generate_plots(β_0s, β_1s, idx, line_kargs, legend_kargs):
    label_list = []
    for βs in zip(β_0s, β_1s):
    
        # 使用 * 解包元组 βs 和 generate_data 函数的元组输出
        # 使用 ** 解包折线关键字参数字典
        ax[idx].plot(*generate_data(*βs), **line_kargs)

        label_list.append(f'$β_0 = {βs[0]}$ | $β_1 = {βs[1]}$')

    # 使用 ** 解包图例关键字参数字典
    ax[idx].legend(label_list, **legend_kargs)

generate_plots(β_0s, β_1s, 0, line_kargs, legend_kargs)

# 我们可以轻松地重用和更新参数
β_1s.append(-2)
β_0s.append(40)
line_kargs['lw'] = 2
line_kargs['alpha'] = 0.4

generate_plots(β_0s, β_1s, 1, line_kargs, legend_kargs)
plt.show()
```

在这个示例中，`*` 解包了存储在元组中的压缩参数 `βs` 和 `generate_data` 函数的输出，
而 `**` 解包了存储在 `legend_kargs` 和 `line_kargs` 中的图形参数。

总结一下，当 `*list`/`*tuple` 和 `**dictionary` 被传入*函数调用*时，它们被解包为单独的参数而不是集合。

区别在于 `*` 将列表和元组解包为*位置参数*，而 `**` 将字典解包为*关键字参数*。

### 任意数量的参数

当我们*定义*函数时，有时希望允许用户向函数传入任意数量的参数。

你可能已经注意到 `ax.plot()` 函数可以处理任意数量的参数。

如果我们查看该函数的[文档](https://github.com/matplotlib/matplotlib/blob/v3.6.2/lib/matplotlib/axes/_axes.py#L1417-L1669)，可以看到该函数定义为：

```
Axes.plot(*args, scalex=True, scaley=True, data=None, **kwargs)
```

我们在*函数定义*的上下文中再次看到了 `*` 和 `**` 运算符。

事实上，`*args` 和 `**kargs` 在 Python 的科学库中无处不在，用于减少冗余并允许灵活的输入。

`*args` 使函数能够处理大小可变的*位置参数*：

```{code-cell} python3
l1 = ['a', 'b', 'c']
l2 = ['b', 'c', 'd']

def arb(*ls):
    print(ls)

arb(l1, l2)
```

输入被传递到函数中并存储在一个元组中。

让我们尝试更多的输入：

```{code-cell} python3
l3 = ['z', 'x', 'b']
arb(l1, l2, l3)
```

类似地，Python 允许我们使用 `**kargs` 向函数传入任意数量的*关键字参数*：

```{code-cell} python3
def arb(**ls):
    print(ls)

# 注意这些是关键字参数
arb(l1=l1, l2=l2)
```

我们可以看到 Python 使用字典来存储这些关键字参数。

让我们尝试更多的输入：

```{code-cell} python3
arb(l1=l1, l2=l2, l3=l3)
```

总体而言，`*args` 和 `**kargs` 在*定义函数*时使用，它们使函数能够接受任意大小的输入。

区别在于带有 `*args` 的函数能够接受任意大小的*位置参数*，而 `**kargs` 允许函数接受任意数量的*关键字参数*。

## 类型提示

```{index} single: Python; Type Hints
```

Python 是一种*动态类型*语言，这意味着您无需声明变量的类型。

（请参阅我们{doc}`之前关于 <need_for_speed>`动态类型与静态类型的讨论。）

然而，Python 支持可选的**类型提示**（也称为类型注解），允许您指定变量、函数参数和返回值的预期类型。

类型提示从 Python 3.5 开始引入，并在后续版本中不断发展。此处展示的所有语法均适用于 Python 3.9 及更高版本。

```{note}
类型提示*在运行时会被 Python 解释器忽略*——它们不会影响代码的执行方式。它们纯粹是信息性的，作为供人类和工具使用的文档。
```

### 基本语法

类型提示使用冒号 `:` 来注解变量和参数，使用箭头 `->` 来注解返回类型。

下面是一个简单的示例：

```{code-cell} python3
def greet(name: str, times: int) -> str:
    return (name + '! ') * times

greet('hello', 3)
```

在这个函数定义中：

- `name: str` 表示 `name` 预期为字符串类型
- `times: int` 表示 `times` 预期为整数类型
- `-> str` 表示该函数返回一个字符串

您也可以直接注解变量：

```{code-cell} python3
x: int = 10
y: float = 3.14
name: str = 'Python'
```

### 常用类型

最常用的类型提示是内置类型：

| 类型      | 示例                          |
|-----------|----------------------------------|
| `int`     | `x: int = 5`                    |
| `float`   | `x: float = 3.14`              |
| `str`     | `x: str = 'hello'`             |
| `bool`    | `x: bool = True`               |
| `list`    | `x: list = [1, 2, 3]`          |
| `dict`    | `x: dict = {'a': 1}`           |

对于容器类型，您可以指定其元素的类型：

```{code-cell} python3
prices: list[float] = [9.99, 4.50, 2.89]
counts: dict[str, int] = {'apples': 3, 'oranges': 5}
```

### 提示不强制类型

对于 Python 新手来说，有一个重要的点需要注意：类型提示在运行时*不会被强制执行*。

如果您传入"错误"的类型，Python 不会抛出错误：

```{code-cell} python3
def add(x: int, y: int) -> int:
    return x + y

# Passes floats — Python doesn't complain
add(1.5, 2.7)
```

提示说明参数类型为 `int`，但 Python 欣然接受 `float` 参数并返回 `4.2`——同样不是 `int`。

这与 C 或 Java 等静态类型语言有着关键区别，在那些语言中，类型不匹配会导致编译错误。

### 为什么使用类型提示？

如果 Python 会忽略它们，为什么还要使用呢？

1. **可读性**：类型提示使函数签名具有自文档化的特性。读者可以立即了解函数期望的输入类型和返回类型。
2. **编辑器支持**：VS Code 等 IDE 利用类型提示提供更好的自动补全、错误检测和内联文档功能。
3. **错误检查**：[mypy](https://mypy.readthedocs.io/) 和 [pyrefly](https://pyrefly.org/) 等工具会分析类型提示，在您运行代码*之前*捕获错误。
4. **大语言模型生成的代码**：大语言模型经常生成带有类型提示的代码，因此理解相关语法有助于您阅读和使用其输出结果。

### 科学 Python 中的类型提示

类型提示与{doc}`性能需求 <need_for_speed>`的讨论密切相关：

* [JAX](https://jax.readthedocs.io/) 和 [Numba](https://numba.pydata.org/) 等高性能库依赖于了解变量类型来编译快速的机器码。
* 虽然这些库在运行时推断类型，而不是直接读取 Python 类型提示，但*概念*是相同的——显式的类型信息能够实现优化。
* 随着 Python 生态系统的发展，类型提示与性能工具之间的联系预计将进一步加强。

目前，类型提示在日常 Python 开发中的主要优势在于*清晰性和工具支持*，随着程序规模的增长，这一优势的价值也会越来越大。

## 装饰器与描述符

```{index} single: Python; Decorators
```

```{index} single: Python; Descriptors
```

让我们看看 Python 开发者常用的一些特殊语法元素。

你可能不会立即需要以下概念，但你会在其他人的代码中看到它们。

因此，在你的 Python 学习过程中，你需要在某个阶段理解它们。

### 装饰器

```{index} single: Python; Decorators
```

装饰器是一种语法糖，虽然很容易避免使用，但事实证明非常受欢迎。

说清楚装饰器的作用非常简单。

另一方面，解释*为什么*你可能会使用它们则需要花费一些努力。

#### 一个示例

假设我们正在开发一个如下所示的程序：

```{code-cell} python3
import numpy as np

def f(x):
    return np.log(np.log(x))

def g(x):
    return np.sqrt(42 * x)

# 程序继续使用 f 和 g 进行各种计算
```

现在假设有一个问题：偶尔会有负数被传入到后续计算中的 `f` 和 `g`。

如果你尝试一下，你会看到当这些函数以负数调用时，它们返回一个名为 `nan` 的 NumPy 对象。

这代表"非数字"（表明你正在尝试在未定义的点处求值一个数学函数）。

也许这不是我们想要的，因为它会导致其他难以在后期发现的问题。

假设我们希望程序在发生这种情况时终止，并给出合理的错误消息。

这个更改很容易实现：

```{code-cell} python3
import numpy as np

def f(x):
    assert x >= 0, "Argument must be nonnegative"
    return np.log(np.log(x))

def g(x):
    assert x >= 0, "Argument must be nonnegative"
    return np.sqrt(42 * x)

# 程序继续使用 f 和 g 进行各种计算
```

然而请注意，这里有一些重复，以两行相同代码的形式出现。

重复使我们的代码更长且更难维护，因此我们应该尽力避免。

在这里这不是什么大问题，但现在想象一下，不是只有 `f` 和 `g`，而是我们有 20 个需要以完全相同方式修改的函数。

这意味着我们需要重复测试逻辑（即测试非负性的 `assert` 行）20 次。

如果测试逻辑更长更复杂，情况就更糟了。

在这种情况下，以下方法会更为简洁：

```{code-cell} python3
import numpy as np

def check_nonneg(func):
    def safe_function(x):
        assert x >= 0, "Argument must be nonnegative"
        return func(x)
    return safe_function

def f(x):
    return np.log(np.log(x))

def g(x):
    return np.sqrt(42 * x)

f = check_nonneg(f)
g = check_nonneg(g)
# 程序继续使用 f 和 g 进行各种计算
```

这看起来很复杂，所以让我们慢慢理解它。

为了理清逻辑，考虑当我们说 `f = check_nonneg(f)` 时会发生什么。

这调用了函数 `check_nonneg`，参数 `func` 设置为等于 `f`。

现在 `check_nonneg` 创建了一个名为 `safe_function` 的新函数，
该函数验证 `x` 为非负数，然后对其调用 `func`（与 `f` 相同）。

最后，全局名称 `f` 被设置为等于 `safe_function`。

现在 `f` 的行为符合我们的期望，`g` 也是如此。

与此同时，测试逻辑只需编写一次。

#### 引入装饰器

```{index} single: Python; Decorators
```

我们代码的最后一个版本仍然不够理想。

例如，如果有人正在阅读我们的代码并想了解 `f` 的工作原理，
他们会寻找函数定义，即：

```{code-cell} python3
def f(x):
    return np.log(np.log(x))
```

他们很可能会错过 `f = check_nonneg(f)` 这一行。

出于这个和其他原因，Python 引入了装饰器。

使用装饰器，我们可以将以下代码：

```{code-cell} python3
def f(x):
    return np.log(np.log(x))

def g(x):
    return np.sqrt(42 * x)

f = check_nonneg(f)
g = check_nonneg(g)
```

替换为：

```{code-cell} python3
@check_nonneg
def f(x):
    return np.log(np.log(x))

@check_nonneg
def g(x):
    return np.sqrt(42 * x)
```

这两段代码做的事情完全相同。

如果它们做的事情相同，我们真的需要装饰器语法吗？

好吧，注意装饰器就位于函数定义的正上方。

因此，任何查看函数定义的人都会看到它们，并了解函数已被修改。

在许多人看来，这使得装饰器语法成为对语言的重大改进。

(descriptors)=

### 描述符

```{index} single: Python; Descriptors
```

描述符解决了变量管理方面的一个常见问题。

为了理解这个问题，考虑一个模拟汽车的 `Car` 类。

假设该类定义了变量 `miles` 和 `kms`，分别以英里和公里为单位给出行驶距离。

该类的一个高度简化的版本可能如下所示：

```{code-cell} python3
class Car:

    def __init__(self, miles=1000):
        self.miles = miles
        self.kms = miles * 1.61

    # 其他一些功能，细节省略
```

我们可能遇到的一个潜在问题是，用户更改了其中一个变量而没有更改另一个：

```{code-cell} python3
car = Car()
car.miles
```

```{code-cell} python3
car.kms
```

```{code-cell} python3
car.miles = 6000
car.kms
```

在最后两行中，我们看到 `miles` 和 `kms` 不同步了。

我们真正想要的是某种机制，使得每次用户设置其中一个变量时，*另一个会自动更新*。

#### 解决方案

在 Python 中，这个问题使用*描述符*来解决。

描述符只是一个实现了某些方法的 Python 对象。

这些方法在通过点属性符号访问对象时被触发。

理解这一点的最好方式是亲眼看看它的实际效果。

考虑这个 `Car` 类的替代版本：

```{code-cell} python3
class Car:

    def __init__(self, miles=1000):
        self._miles = miles
        self._kms = miles * 1.61

    def set_miles(self, value):
        self._miles = value
        self._kms = value * 1.61

    def set_kms(self, value):
        self._kms = value
        self._miles = value / 1.61

    def get_miles(self):
        return self._miles

    def get_kms(self):
        return self._kms

    miles = property(get_miles, set_miles)
    kms = property(get_kms, set_kms)
```

首先让我们检查一下是否获得了我们期望的行为：

```{code-cell} python3
car = Car()
car.miles
```

```{code-cell} python3
car.miles = 6000
car.kms
```

是的，这就是我们想要的——`car.kms` 会自动更新。

#### 工作原理

名称 `_miles` 和 `_kms` 是我们用来存储变量值的任意名称。

对象 `miles` 和 `kms` 是*属性*，一种常见的描述符。

方法 `get_miles`、`set_miles`、`get_kms` 和 `set_kms` 定义了
当你获取（即访问）或设置（绑定）这些变量时会发生什么：

* 即所谓的 "getter" 和 "setter" 方法。

Python 内置函数 `property` 接受 getter 和 setter 方法并创建一个属性。

例如，在 `car` 作为 `Car` 的实例创建之后，对象 `car.miles` 是一个属性。

作为属性，当我们通过 `car.miles = 6000` 设置其值时，它的 setter 方法会被触发——在本例中是 `set_miles`。

#### 装饰器与属性

```{index} single: Python; Decorators
```

```{index} single: Python; Properties
```

如今，通过装饰器使用 `property` 函数非常普遍。

这是我们 `Car` 类的另一个版本，与之前的工作方式相同，但现在使用装饰器来设置属性：

```{code-cell} python3
class Car:

    def __init__(self, miles=1000):
        self._miles = miles
        self._kms = miles * 1.61

    @property
    def miles(self):
        return self._miles

    @property
    def kms(self):
        return self._kms

    @miles.setter
    def miles(self, value):
        self._miles = value
        self._kms = value * 1.61

    @kms.setter
    def kms(self, value):
        self._kms = value
        self._miles = value / 1.61
```

我们不会在这里介绍所有细节。

如需进一步了解，可以参考[描述符文档](https://docs.python.org/3/howto/descriptor.html)。

(paf_generators)=

## 生成器

```{index} single: Python; Generators
```

生成器是一种迭代器（即它与 `next` 函数配合使用）。

我们将学习两种构建生成器的方式：生成器表达式和生成器函数。

### 生成器表达式

构建生成器最简单的方式是使用*生成器表达式*。

就像列表推导式，但使用圆括号。

这是列表推导式：

```{code-cell} python3
singular = ('dog', 'cat', 'bird')
type(singular)
```

```{code-cell} python3
plural = [string + 's' for string in singular]
plural
```

```{code-cell} python3
type(plural)
```

这是生成器表达式：

```{code-cell} python3
singular = ('dog', 'cat', 'bird')
plural = (string + 's' for string in singular)
type(plural)
```

```{code-cell} python3
next(plural)
```

```{code-cell} python3
next(plural)
```

```{code-cell} python3
next(plural)
```

由于 `sum()` 可以在迭代器上调用，我们可以这样做：

```{code-cell} python3
sum((x * x for x in range(10)))
```

函数 `sum()` 调用 `next()` 来获取元素，累加各项。

事实上，在这种情况下我们可以省略外层括号：

```{code-cell} python3
sum(x * x for x in range(10))
```

### 生成器函数

```{index} single: Python; Generator Functions
```

创建生成器对象最灵活的方式是使用生成器函数。

让我们看一些示例。

#### 示例 1

这是一个非常简单的生成器函数示例：

```{code-cell} python3
def f():
    yield 'start'
    yield 'middle'
    yield 'end'
```

它看起来像一个函数，但使用了我们以前没有见过的关键字 `yield`。

让我们在运行此代码后看看它是如何工作的：

```{code-cell} python3
type(f)
```

```{code-cell} python3
gen = f()
gen
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
---
tags: [raises-exception]
---
next(gen)
```

生成器函数 `f()` 用于创建生成器对象（在本例中是 `gen`）。

生成器是迭代器，因为它们支持 `next` 方法。

第一次调用 `next(gen)` 时：

* 执行 `f()` 函数体中的代码，直到遇到 `yield` 语句。
* 将该值返回给 `next(gen)` 的调用者。

第二次调用 `next(gen)` 时，从*下一行*开始执行：

```{code-cell} python3
def f():
    yield 'start'
    yield 'middle'  # 从这一行开始！
    yield 'end'
```

并继续执行到下一个 `yield` 语句。

此时，它将 `yield` 后面的值返回给 `next(gen)` 的调用者，依此类推。

当代码块结束时，生成器抛出 `StopIteration` 错误。

#### 示例 2

我们的下一个示例从调用者接收参数 `x`：

```{code-cell} python3
def g(x):
    while x < 100:
        yield x
        x = x * x
```

让我们看看它是如何工作的：

```{code-cell} python3
g
```

```{code-cell} python3
gen = g(2)
type(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
next(gen)
```

```{code-cell} python3
---
tags: [raises-exception]
---
next(gen)
```

调用 `gen = g(2)` 将 `gen` 绑定到一个生成器。

在生成器内部，名称 `x` 被绑定到 `2`。

当我们调用 `next(gen)` 时：

* `g()` 的函数体执行到 `yield x` 这一行，并返回 `x` 的值。

注意 `x` 的值在生成器内部被保留。

当我们再次调用 `next(gen)` 时，执行从*上次停止的地方*继续：

```{code-cell} python3
def g(x):
    while x < 100:
        yield x
        x = x * x  # 从这里继续执行
```

当 `x < 100` 不成立时，生成器抛出 `StopIteration` 错误。

顺便说一下，生成器内部的循环可以是无限的：

```{code-cell} python3
def g(x):
    while 1:
        yield x
        x = x * x
```

### 迭代器的优势

在这里使用迭代器有什么优势呢？

假设我们想对二项分布（n，0.5）进行采样。

一种方法如下：

```{code-cell} python3
import random
n = 10000000
draws = [random.uniform(0, 1) < 0.5 for i in range(n)]
sum(draws)
```

但我们在这里创建了两个巨大的列表：`range(n)` 和 `draws`。

这消耗大量内存且非常慢。

如果我们将 `n` 变得更大，就会发生这种情况：

```{code-cell} python3
---
tags: [raises-exception]
---
n = 100000000
draws = [random.uniform(0, 1) < 0.5 for i in range(n)]
```

我们可以使用迭代器来避免这些问题。

这是生成器函数：

```{code-cell} python3
def f(n):
    i = 1
    while i <= n:
        yield random.uniform(0, 1) < 0.5
        i += 1
```

现在让我们进行求和：

```{code-cell} python3
n = 10000000
draws = f(n)
draws
```

```{code-cell} python3
sum(draws)
```

总结一下，可迭代对象：

* 避免了创建大型列表/元组的需要，以及
* 提供了一个统一的迭代接口，可以在 `for` 循环中透明地使用。

## 练习


```{exercise-start}
:label: paf_ex1
```

完成以下代码，并使用[此 csv 文件](https://raw.githubusercontent.com/QuantEcon/lecture-python-programming/master/source/_static/lecture_specific/python_advanced_features/test_table.csv)进行测试，我们假设你已将该文件放在当前工作目录中：

```{code-block} python3
:class: no-execute

def column_iterator(target_file, column_number):
    """A generator function for CSV files.
    When called with a file name target_file (string) and column number
    column_number (integer), the generator function returns a generator
    that steps through the elements of column column_number in file
    target_file.
    """
    # put your code here

dates = column_iterator('test_table.csv', 1)

for date in dates:
    print(date)
```

```{exercise-end}
```

```{solution-start} paf_ex1
:class: dropdown
```

一种解决方案如下：

```{code-cell} python3
def column_iterator(target_file, column_number):
    """A generator function for CSV files.
    When called with a file name target_file (string) and column number
    column_number (integer), the generator function returns a generator
    which steps through the elements of column column_number in file
    target_file.
    """
    f = open(target_file, 'r')
    for line in f:
        yield line.split(',')[column_number - 1]
    f.close()

dates = column_iterator('test_table.csv', 1)

i = 1
for date in dates:
    print(date)
    if i == 10:
        break
    i += 1
```

```{solution-end}
```
