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
  Objects: 对象
  Objects::Type: 类型
  Objects::Identity: 标识符
  'Objects::Object Content: Data and Attributes': 对象内容：数据与属性
  Objects::Methods: 方法
  Inspection Using Rich: 使用 Rich 进行检查
  A Little Mystery: 一个小谜题
  Summary: 总结
  Exercises: 练习
---

(oop_intro)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# OOP I：对象与方法

## 概述

传统的编程范式（如 Fortran、C、MATLAB 等）被称为[过程式编程](https://en.wikipedia.org/wiki/Procedural_programming)。

其工作方式如下：

* 程序具有一个状态，对应于其变量的值。
* 调用函数来对状态进行操作和转换。
* 最终输出通过一系列函数调用产生。

另外两种重要的编程范式是[面向对象编程](https://en.wikipedia.org/wiki/Object-oriented_programming)（OOP）和[函数式编程](https://en.wikipedia.org/wiki/Functional_programming)。

在面向对象编程范式中，数据和函数被捆绑在一起形成"对象"——在这种情境下，函数被称为**方法**。

方法被调用来转换对象中包含的数据。

* 可以想象一个 Python 列表，它包含数据，并具有诸如 `append()` 和 `pop()` 这样用于转换数据的方法。

函数式编程语言建立在函数组合的思想之上。

* 有影响力的例子包括 [Lisp](https://en.wikipedia.org/wiki/Common_Lisp)、[Haskell](https://en.wikipedia.org/wiki/Haskell) 和 [Elixir](https://en.wikipedia.org/wiki/Elixir_(programming_language))。

那么 Python 属于哪一类呢？

实际上，Python 是一种实用主义语言，它融合了面向对象、函数式和过程式风格，而非采取纯粹主义的方法。

一方面，这使得 Python 及其用户能够有选择地借鉴不同范式的优点。

另一方面，这种不纯粹性有时可能会导致一些混淆。

幸运的是，如果你理解 Python 在基础层面上*是*面向对象的，这种混淆就会降到最低。

我们的意思是，在 Python 中，*一切皆对象*。

在本讲座中，我们将解释这句话的含义以及它为何重要。

我们将使用以下第三方库：

```{code-cell} python3
:tags: [hide-output]
!pip install rich
```

## 对象

```{index} single: Python; Objects
```

在 Python 中，*对象*是存储在计算机内存中的数据和指令的集合，包含：

1. 类型
1. 唯一标识符
1. 数据（即内容）
1. 方法

以下将依次定义和讨论这些概念。

(type)=
### 类型

```{index} single: Python; Type
```

Python 提供了不同类型的对象，以适应不同类别的数据。

例如：

```{code-cell} python3
s = 'This is a string'
type(s)
```

```{code-cell} python3
x = 42   # Now let's create an integer
type(x)
```

对象的类型对许多表达式都很重要。

例如，两个字符串之间的加法运算符表示连接：

```{code-cell} python3
'300' + 'cc'
```

而两个数字之间则表示普通加法：

```{code-cell} python3
300 + 400
```

考虑以下表达式：

```{code-cell} python3
---
tags: [raises-exception]
---
'300' + 400
```

这里我们混合了类型，Python 不清楚用户是想要

* 将 `'300'` 转换为整数，然后将其与 `400` 相加，还是
* 将 `400` 转换为字符串，然后将其与 `'300'` 连接

某些语言可能会尝试猜测，但 Python 是*强类型*的：

* 类型很重要，隐式类型转换很少发生。
* Python 会通过引发 `TypeError` 来响应。

为了避免错误，你需要通过更改相关类型来明确你的意图。

例如：

```{code-cell} python3
int('300') + 400   # To add as numbers, change the string to an integer
```

(identity)=
### 标识符

```{index} single: Python; Identity
```

在 Python 中，每个对象都有一个唯一标识符，帮助 Python（和我们）跟踪该对象。

可以通过 `id()` 函数获取对象的标识符：

```{code-cell} python3
y = 2.5
z = 2.5
id(y)
```

```{code-cell} python3
id(z)
```

在这个例子中，`y` 和 `z` 恰好具有相同的值（即 `2.5`），但它们不是同一个对象。

对象的标识符实际上就是该对象在内存中的地址。

### 对象内容：数据与属性

```{index} single: Python; Content
```

如果我们设置 `x = 42`，那么我们创建了一个包含数据 `42` 的 `int` 类型对象。

实际上，它包含的内容更多，如以下示例所示：

```{code-cell} python3
x = 42
x
```

```{code-cell} python3
x.imag
```

```{code-cell} python3
x.__class__
```

当 Python 创建这个整数对象时，它会随之存储各种辅助信息，例如虚部和类型。

点号后面的任何名称都称为点号左侧对象的*属性*。

* 例如，`imag` 和 `__class__` 是 `x` 的属性。

从这个例子中我们可以看到，对象具有包含辅助信息的属性。

它们还具有像函数一样起作用的属性，称为*方法*。

这些属性非常重要，让我们深入讨论它们。

(methods)=
### 方法

```{index} single: Python; Methods
```

方法是*与对象捆绑在一起的函数*。

形式上，方法是对象的**可调用**属性——即可以作为函数调用的属性：

```{code-cell} python3
x = ['foo', 'bar']
callable(x.append)
```

```{code-cell} python3
callable(x.__doc__)
```

方法通常作用于它们所属对象中包含的数据，或将该数据与其他数据结合：

```{code-cell} python3
x = ['a', 'b']
x.append('c')
s = 'This is a string'
s.upper()
```

```{code-cell} python3
s.lower()
```

```{code-cell} python3
s.replace('This', 'That')
```

Python 的大量功能都是围绕方法调用组织的。

例如，考虑以下代码：

```{code-cell} python3
x = ['a', 'b']
x[0] = 'aa'  # Item assignment using square bracket notation
x
```

看起来这里没有使用任何方法，但实际上方括号赋值符号只是方法调用的一个便捷接口。

实际发生的是 Python 调用了 `__setitem__` 方法，如下所示：

```{code-cell} python3
x = ['a', 'b']
x.__setitem__(0, 'aa')  # Equivalent to x[0] = 'aa'
x
```

（如果你愿意，你可以修改 `__setitem__` 方法，使方括号赋值做一些完全不同的事情）

## 使用 Rich 进行检查

有一个名为 [rich](https://github.com/Textualize/rich) 的好用包，可以帮助我们查看对象的内容。

例如：

```{code-cell} python3
from rich import inspect
x = 10
inspect(10)
```

如果我们也想查看方法，可以使用：

```{code-cell} python3
inspect(10, methods=True)
```

实际上还有更多方法，如果你执行 `inspect(10, all=True)` 可以看到。

## 一个小谜题

在本讲座中，我们声称 Python 在本质上是一种面向对象的语言。

但以下是一个看起来更像过程式的例子：

```{code-cell} python3
x = ['a', 'b']
m = len(x)
m
```

如果 Python 是面向对象的，为什么我们不使用 `x.len()` 呢？

答案与 Python 追求可读性和一致风格这一事实有关。

在 Python 中，用户构建自定义对象是很常见的——我们将在{doc}`后面 <python_oop>`讨论如何做到这一点。

用户很常见地会向他们的对象添加方法来测量对象的长度（根据适当的定义）。

在命名这样的方法时，自然的选择是 `len()` 和 `length()`。

如果一些用户选择 `len()`，而另一些用户选择 `length()`，那么风格将不一致，更难记忆。

为了避免这种情况，Python 的创建者选择将 `len()` 作为内置函数，以强调 `len()` 是约定俗成的用法。

话虽如此，Python 在底层*仍然*是面向对象的。

实际上，上面讨论的列表 `x` 有一个名为 `__len__()` 的方法。

`len()` 函数所做的就是调用这个方法。

换句话说，以下代码是等价的：

```{code-cell} python3
x = ['a', 'b']
len(x)
```

和

```{code-cell} python3
x = ['a', 'b']
x.__len__()
```

## 总结

本讲座的核心信息很明确：

* 在 Python 中，*内存中的一切都被视为对象*。

这不仅包括列表、字符串等，还包括一些不那么明显的东西，例如：

* 函数（一旦被读入内存）
* 模块（同上）
* 打开用于读取或写入的文件
* 整数等

记住一切皆对象将帮助你与程序交互并编写清晰的 Python 风格代码。

## 练习

```{exercise-start}
:label: oop_intro_ex1
```

我们之前已经接触过{any}`布尔数据类型 <boolean>`。

利用本讲座中学到的知识，打印布尔对象 `True` 的方法列表。

```{hint}
:class: dropdown

你可以使用 `callable()` 来测试对象的某个属性是否可以作为函数调用。
```

```{exercise-end}
```

```{solution-start} oop_intro_ex1
:class: dropdown
```

首先，我们需要找到 `True` 的所有属性，可以通过以下方式完成：

```{code-cell} python3
print(sorted(True.__dir__()))
```

或者：

```{code-cell} python3
print(sorted(dir(True)))
```

由于布尔数据类型是一种原始类型，你也可以在内置命名空间中找到它：

```{code-cell} python3
print(dir(__builtins__.bool))
```

这里我们使用 `for` 循环来筛选出可调用的属性：

```{code-cell} python3
attributes = dir(__builtins__.bool)
callablels = []

for attribute in attributes:
  # 使用 eval() 将字符串作为表达式求值
  if callable(eval(f'True.{attribute}')):
    callablels.append(attribute)
print(callablels)
```

```{solution-end}
```