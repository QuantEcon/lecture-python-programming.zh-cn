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
  title: Python 基础要点
  headings:
    Overview: 概述
    Data Types: 数据类型
    Data Types::Primitive Data Types: 基本数据类型
    Data Types::Primitive Data Types::Boolean Values: 布尔值
    Data Types::Primitive Data Types::Numeric Types: 数值类型
    Data Types::Containers: 容器
    Data Types::Containers::Slice Notation: 切片表示法
    Data Types::Containers::Sets and Dictionaries: 集合与字典
    Input and Output: 输入与输出
    Input and Output::Paths: 路径
    Iterating: 迭代
    Iterating::Looping over Different Objects: 遍历不同对象
    Iterating::Looping without Indices: 不使用索引的循环
    Iterating::List Comprehensions: 列表推导式
    Comparisons and Logical Operators: 比较与逻辑运算符
    Comparisons and Logical Operators::Comparisons: 比较
    Comparisons and Logical Operators::Combining Expressions: 组合表达式
    Coding Style and Documentation: 编码风格与文档
    'Coding Style and Documentation::Python Style Guidelines: PEP8': Python 风格指南：PEP8
    Coding Style and Documentation::Docstrings: 文档字符串
    Exercises: 练习
---

(python_done_right)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Python 基础要点

## 概述

我们已经相当快速地涵盖了大量材料，重点放在示例上。

现在让我们以更系统的方式介绍 Python 的一些核心特性。

这种方式不那么令人兴奋，但有助于厘清一些细节。

## 数据类型

```{index} single: Python; Data Types
```

计算机程序通常需要跟踪各种数据类型。

例如，`1.5` 是浮点数，而 `1` 是整数。

程序需要区分这两种类型，原因有很多。

其一是它们在内存中的存储方式不同。

其二是算术运算不同。

* 例如，浮点算术在大多数机器上由专用的浮点运算单元（FPU）实现。

一般来说，浮点数包含更多信息，但整数上的算术运算更快且更精确。

Python 提供了许多其他内置数据类型，其中一些我们已经见过了。

* 字符串、列表等。

让我们进一步了解它们。

### 基本数据类型

(boolean)=
#### 布尔值

一种简单的数据类型是**布尔值**，其值可以是 `True` 或 `False`。

```{code-cell} python3
x = True
x
```

我们可以使用 `type()` 函数检查内存中任何对象的类型。

```{code-cell} python3
type(x)
```

在下面这行代码中，解释器计算 = 右侧的表达式并将 y 绑定到该值。

```{code-cell} python3
y = 100 < 10
y
```

```{code-cell} python3
type(y)
```

在算术表达式中，`True` 被转换为 `1`，`False` 被转换为 `0`。

这称为**布尔算术**，在编程中经常很有用。

下面是一些示例。

```{code-cell} python3
x + y
```

```{code-cell} python3
x * y
```

```{code-cell} python3
True + True
```

```{code-cell} python3
bools = [True, True, False, True]  # 布尔值列表

sum(bools)
```

#### 数值类型

数值类型也是重要的基本数据类型。

我们之前已经见过 `integer`（整数）和 `float`（浮点数）类型。

**复数**是 Python 中另一种基本数据类型。

```{code-cell} python3
x = complex(1, 2)
y = complex(2, 1)
print(x * y)

type(x)
```

### 容器

Python 有几种基本类型用于存储（可能是异构的）数据集合。

我们{ref}`已经讨论过列表 <lists_ref>`。

```{index} single: Python; Tuples
```

一种相关的数据类型是**元组**，它是"不可变"的列表。

```{code-cell} python3
x = ('a', 'b')  # 使用圆括号而非方括号
x = 'a', 'b'    # 或者不使用括号——含义完全相同
x
```

```{code-cell} python3
type(x)
```

在 Python 中，如果一个对象一旦创建就无法更改，则称该对象为**不可变的**。

相反，如果一个对象在创建后仍可以修改，则称其为**可变的**。

Python 列表是可变的。

```{code-cell} python3
x = [1, 2]
x[0] = 10
x
```

但元组不是。

```{code-cell} python3
---
tags: [raises-exception]
---
x = (1, 2)
x[0] = 10
```

稍后我们将进一步讨论可变和不可变数据的作用。

