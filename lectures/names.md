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
  Overview: 概述
  Variable Names in Python: Python 中的变量名称
  Namespaces: 命名空间
  Viewing Namespaces: 查看命名空间
  Interactive Sessions: 交互式会话
  The Global Namespace: 全局命名空间
  Local Namespaces: 局部命名空间
  The `__builtins__` Namespace: '`__builtins__` 命名空间'
  Name Resolution: 名称解析
  'Name Resolution::{index}`Mutable <single: Mutable>` Versus {index}`Immutable <single: Immutable>` Parameters': '{index}`可变 <single: Mutable>` 与 {index}`不可变 <single: Immutable>` 参数'
---

(oop_names)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# 名称与命名空间

## 概述

本讲座的主题是变量名称、如何使用它们以及 Python 解释器如何理解它们。

这听起来可能有些枯燥，但 Python 采用的名称处理模型既优雅又有趣。

此外，如果你对 Python 中名称的工作原理有深入理解，将为自己节省大量调试时间。

(var_names)=
## Python 中的变量名称

```{index} single: Python; Variable Names
```

考虑以下 Python 语句：

```{code-cell} python3
x = 42
```

我们现在知道，当这条语句被执行时，Python 会在计算机内存中创建一个 `int` 类型的对象，其中包含：

* 值 `42`
* 一些相关属性

但 `x` 本身是什么呢？

在 Python 中，`x` 被称为**名称**，语句 `x = 42` 将名称 `x` **绑定**到我们刚刚讨论的整数对象上。

在底层，这个将名称绑定到对象的过程是作为字典实现的——稍后我们会详细介绍。

将两个或多个名称绑定到同一个对象是完全没有问题的，无论该对象是什么：

```{code-cell} python3
def f(string):      # Create a function called f
    print(string)   # that prints any string it's passed

g = f
id(g) == id(f)
```

```{code-cell} python3
g('test')
```

第一步，创建了一个函数对象，名称 `f` 被绑定到它上面。

将名称 `g` 绑定到同一个对象之后，我们可以在任何使用 `f` 的地方使用 `g`。

当绑定到某个对象的名称数量降为零时会发生什么？

下面是这种情况的一个例子，名称 `x` 首先被绑定到一个对象，然后被**重新绑定**到另一个对象：

```{code-cell} python3
x = 'foo'
id(x)
x = 'bar'  
id(x)
```

在这种情况下，当我们将 `x` 重新绑定到 `'bar'` 之后，没有任何名称绑定到第一个对象 `'foo'`。

这会触发 `'foo'` 被垃圾回收。

换句话说，存储该对象的内存空间被释放并归还给操作系统。

垃圾回收实际上是计算机科学中一个活跃的研究领域。

