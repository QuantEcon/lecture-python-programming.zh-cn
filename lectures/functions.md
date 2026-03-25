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
  title: 函数
  headings:
    Overview: 概述
    Function Basics: 函数基础
    Function Basics::Built-In Functions: 内置函数
    Function Basics::Third Party Functions: 第三方函数
    Defining Functions: 定义函数
    Defining Functions::Basic Syntax: 基本语法
    Defining Functions::Keyword Arguments: 关键字参数
    Defining Functions::The Flexibility of Python Functions: Python 函数的灵活性
    'Defining Functions::One-Line Functions: `lambda`': 单行函数：`lambda`
    Defining Functions::Why Write Functions?: 为什么要编写函数？
    Applications: 应用
    Applications::Random Draws: 随机抽取
    Applications::Adding Conditions: 添加条件
    Recursive Function Calls (Advanced): 递归函数调用（进阶）
    Exercises: 练习
    Advanced Exercises: 进阶练习
---

(functions)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# 函数

```{index} single: Python; User-defined functions
```

## 概述

函数是几乎所有编程语言都提供的极其有用的构造。

我们已经接触过几个函数，例如

* NumPy 中的 `sqrt()` 函数，以及
* 内置的 `print()` 函数

在本讲座中，我们将

1. 系统地介绍函数，涵盖语法和使用场景，以及
2. 学习构建我们自己的用户自定义函数。

我们将使用以下导入。

```{code-cell} ipython
import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n
```

## 函数基础

函数是程序中实现特定任务的命名代码段。

许多函数已经存在，我们可以直接使用它们。

首先我们回顾这些函数，然后讨论如何构建我们自己的函数。

### 内置函数

Python 有许多**内置**函数，无需 `import` 即可使用。

我们已经接触过一些

```{code-cell} python3
max(19, 20)
```

```{code-cell} python3
print('foobar')
```

```{code-cell} python3
str(22)
```

```{code-cell} python3
type(22)
```