元组（和列表）可以按如下方式"解包"。

```{code-cell} python3
integers = (10, 20, 30)
x, y, z = integers
x
```

```{code-cell} python3
y
```

实际上，你{ref}`已经见过这样的例子 <tuple_unpacking_example>`了。

元组解包非常方便，我们将经常使用它。

#### 切片表示法

```{index} single: Python; Slicing
```

要访问序列（列表、元组或字符串）的多个元素，可以使用 Python 的切片表示法。

例如，

```{code-cell} python3
a = ["a", "b", "c", "d", "e"]
a[1:]
```

```{code-cell} python3
a[1:3]
```

一般规则是 `a[m:n]` 返回从 `a[m]` 开始的 `n - m` 个元素。

负数也是允许的。

```{code-cell} python3
a[-2:]  # 列表的最后两个元素
```

你还可以使用 `[start:end:step]` 格式来指定步长。

```{code-cell} python3
a[::2]
```

使用负步长，可以以相反的顺序返回序列。

```{code-cell} python3
a[-2::-1] # 从倒数第二个元素向后遍历到第一个元素
```

相同的切片表示法适用于元组和字符串。

```{code-cell} python3
s = 'foobar'
s[-3:]  # 选择最后三个元素
```

#### 集合与字典

```{index} single: Python; Sets
```

```{index} single: Python; Dictionaries
```