如果你感兴趣，可以[阅读更多关于垃圾回收的内容](https://rushter.com/blog/python-garbage-collector/)。

## 命名空间

```{index} single: Python; Namespaces
```

回顾前面的讨论，语句

```{code-cell} python3
x = 42
```

将名称 `x` 绑定到右侧的整数对象上。

我们还提到，这个将 `x` 绑定到正确对象的过程是作为字典实现的。

这个字典被称为命名空间。

```{admonition} 定义
**命名空间**是一个符号表，将名称映射到内存中的对象。
```

Python 使用多个命名空间，并根据需要动态创建它们。

例如，每次我们导入一个模块时，Python 都会为该模块创建一个命名空间。

为了观察这一过程，假设我们编写一个脚本 `mathfoo.py`，其中只有一行：

```{code-cell} python3
%%file mathfoo.py
pi = 'foobar'
```

现在我们启动 Python 解释器并导入它：

```{code-cell} python3
import mathfoo
```

接下来让我们从标准库导入 `math` 模块：

```{code-cell} python3
import math
```

这两个模块都有一个叫做 `pi` 的属性：

```{code-cell} python3
math.pi
```

```{code-cell} python3
mathfoo.pi
```

`pi` 的这两个不同绑定存在于不同的命名空间中，每个命名空间都作为一个字典实现。

如果你愿意，可以使用 `module_name.__dict__` 直接查看字典。

```{code-cell} python3
import math

math.__dict__.items()
```

```{code-cell} python3
import mathfoo

mathfoo.__dict__
```

如你所知，我们使用点属性符号来访问命名空间中的元素：

```{code-cell} python3
math.pi
```

这与 `math.__dict__['pi']` 完全等价：

```{code-cell} python3
math.__dict__['pi'] 
```

## 查看命名空间

如上所示，可以通过输入 `math.__dict__` 来打印 `math` 命名空间。

另一种查看其内容的方式是输入 `vars(math)`：

```{code-cell} python3
vars(math).items()
```

如果你只想查看名称，可以输入：

```{code-cell} python3
# 显示前 10 个名称
dir(math)[0:10]
```

注意特殊名称 `__doc__` 和 `__name__`。

这些名称在任何模块被导入时都会在命名空间中初始化：

* `__doc__` 是模块的文档字符串
* `__name__` 是模块的名称

```{code-cell} python3
print(math.__doc__)
```

```{code-cell} python3
math.__name__
```

## 交互式会话

```{index} single: Python; Interpreter
```

在 Python 中，解释器执行的**所有**代码都在某个模块中运行。

那么在提示符处输入的命令呢？

这些命令也被视为在某个模块内执行——在这种情况下，该模块称为 `__main__`。

为了验证这一点，我们可以通过在提示符处查看 `__name__` 的值来查看当前模块名称：

```{code-cell} python3
print(__name__)
```

当我们使用 IPython 的 `run` 命令运行脚本时，文件的内容也作为 `__main__` 的一部分执行。

为了观察这一点，让我们创建一个文件 `mod.py`，打印其自身的 `__name__` 属性：

```{code-cell} ipython
%%file mod.py
print(__name__)
```

现在让我们看看在 IPython 中运行它的两种不同方式：

```{code-cell} python3
import mod  # Standard import
```

```{code-cell} ipython
%run mod.py  # Run interactively
```

在第二种情况下，代码作为 `__main__` 的一部分执行，所以 `__name__` 等于 `__main__`。

要查看 `__main__` 的命名空间内容，我们使用 `vars()` 而不是 `vars(__main__)`。

如果你在 IPython 中这样做，你会看到很多 IPython 在启动会话时需要并初始化的变量。

如果你只想查看自己初始化的变量，请使用 `%whos`：

```{code-cell} ipython
x = 2
y = 3

import numpy as np

%whos
```

## 全局命名空间

```{index} single: Python; Namespace (Global)
```

Python 文档中经常提到"全局命名空间"。

全局命名空间是*当前正在执行的模块的命名空间*。

例如，假设我们启动解释器并开始进行赋值操作。

我们现在在模块 `__main__` 中工作，因此 `__main__` 的命名空间就是全局命名空间。

接下来，我们导入一个名为 `amodule` 的模块：

```{code-block} python3
:class: no-execute

import amodule
```

此时，解释器为模块 `amodule` 创建一个命名空间，并开始执行模块中的命令。

在此过程中，命名空间 `amodule.__dict__` 就是全局命名空间。

模块执行完毕后，解释器返回到发出导入语句的模块。

在本例中是 `__main__`，所以 `__main__` 的命名空间再次成为全局命名空间。

## 局部命名空间

```{index} single: Python; Namespace (Local)
```

重要事实：当我们调用一个函数时，解释器会为该函数创建一个*局部命名空间*，并在该命名空间中注册变量。

这样做的原因稍后将会解释。

局部命名空间中的变量称为*局部变量*。

函数返回后，命名空间被释放并消失。

在函数执行期间，我们可以用 `locals()` 查看局部命名空间的内容。

例如，考虑：

```{code-cell} python3
def f(x):
    a = 2
    print(locals())
    return a * x
```

现在让我们调用该函数：

```{code-cell} python3
f(1)
```

你可以看到 `f` 在被销毁之前的局部命名空间。

## `__builtins__` 命名空间

```{index} single: Python; Namespace (__builtins__)
```

我们一直在使用各种内置函数，例如 `max()、dir()、str()、list()、len()、range()、type()` 等。

这些名称的访问是如何工作的？

* 这些定义存储在一个名为 `__builtin__` 的模块中。
* 它们有自己的命名空间，称为 `__builtins__`。

```{code-cell} python3
# 显示 `__main__` 中的前 10 个名称
dir()[0:10]
```

```{code-cell} python3
# 显示 `__builtins__` 中的前 10 个名称
dir(__builtins__)[0:10]
```

我们可以按如下方式访问命名空间中的元素：

```{code-cell} python3
__builtins__.max
```

但 `__builtins__` 是特殊的，因为我们也可以始终直接访问它们：

```{code-cell} python3
max
```

```{code-cell} python3
__builtins__.max == max
```

下一节将解释这是如何工作的……

## 名称解析

```{index} single: Python; Namespace (Resolution)
```

命名空间非常有用，因为它们帮助我们组织变量名称。

（在提示符处输入 `import this`，查看打印出的最后一项）

然而，我们确实需要理解 Python 解释器如何处理多个命名空间。

理解执行流程将帮助我们在编写和调试程序时检查哪些变量在作用域内以及如何操作它们。

在任何执行时刻，实际上至少有两个可以直接访问的命名空间。

（"直接访问"意味着不使用点号，例如 `pi` 而不是 `math.pi`）

这些命名空间是：

* 全局命名空间（当前正在执行的模块的）
* 内置命名空间

如果解释器正在执行一个函数，那么可以直接访问的命名空间是：

* 函数的局部命名空间
* 全局命名空间（当前正在执行的模块的）
* 内置命名空间

有时函数定义在其他函数内部，如下所示：

```{code-cell} python3
def f():
    a = 2
    def g():
        b = 4
        print(a * b)
    g()
```

这里 `f` 是 `g` 的*外层函数*，每个函数都有自己的命名空间。

现在我们可以给出命名空间解析的规则：

解释器搜索名称的顺序是：

1. 局部命名空间（如果存在）
1. 外层命名空间的层次结构（如果存在）
1. 全局命名空间
1. 内置命名空间

如果在这些命名空间中都找不到该名称，解释器会引发 `NameError`。

这被称为 **LEGB 规则**（局部、外层、全局、内置）。

下面是一个有助于说明的例子。

这里的可视化图由 Jupyter notebook 中的 [nbtutor](https://github.com/lgpage/nbtutor) 创建。

它们可以帮助你在学习新语言时更好地理解程序。

考虑一个如下所示的脚本 `test.py`：

```{code-cell} python3
%%file test.py
def g(x):
    a = 1
    x = x + a
    return x

a = 0
y = g(10)
print("a = ", a, "y = ", y)
```

当我们运行这个脚本时会发生什么？

```{code-cell} ipython
%run test.py
```

首先：

* 全局命名空间 `{}` 被创建。

```{figure} /_static/lecture_specific/oop_intro/global.png
```

* 函数对象被创建，`g` 在全局命名空间中绑定到它。
* 名称 `a` 被绑定到 `0`，同样在全局命名空间中。

```{figure} /_static/lecture_specific/oop_intro/global2.png
```

接下来通过 `y = g(10)` 调用 `g`，导致以下一系列操作：

* 为函数创建局部命名空间。
* 局部名称 `x` 和 `a` 被绑定，使局部命名空间变为 `{'x': 10, 'a': 1}`。

注意全局的 `a` 不受局部的 `a` 影响。

```{figure} /_static/lecture_specific/oop_intro/local1.png
```

* 语句 `x = x + a` 使用局部的 `a` 和局部的 `x` 计算 `x + a`，并将局部名称 `x` 绑定到结果。
* 该值被返回，`y` 在全局命名空间中绑定到它。
* 局部的 `x` 和 `a` 被丢弃（局部命名空间被释放）。

```{figure} /_static/lecture_specific/oop_intro/local_return.png
```

(mutable_vs_immutable)=
### {index}`可变 <single: Mutable>` 与 {index}`不可变 <single: Immutable>` 参数

现在是进一步讨论可变对象与不可变对象的好时机。

考虑以下代码段：

```{code-cell} python3
def f(x):
    x = x + 1
    return x

x = 1
print(f(x), x)
```

我们现在理解这里会发生什么：代码打印 `2` 作为 `f(x)` 的值，打印 `1` 作为 `x` 的值。

首先，`f` 和 `x` 在全局命名空间中注册。

调用 `f(x)` 创建一个局部命名空间，并将 `x` 添加到其中，绑定到 `1`。

接下来，这个局部的 `x` 被重新绑定到新的整数对象 `2`，并返回该值。

这一切都不影响全局的 `x`。

然而，当我们使用**可变**数据类型（例如列表）时，情况就不同了：

```{code-cell} python3
def f(x):
    x[0] = x[0] + 1
    return x

x = [1]
print(f(x), x)
```

这会打印 `[2]` 作为 `f(x)` 的值，`x` 的值*相同*。

以下是发生的情况：

* `f` 在全局命名空间中注册为一个函数

```{figure} /_static/lecture_specific/oop_intro/mutable1.png
```

* `x` 在全局命名空间中绑定到 `[1]`

```{figure} /_static/lecture_specific/oop_intro/mutable2.png
```

* 调用 `f(x)`
    * 创建一个局部命名空间
    * 将 `x` 添加到局部命名空间，绑定到 `[1]`

```{figure} /_static/lecture_specific/oop_intro/mutable3.png
```

```{note}
全局的 `x` 和局部的 `x` 指向同一个 `[1]`
```

我们可以看到局部 `x` 的标识符和全局 `x` 的标识符是相同的：

```{code-cell} python3
def f(x):
    x[0] = x[0] + 1
    print(f'the identity of local x is {id(x)}')
    return x

x = [1]
print(f'the identity of global x is {id(x)}')
print(f(x), x)
```

* 在 `f(x)` 内部
    * 列表 `[1]` 被修改为 `[2]`
    * 返回列表 `[2]`

```{figure} /_static/lecture_specific/oop_intro/mutable4.png
```

* 局部命名空间被释放，局部的 `x` 消失

```{figure} /_static/lecture_specific/oop_intro/mutable5.png
```

如果你想分别修改局部的 `x` 和全局的 `x`，可以创建列表的[*副本*](https://docs.python.org/3/library/copy.html)，并将副本赋值给局部的 `x`。

我们将留给你自行探索。