Python 内置函数的完整列表在[这里](https://docs.python.org/3/library/functions.html)。


### 第三方函数

如果内置函数不能满足我们的需求，我们要么需要导入函数，要么创建自己的函数。

导入和使用函数的示例已在{doc}`上一讲 <python_by_example>`中给出。

这里再举一个例子，用于检验给定年份是否为闰年：

```{code-cell} python3
import calendar
calendar.isleap(2024)
```

## 定义函数

在许多情况下，能够定义我们自己的函数是非常有用的。

让我们从讨论如何定义函数开始。

### 基本语法

这是一个非常简单的 Python 函数，实现了数学函数 $f(x) = 2 x + 1$

```{code-cell} python3
def f(x):
    return 2 * x + 1
```

现在我们已经定义了这个函数，让我们*调用*它并检验它是否符合预期：

```{code-cell} python3
f(1)   
```

```{code-cell} python3
f(10)
```

这是一个较长的函数，用于计算给定数字的绝对值。

（这样的函数已经作为内置函数存在了，但我们作为练习来编写自己的版本。）

```{code-cell} python3
def new_abs_function(x):
    if x < 0:
        abs_value = -x
    else:
        abs_value = x
    return abs_value
```

让我们回顾一下这里的语法。

* `def` 是用于开始函数定义的 Python 关键字。
* `def new_abs_function(x):` 表明该函数名为 `new_abs_function`，且有一个参数 `x`。
* 缩进的代码是称为*函数体*的代码块。
* `return` 关键字表示 `abs_value` 是应该返回给调用代码的对象。

整个函数定义由 Python 解释器读取并存储在内存中。

让我们调用它来验证它是否正常工作：

```{code-cell} python3
print(new_abs_function(3))
print(new_abs_function(-3))
```


注意，一个函数可以有任意多个 `return` 语句（包括零个）。

函数的执行在遇到第一个 return 时终止，这允许如下示例的代码

```{code-cell} python3
def f(x):
    if x < 0:
        return 'negative'
    return 'nonnegative'
```

（通常不鼓励编写具有多个 return 语句的函数，因为这会使逻辑难以跟踪。）

没有 return 语句的函数会自动返回特殊的 Python 对象 `None`。

(pos_args)=
### 关键字参数

```{index} single: Python; keyword arguments
```

在{ref}`前一讲 <python_by_example>`中，你遇到了以下语句

```{code-block} python3
:class: no-execute

plt.plot(x, 'b-', label="white noise")
```

在这个对 Matplotlib 的 `plot` 函数的调用中，注意最后一个参数是用 `name=argument` 语法传递的。

这称为*关键字参数*，其中 `label` 是关键字。

非关键字参数称为*位置参数*，因为它们的含义由顺序决定

* `plot(x, 'b-')` 与 `plot('b-', x)` 不同

当函数有很多参数时，关键字参数特别有用，因为此时很难记住正确的顺序。

你可以毫无困难地在用户自定义函数中采用关键字参数。

下一个示例说明了语法

```{code-cell} python3
def f(x, a=1, b=1):
    return a + b * x
```

我们在 `f` 的定义中提供的关键字参数值成为默认值

```{code-cell} python3
f(2)
```

它们可以按如下方式修改

```{code-cell} python3
f(2, a=4, b=5)
```

### Python 函数的灵活性

正如我们在{ref}`前一讲 <python_by_example>`中讨论的，Python 函数非常灵活。

特别是

* 在给定文件中可以定义任意数量的函数。
* 函数可以（且经常）在其他函数内部定义。
* 任何对象都可以作为参数传递给函数，包括其他函数。
* 函数可以返回任何类型的对象，包括函数。

我们将在以下章节中给出将函数传递给函数是多么简单的示例。

### 单行函数：`lambda`

```{index} single: Python; lambda functions
```

`lambda` 关键字用于在一行上创建简单函数。

例如，以下定义

```{code-cell} python3
def f(x):
    return x**3
```

和

```{code-cell} python3
f = lambda x: x**3
```

是完全等价的。

为了理解 `lambda` 为什么有用，假设我们想计算 $\int_0^2 x^3 dx$（并且忘记了高中微积分）。

SciPy 库有一个名为 `quad` 的函数可以为我们完成这个计算。

`quad` 函数的语法是 `quad(f, a, b)`，其中 `f` 是一个函数，`a` 和 `b` 是数字。

要创建函数 $f(x) = x^3$，我们可以如下使用 `lambda`

```{code-cell} python3
from scipy.integrate import quad

quad(lambda x: x**3, 0, 2)
```

这里由 `lambda` 创建的函数被称为*匿名*函数，因为它从未被赋予名称。


### 为什么要编写函数？

用户自定义函数对于提高代码的清晰度非常重要，通过

* 分离不同的逻辑线索
* 促进代码重用

（将同样的事情写两遍[几乎总是一个坏主意](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)）

我们将在{doc}`后面 <writing_good_code>`进一步讨论这个问题。

## 应用

### 随机抽取

再次考虑{doc}`前一讲 <python_by_example>`中的以下代码

```{code-cell} python3
ts_length = 100
ϵ_values = []   # 空列表

for i in range(ts_length):
    e = np.random.randn()
    ϵ_values.append(e)

plt.plot(ϵ_values)
plt.show()
```

我们将把这个程序分成两部分：

1. 一个生成随机变量列表的用户自定义函数。
1. 程序的主要部分，它
    1. 调用此函数获取数据
    1. 绘制数据

这在下一个程序中实现

(funcloopprog)=
```{code-cell} python3
def generate_data(n):
    ϵ_values = []
    for i in range(n):
        e = np.random.randn()
        ϵ_values.append(e)
    return ϵ_values

data = generate_data(100)
plt.plot(data)
plt.show()
```

当解释器到达表达式 `generate_data(100)` 时，它以 `n` 等于 100 执行函数体。

最终结果是名称 `data` 被*绑定*到函数返回的列表 `ϵ_values`。

### 添加条件

```{index} single: Python; Conditions
```

我们的函数 `generate_data()` 功能相当有限。

让我们通过赋予它根据需要返回标准正态分布或 $(0, 1)$ 上均匀随机变量的能力，使其稍微更有用。

这在下面的代码中实现。

(funcloopprog2)=
```{code-cell} python3
def generate_data(n, generator_type):
    ϵ_values = []
    for i in range(n):
        if generator_type == 'U':
            e = np.random.uniform(0, 1)
        else:
            e = np.random.randn()
        ϵ_values.append(e)
    return ϵ_values

data = generate_data(100, 'U')
plt.plot(data)
plt.show()
```

希望 if/else 子句的语法是不言自明的，缩进再次划定了代码块的范围。

注意

* 我们将参数 `U` 作为字符串传递，这就是为什么我们将其写为 `'U'`。
* 注意等号测试使用 `==` 语法，而不是 `=`。
    * 例如，语句 `a = 10` 将名称 `a` 赋值给值 `10`。
    * 表达式 `a == 10` 根据 `a` 的值求值为 `True` 或 `False`。

现在，有几种方法可以简化上面的代码。

例如，我们可以完全去掉条件判断，只需将所需的生成器类型*作为函数*传递。

要理解这一点，请考虑以下版本。

(test_program_6)=
```{code-cell} python3
def generate_data(n, generator_type):
    ϵ_values = []
    for i in range(n):
        e = generator_type()
        ϵ_values.append(e)
    return ϵ_values

data = generate_data(100, np.random.uniform)
plt.plot(data)
plt.show()
```

现在，当我们调用函数 `generate_data()` 时，我们将 `np.random.uniform` 作为第二个参数传递。

这个对象是一个*函数*。

当函数调用 `generate_data(100, np.random.uniform)` 被执行时，Python 以 `n` 等于 100 和名称 `generator_type` "绑定"到函数 `np.random.uniform` 来运行函数代码块。

* 在这些行被执行时，名称 `generator_type` 和 `np.random.uniform` 是"同义词"，可以以相同的方式使用。

这个原则更普遍地适用——例如，考虑以下代码

```{code-cell} python3
max(7, 2, 4)   # max() 是 Python 内置函数
```

```{code-cell} python3
m = max
m(7, 2, 4)
```

这里我们为内置函数 `max()` 创建了另一个名称，然后可以以相同的方式使用它。

在我们程序的背景下，将新名称绑定到函数的能力意味着*将函数作为参数传递给另一个函数*没有任何问题——正如我们上面所做的那样。


(recursive_functions)=
## 递归函数调用（进阶）

```{index} single: Python; Recursion
```

这是一个进阶主题，你可以随意跳过。

同时，这是一个很好的想法，你应该在编程生涯的某个阶段学习它。

基本上，递归函数是一个调用自身的函数。

例如，考虑在某个 t 时计算 $x_t$ 的问题，其中

```{math}
:label: xseqdoub

x_{t+1} = 2 x_t, \quad x_0 = 1
```

显然答案是 $2^t$。

我们可以用循环很容易地计算这个

```{code-cell} python3
def x_loop(t):
    x = 1
    for i in range(t):
        x = 2 * x
    return x
```

我们也可以使用递归解，如下所示

```{code-cell} python3
def x(t):
    if t == 0:
        return 1
    else:
        return 2 * x(t-1)
```

这里发生的情况是每次连续调用都在*栈*中使用其自己的*帧*

* 帧是存储给定函数调用的局部变量的地方
* 栈是用于处理函数调用的内存
  * 一个先进后出（FILO）队列

这个例子有些刻意，因为第一个（迭代）解通常比递归解更受欢迎。

我们以后会遇到递归的不那么刻意的应用。


(factorial_exercise)=
## 练习

```{exercise-start}
:label: func_ex1
```

回想一下，$n!$ 读作"$n$ 的阶乘"，定义为
$n! = n \times (n - 1) \times \cdots \times 2 \times 1$。

我们这里只考虑 $n$ 为正整数。

各种模块中都有计算这个的函数，但让我们作为练习编写自己的版本。

特别地，编写一个函数 `factorial`，使得对任意正整数 $n$，`factorial(n)` 返回 $n!$。

```{exercise-end}
```


```{solution-start} func_ex1
:class: dropdown
```

这是一种解法：

```{code-cell} python3
def factorial(n):
    k = 1
    for i in range(n):
        k = k * (i + 1)
    return k

factorial(4)
```


```{solution-end}
```


```{exercise-start}
:label: func_ex2
```

[二项随机变量](https://en.wikipedia.org/wiki/Binomial_distribution) $Y \sim Bin(n, p)$ 表示 $n$ 次二元试验中的成功次数，其中每次试验以概率 $p$ 成功。

除了 `from numpy.random import uniform` 之外不使用任何其他导入，编写一个函数
`binomial_rv`，使得 `binomial_rv(n, p)` 生成 $Y$ 的一次抽取。

```{hint}
:class: dropdown

如果 $U$ 在 $(0, 1)$ 上均匀分布且 $p \in (0,1)$，则表达式 `U < p` 以概率 $p$ 求值为 `True`。
```

```{exercise-end}
```


```{solution-start} func_ex2
:class: dropdown
```

这是一种解法：

```{code-cell} python3
from numpy.random import uniform

def binomial_rv(n, p):
    count = 0
    for i in range(n):
        U = uniform()
        if U < p:
            count = count + 1    # 或者 count += 1
    return count

binomial_rv(10, 0.5)
```

```{solution-end}
```


```{exercise-start}
:label: func_ex3
```

首先，编写一个函数，返回以下随机装置的一次实现

1. 掷一枚无偏硬币 10 次。
1. 如果在此序列中正面连续出现 `k` 次或更多次至少一次，支付一美元。
1. 如果没有，则不支付。

其次，编写另一个函数执行相同的任务，但上述随机装置的第二条规则变为

- 如果在此序列中正面出现 `k` 次或更多次，支付一美元。

除了 `from numpy.random import uniform` 之外不使用任何其他导入。

```{exercise-end}
```

```{solution-start} func_ex3
:class: dropdown
```

这是第一个随机装置的函数。




```{code-cell} python3
from numpy.random import uniform

def draw(k):  # 如果序列中连续成功 k 次则支付

    payoff = 0
    count = 0

    for i in range(10):
        U = uniform()
        count = count + 1 if U < 0.5 else 0
        print(count)    # 打印计数以便清楚
        if count == k:
            payoff = 1

    return payoff

draw(3)
```

这是第二个随机装置的另一个函数。

```{code-cell} python3
def draw_new(k):  # 如果序列中成功 k 次则支付

    payoff = 0
    count = 0

    for i in range(10):
        U = uniform()
        count = count + ( 1 if U < 0.5 else 0 )
        print(count)
        if count == k:
            payoff = 1

    return payoff

draw_new(3)
```

```{solution-end}
```


## 进阶练习

在以下练习中，我们将一起编写递归函数。


```{exercise-start}
:label: func_ex4
```

斐波那契数列定义为

```{math}
:label: fib

x_{t+1} = x_t + x_{t-1}, \quad x_0 = 0, \; x_1 = 1
```

数列中的前几个数为 $0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55$。

编写一个函数，对任意 t 递归计算第 $t$ 个斐波那契数。

```{exercise-end}
```

```{solution-start} func_ex4
:class: dropdown
```

这是标准解法

```{code-cell} python3
def x(t):
    if t == 0:
        return 0
    if t == 1:
        return 1
    else:
        return x(t-1) + x(t-2)
```

让我们测试一下

```{code-cell} python3
print([x(i) for i in range(10)])
```

```{solution-end}
```

```{exercise-start}
:label: func_ex5
```

使用递归重写[练习 1](factorial_exercise) 中的函数 `factorial()`。

```{exercise-end}
```

```{solution-start} func_ex5
:class: dropdown
```

这是标准解法

```{code-cell} python3
def recursion_factorial(n):
   if n == 1:
       return n
   else:
       return n * recursion_factorial(n-1)
```

让我们测试一下

```{code-cell} python3
print([recursion_factorial(i) for i in range(1, 10)])
```

```{solution-end}
```