在继续之前，我们还需要提到另外两种容器类型：[集合](https://docs.python.org/3/tutorial/datastructures.html#sets)和[字典](https://docs.python.org/3/tutorial/datastructures.html#dictionaries)。

字典很像列表，不同之处在于其中的项目是按名称而非按编号来索引的。

```{code-cell} python3
d = {'name': 'Frodo', 'age': 33}
type(d)
```

```{code-cell} python3
d['age']
```

`'name'` 和 `'age'` 称为*键*。

键所映射到的对象（`'Frodo'` 和 `33`）称为`值`。

集合是无序且不含重复元素的集合，集合方法提供了常用的集合论运算。

```{code-cell} python3
s1 = {'a', 'b'}
type(s1)
```

```{code-cell} python3
s2 = {'b', 'c'}
s1.issubset(s2)
```

```{code-cell} python3
s1.intersection(s2)
```

`set()` 函数从序列创建集合。

```{code-cell} python3
s3 = set(('foo', 'bar', 'foo'))
s3
```

## 输入与输出

```{index} single: Python; IO
```

让我们简要回顾一下文本文件的读写，先从写入开始。

```{code-cell} python3
f = open('newfile.txt', 'w')   # 打开 'newfile.txt' 用于写入
f.write('Testing\n')           # 这里 '\n' 表示换行
f.write('Testing again')
f.close()
```

这里

* 内置函数 `open()` 创建一个用于写入的文件对象。
* `write()` 和 `close()` 都是文件对象的方法。

我们创建的这个文件在哪里？

回想一下，Python 维护着一个当前工作目录（pwd）的概念，可以在 Jupyter 或 IPython 中通过以下方式找到。

```{code-cell} ipython
%pwd
```

如果未指定路径，则 Python 会写入到该目录。

我们也可以使用 Python 读取 `newfile.txt` 的内容，如下所示。

```{code-cell} python3
f = open('newfile.txt', 'r')
out = f.read()
out
```

```{code-cell} python3
print(out)
```

事实上，现代 Python 中推荐的方法是使用 `with` 语句来确保文件被正确地获取和释放。

将操作包含在同一个块中也提高了代码的清晰度。

```{note}
这种块在正式上被称为[*上下文*](https://realpython.com/python-with-statement/#the-with-statement-approach)。
```

让我们尝试将上面的两个示例转换为 `with` 语句。

首先修改写入示例。

```{code-cell} python3

with open('newfile.txt', 'w') as f:  
    f.write('Testing\n')         
    f.write('Testing again')
```

注意，我们不需要调用 `close()` 方法，因为 `with` 块会确保在块结束时关闭流。

稍作修改，我们也可以使用 `with` 读取文件。

```{code-cell} python3
with open('newfile.txt', 'r') as fo:
    out = fo.read()
    print(out)
```

现在假设我们想从一个文件读取输入并将输出写入另一个文件。
下面是如何使用 `with` 语句正确地获取和归还资源给操作系统来完成此任务的方法：

```{code-cell} python3
with open("newfile.txt", "r") as f:
    file = f.readlines()
    with open("output.txt", "w") as fo:
        for i, line in enumerate(file):
            fo.write(f'Line {i}: {line} \n')
```

输出文件将是：

```{code-cell} python3
with open('output.txt', 'r') as fo:
    print(fo.read())
```

我们可以通过将两个 `with` 语句合并为一行来简化上面的示例。

```{code-cell} python3
with open("newfile.txt", "r") as f, open("output2.txt", "w") as fo:
        for i, line in enumerate(f):
            fo.write(f'Line {i}: {line} \n')
```

输出文件将是相同的。

```{code-cell} python3
with open('output2.txt', 'r') as fo:
    print(fo.read())
```

假设我们想继续向现有文件中写入内容，而不是覆盖它。

我们可以将模式切换为 `a`，即追加模式。

```{code-cell} python3
with open('output2.txt', 'a') as fo:
    fo.write('\nThis is the end of the file')
```

```{code-cell} python3
with open('output2.txt', 'r') as fo:
    print(fo.read())
```

```{note}
注意，这里我们只介绍了 `r`、`w` 和 `a` 模式，它们是最常用的模式。
Python 提供了[多种模式](https://www.geeksforgeeks.org/python/reading-writing-text-files-python/)，你可以自行实验。
```

### 路径

```{index} single: Python; Paths
```

注意，如果 `newfile.txt` 不在当前工作目录中，则对 `open()` 的调用将失败。

在这种情况下，你可以将文件移到 pwd，或者指定文件的[完整路径](https://en.wikipedia.org/wiki/Path_%28computing%29)。

```{code-block} python3
:class: no-execute

f = open('insert_full_path_to_file/newfile.txt', 'r')
```

(iterating_version_1)=
## 迭代

```{index} single: Python; Iteration
```

计算中最重要的任务之一是遍历数据序列并执行给定操作。

Python 的优势之一在于其通过 `for` 循环提供的简单、灵活的迭代接口。

### 遍历不同对象

许多 Python 对象是"可迭代的"，即可以被循环遍历。

举个例子，让我们将列出美国城市及其人口的文件 us_cities.txt 写入当前工作目录。

(us_cities_data)=
```{code-cell} ipython
%%writefile us_cities.txt
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

这里 `%%writefile` 是一个 [IPython 单元魔法命令](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cell-magics)。

假设我们想让信息更易读，通过将名称首字母大写并为千位添加逗号分隔符。

下面的程序读取数据并进行转换：

```{code-cell} python3
data_file = open('us_cities.txt', 'r')
for line in data_file:
    city, population = line.split(':')         # 元组解包
    city = city.title()                        # 城市名称首字母大写
    population = f'{int(population):,}'        # 为数字添加逗号
    print(city.ljust(15) + population)
data_file.close()
```

这里 `f'` 是一个 f 字符串，[用于将变量插入字符串](https://docs.python.org/3/library/string.html#formatspec)。

每行的重新格式化是三种不同字符串方法的结果，其细节可以留到以后再说。

这个程序对我们来说有趣的部分是第 2 行，它表明：

1. 文件对象 `data_file` 是可迭代的，即它可以放在 `for` 循环中 `in` 的右侧。
1. 迭代逐行步进文件。

这使得我们的程序具有简洁、方便的语法。

许多其他类型的对象也是可迭代的，我们稍后将讨论其中一些。

### 不使用索引的循环

你可能已经注意到，Python 倾向于不使用显式索引进行循环。

例如，

```{code-cell} python3
x_values = [1, 2, 3]  # 一些可迭代的 x
for x in x_values:
    print(x * x)
```

优于

```{code-cell} python3
for i in range(len(x_values)):
    print(x_values[i] * x_values[i])
```

当你比较这两种替代方案时，可以看出为何首选第一种。

Python 提供了一些工具来简化不使用索引的循环。

其中之一是 `zip()`，用于同步遍历两个序列中的配对元素。

例如，尝试运行以下代码：

```{code-cell} python3
countries = ('Japan', 'Korea', 'China')
cities = ('Tokyo', 'Seoul', 'Beijing')
for country, city in zip(countries, cities):
    print(f'The capital of {country} is {city}')
```

`zip()` 函数也可用于创建字典——例如：

```{code-cell} python3
names = ['Tom', 'John']
marks = ['E', 'F']
dict(zip(names, marks))
```

如果我们确实需要列表中的索引，一种选择是使用 `enumerate()`。

要理解 `enumerate()` 的作用，请考虑以下示例：

```{code-cell} python3
letter_list = ['a', 'b', 'c']
for index, letter in enumerate(letter_list):
    print(f"letter_list[{index}] = '{letter}'")
```

(list_comprehensions)=
### 列表推导式

```{index} single: Python; List comprehension
```

我们也可以通过使用*列表推导式*来大幅简化生成随机抽样列表的代码。

[列表推导式](https://en.wikipedia.org/wiki/List_comprehension)是一种用于创建列表的优雅 Python 工具。

考虑以下示例，其中列表推导式位于第二行的右侧：

```{code-cell} python3
animals = ['dog', 'cat', 'bird']
plurals = [animal + 's' for animal in animals]
plurals
```

这是另一个示例：

```{code-cell} python3
range(8)
```

```{code-cell} python3
doubles = [2 * x for x in range(8)]
doubles
```

## 比较与逻辑运算符

### 比较

```{index} single: Python; Comparison
```

许多不同类型的表达式会求值为布尔值之一（即 `True` 或 `False`）。

一种常见类型是比较，例如：

```{code-cell} python3
x, y = 1, 2
x < y
```

```{code-cell} python3
x > y
```

Python 的一个优点是我们可以*链式*使用不等式：

```{code-cell} python3
1 < 2 < 3
```

```{code-cell} python3
1 <= 2 <= 3
```

如我们之前所见，测试相等性时使用 `==`：

```{code-cell} python3
x = 1    # 赋值
x == 2   # 比较
```

"不等于"使用 `!=`：

```{code-cell} python3
1 != 2
```

注意，在测试条件时，我们可以使用**任何**有效的 Python 表达式：

```{code-cell} python3
x = 'yes' if 42 else 'no'
x
```

```{code-cell} python3
x = 'yes' if [] else 'no'
x
```

这里发生了什么？

规则是：

* 求值为零、空序列或容器（字符串、列表等）以及 `None` 的表达式都等价于 `False`。
    * 例如，`[]` 和 `()` 在 `if` 子句中等价于 `False`。
* 所有其他值等价于 `True`。
    * 例如，`42` 在 `if` 子句中等价于 `True`。

### 组合表达式

```{index} single: Python; Logical Expressions
```

我们可以使用 `and`、`or` 和 `not` 组合表达式。

这些是标准的逻辑连接词（合取、析取和否定）：

```{code-cell} python3
1 < 2 and 'f' in 'foo'
```

```{code-cell} python3
1 < 2 and 'g' in 'foo'
```

```{code-cell} python3
1 < 2 or 'g' in 'foo'
```

```{code-cell} python3
not True
```

```{code-cell} python3
not not True
```

记住：

* `P and Q` 当两者都为 `True` 时为 `True`，否则为 `False`。
* `P or Q` 当两者都为 `False` 时为 `False`，否则为 `True`。

我们还可以使用 `all()` 和 `any()` 来测试一系列表达式：

```{code-cell} python3
all([1 <= 2 <= 3, 5 <= 6 <= 7])
```
```{code-cell} python3
all([1 <= 2 <= 3, "a" in "letter"])
```
```{code-cell} python3
any([1 <= 2 <= 3, "a" in "letter"])
```

```{note}
* `all()` 当序列中*所有*布尔值/表达式均为 `True` 时返回 `True`。
* `any()` 当序列中*任意*布尔值/表达式为 `True` 时返回 `True`。
```

## 编码风格与文档

一致的编码风格和文档的使用可以使代码更易于理解和维护。

### Python 风格指南：PEP8

```{index} single: Python; PEP8
```

你可以通过在提示符处输入 `import this` 来了解 Python 的编程哲学。

除其他事项外，Python 强烈倡导编程风格的一致性。

我们都听过关于一致性和小聪明的说法。

然而在编程中，如同在数学中，情况恰恰相反。

* 一篇数学论文中若将符号 $\cup$ 和 $\cap$ 对调，即使作者在第一页就告知读者，也会让人非常难以阅读。

在 Python 中，标准风格在 [PEP8](https://peps.python.org/pep-0008/) 中有所规定。

（有时我们在这些讲座中会偏离 PEP8，以更好地匹配数学符号。）

(Docstrings)=
### 文档字符串

```{index} single: Python; Docstrings
```

Python 有一个为模块、类、函数等添加注释的系统，称为*文档字符串*。

文档字符串的优点在于它们在运行时可用。

尝试运行以下代码：

```{code-cell} python3
def f(x):
    """
    This function squares its argument
    """
    return x**2
```

运行此代码后，文档字符串即可使用。

```{code-cell} ipython
f?
```

```{code-block} ipython
:class: no-execute

Type:       function
String Form:<function f at 0x2223320>
File:       /home/john/temp/temp.py
Definition: f(x)
Docstring:  This function squares its argument
```

```{code-cell} ipython
f??
```

```{code-block} ipython
:class: no-execute

Type:       function
String Form:<function f at 0x2223320>
File:       /home/john/temp/temp.py
Definition: f(x)
Source:
def f(x):
    """
    This function squares its argument
    """
    return x**2
```

使用一个问号可以调出文档字符串，使用两个问号则可以同时获得源代码。

你可以在 [PEP257](https://peps.python.org/pep-0257/) 中找到文档字符串的约定。

## 练习

完成以下练习。

（对于某些练习，内置函数 `sum()` 会很方便。）

```{exercise-start}
:label: pyess_ex1
```

第 1 部分：给定两个等长的数值列表或元组 `x_vals` 和 `y_vals`，使用 `zip()` 计算它们的内积。

第 2 部分：用一行代码，计算 0,...,99 中偶数的个数。

第 3 部分：给定 `pairs = ((2, 5), (4, 2), (9, 8), (12, 10))`，计算满足 `a` 和 `b` 都是偶数的配对 `(a, b)` 的数量。

```{hint}
:class: dropdown

`x % 2` 若 `x` 为偶数则返回 0，否则返回 1。

```

```{exercise-end}
```


```{solution-start} pyess_ex1
:class: dropdown
```

**第 1 部分解答：**

这是一种可能的解法：

```{code-cell} python3
x_vals = [1, 2, 3]
y_vals = [1, 1, 1]
sum([x * y for x, y in zip(x_vals, y_vals)])
```

这样也行：

```{code-cell} python3
sum(x * y for x, y in zip(x_vals, y_vals))
```

**第 2 部分解答：**

一种解法是：

```{code-cell} python3
sum([x % 2 == 0 for x in range(100)])
```

这样也行：

```{code-cell} python3
sum(x % 2 == 0 for x in range(100))
```

一些不太自然但有助于说明列表推导式灵活性的替代方案是：

```{code-cell} python3
len([x for x in range(100) if x % 2 == 0])
```

以及

```{code-cell} python3
sum([1 for x in range(100) if x % 2 == 0])
```

**第 3 部分解答：**

这是一种可能的解法：

```{code-cell} python3
pairs = ((2, 5), (4, 2), (9, 8), (12, 10))
sum([x % 2 == 0 and y % 2 == 0 for x, y in pairs])
```

```{solution-end}
```

```{exercise-start}
:label: pyess_ex2
```

考虑多项式

```{math}
:label: polynom0

p(x)
= a_0 + a_1 x + a_2 x^2 + \cdots a_n x^n
= \sum_{i=0}^n a_i x^i
```

编写一个函数 `p`，使得 `p(x, coeff)` 在给定点 `x` 和系数列表 `coeff`（$a_1, a_2, \cdots a_n$）的情况下，计算 {eq}`polynom0` 中的值。

尝试在循环中使用 `enumerate()`。

```{exercise-end}
```

```{solution-start} pyess_ex2
:class: dropdown
```

这是一种解法：

```{code-cell} python3
def p(x, coeff):
    return sum(a * x**i for i, a in enumerate(coeff))
```

```{code-cell} python3
p(1, (2, 4))
```

```{solution-end}
```


```{exercise-start}
:label: pyess_ex3
```

编写一个函数，以字符串为参数，返回字符串中大写字母的数量。

```{hint}
:class: dropdown

`'foo'.upper()` 返回 `'FOO'`。

```

```{exercise-end}
```

```{solution-start} pyess_ex3
:class: dropdown
```

这是一种解法：

```{code-cell} python3
def f(string):
    count = 0
    for letter in string:
        if letter == letter.upper() and letter.isalpha():
            count += 1
    return count

f('The Rain in Spain')
```

另一种更具 Python 风格的解法：

```{code-cell} python3
def count_uppercase_chars(s):
    return sum([c.isupper() for c in s])

count_uppercase_chars('The Rain in Spain')
```

```{solution-end}
```



```{exercise}
:label: pyess_ex4

编写一个函数，以两个序列 `seq_a` 和 `seq_b` 为参数，若 `seq_a` 中的每个元素也是 `seq_b` 的元素则返回 `True`，否则返回 `False`。

* "序列"是指列表、元组或字符串。
* 不使用[集合](https://docs.python.org/3/tutorial/datastructures.html#sets)及其方法来完成此练习。
```

```{solution-start} pyess_ex4
:class: dropdown
```

这是一种解法：

```{code-cell} python3
def f(seq_a, seq_b):
    for a in seq_a:
        if a not in seq_b:
            return False
    return True

# == 测试 == #
print(f("ab", "cadb"))
print(f("ab", "cjdb"))
print(f([1, 2], [1, 2, 3]))
print(f([1, 2, 3], [1, 2]))
```

另一种使用 `all()` 的更具 Python 风格的解法：

```{code-cell} python3
def f(seq_a, seq_b):
  return all([i in seq_b for i in seq_a])

# == 测试 == #
print(f("ab", "cadb"))
print(f("ab", "cjdb"))
print(f([1, 2], [1, 2, 3]))
print(f([1, 2, 3], [1, 2]))
```

当然，如果我们使用 `sets` 数据类型，解法会更简单：

```{code-cell} python3
def f(seq_a, seq_b):
    return set(seq_a).issubset(set(seq_b))
```

```{solution-end}
```


```{exercise}
:label: pyess_ex5

当我们介绍数值库时，将会看到它们包含许多用于插值和函数近似的替代方案。

尽管如此，让我们作为练习编写自己的函数近似例程。

特别地，不使用任何导入，编写一个函数 `linapprox`，接受以下参数：

* 一个将某区间 $[a, b]$ 映射到 $\mathbb R$ 的函数 `f`。
* 两个标量 `a` 和 `b`，提供该区间的端点。
* 一个整数 `n`，确定网格点的数量。
* 一个满足 `a <= x <= b` 的数 `x`。

并返回 `f` 在 `x` 处基于 `n` 个均匀间隔网格点 `a = point[0] < point[1] < ... < point[n-1] = b` 的[分段线性插值](https://en.wikipedia.org/wiki/Linear_interpolation)。

以清晰为目标，而非效率。
```

```{solution-start} pyess_ex5
:class: dropdown
```

这是一种解法：

```{code-cell} python3
def linapprox(f, a, b, n, x):
    """
    Evaluates the piecewise linear interpolant of f at x on the interval
    [a, b], with n evenly spaced grid points.

    Parameters
    ==========
        f : function
            The function to approximate

        x, a, b : scalars (floats or integers)
            Evaluation point and endpoints, with a <= x <= b

        n : integer
            Number of grid points

    Returns
    =======
        A float. The interpolant evaluated at x

    """
    length_of_interval = b - a
    num_subintervals = n - 1
    step = length_of_interval / num_subintervals

    # === 找到第一个大于 x 的网格点 === #
    point = a
    while point <= x:
        point += step

    # === x 必须位于网格点 (point - step) 和 point 之间 === #
    u, v = point - step, point

    return f(u) + (x - u) * (f(v) - f(u)) / (v - u)
```

```{solution-end}
```


```{exercise-start}
:label: pyess_ex6
```

使用列表推导式语法，我们可以简化以下代码中的循环。

```{code-cell} python3
import numpy as np

n = 100
ϵ_values = []
for i in range(n):
    e = np.random.randn()
    ϵ_values.append(e)
```

```{exercise-end}
```

```{solution-start} pyess_ex6
:class: dropdown
```

这是一种解法。

```{code-cell} python3
n = 100
ϵ_values = [np.random.randn() for i in range(n)]
```

```{solution-end}
```