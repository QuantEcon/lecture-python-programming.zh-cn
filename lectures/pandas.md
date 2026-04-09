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
  title: Pandas
  headings:
    Overview: 概述
    Series: Series
    DataFrames: DataFrames
    DataFrames::Select Data by Position: 按位置选择数据
    DataFrames::Select Data by Conditions: 按条件选择数据
    DataFrames::Apply Method: Apply 方法
    DataFrames::Make Changes in DataFrames: 修改 DataFrames
    DataFrames::Standardization and Visualization: 标准化与可视化
    On-Line Data Sources: 在线数据来源
    On-Line Data Sources::Accessing Data with requests: 使用 requests 访问数据
    On-Line Data Sources::Using wbgapi and yfinance to Access Data: 使用 wbgapi 和 yfinance 访问数据
    Exercises: 练习
---

(pd)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`Pandas <single: Pandas>`

```{index} single: Python; Pandas
```

除了 Anaconda 中已有的内容之外，本讲座还需要以下库：

```{code-cell} ipython3
:tags: [hide-output]

!pip install --upgrade wbgapi
!pip install --upgrade yfinance
```

## 概述

[Pandas](https://pandas.pydata.org/) 是一个用于 Python 的快速、高效的数据分析工具包。

近年来，随着数据科学和机器学习等领域的兴起，其受欢迎程度急剧上升。

以下是来自 Stack Overflow Trends 的与 Matlab 和 STATA 的历史受欢迎程度比较：

```{figure} /_static/lecture_specific/pandas/pandas_vs_rest.png
:scale: 100
```

正如 [NumPy](https://numpy.org/) 提供基本的数组数据类型和核心数组操作一样，pandas

1. 定义了用于处理数据的基础结构，并且
1. 赋予了它们便于执行以下操作的方法：
    * 读入数据
    * 调整索引
    * 处理日期和时间序列
    * 排序、分组、重新排序和一般数据整理 [^mung]
    * 处理缺失值，等等

更复杂的统计功能留给其他包，如构建在 pandas 之上的 [statsmodels](https://www.statsmodels.org/) 和 [scikit-learn](https://scikit-learn.org/)。

本讲座将提供 pandas 的基本介绍。

在整个讲座中，我们将假设已进行了以下导入：

```{code-cell} ipython3
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
import requests
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n
```

pandas 定义的两种重要数据类型是 `Series` 和 `DataFrame`。

你可以将 `Series` 视为数据的"一列"，例如对单个变量的一组观测值。

`DataFrame` 是一个二维对象，用于存储相关的数据列。

## Series

```{index} single: Pandas; Series
```

让我们从 Series 开始。

我们首先创建一个包含四个随机观测值的 Series：

```{code-cell} ipython3
s = pd.Series(np.random.randn(4), name='每日收益')
s
```

在这里，你可以将索引 `0, 1, 2, 3` 想象为四家上市公司的编号，而值则是它们股票的每日收益。

pandas 的 `Series` 建立在 NumPy 数组之上，支持许多类似的操作：

```{code-cell} ipython3
s * 100
```

```{code-cell} ipython3
np.abs(s)
```

但 `Series` 提供的功能比 NumPy 数组更多。

它们不仅有一些额外的（面向统计的）方法：

```{code-cell} ipython3
s.describe()
```

而且它们的索引更加灵活：

```{code-cell} ipython3
s.index = ['AMZN', 'AAPL', 'MSFT', 'GOOG']
s
```

从这个角度来看，`Series` 就像快速、高效的 Python 字典（限制是字典中的所有项目具有相同的类型——在本例中为浮点数）。

事实上，你可以使用与 Python 字典相同的许多语法：

```{code-cell} ipython3
s['AMZN']
```

```{code-cell} ipython3
s['AMZN'] = 0
s
```

```{code-cell} ipython3
'AAPL' in s
```

## DataFrames

```{index} single: Pandas; DataFrames
```

`Series` 是单列数据，而 `DataFrame` 是多列数据，每个变量对应一列。

本质上，pandas 中的 `DataFrame` 类似于（高度优化的）Excel 电子表格。

因此，它是表示和分析自然组织成行和列的数据的强大工具，通常具有用于单个行和单个列的描述性索引。

让我们看一个从 CSV 文件 `pandas/data/test_pwt.csv` 读取数据的示例，该文件取自 [Penn World Tables](https://www.rug.nl/ggdc/productivity/pwt/pwt-releases/pwt-7.0)。

该数据集包含以下指标：

| 变量名 | 描述 |
| :-: | :-: |
| POP | 人口（千人） |
| XRAT | 对美元汇率 |                     
| tcgdp | PPP 转换后的总国内生产总值（百万国际元） |
| cc | PPP 转换后人均国内生产总值的消费份额（%） |
| cg | PPP 转换后人均国内生产总值的政府消费份额（%） |


我们将使用 `pandas` 函数 `read_csv` 从 URL 读取数据。

```{code-cell} ipython3
df = pd.read_csv('https://raw.githubusercontent.com/QuantEcon/lecture-python-programming/main/lectures/_static/lecture_specific/pandas/data/test_pwt.csv')
type(df)
```

以下是 `test_pwt.csv` 的内容：

```{code-cell} ipython3
df
```

### 按位置选择数据

在实践中，我们经常需要查找、选择并处理我们感兴趣的数据子集。

我们可以使用标准 Python 数组切片符号选择特定行：

```{code-cell} ipython3
df[2:5]
```

要选择列，我们可以传递一个包含所需列名（以字符串表示）的列表：

```{code-cell} ipython3
df[['country', 'tcgdp']]
```

要使用整数选择行和列，应使用 `iloc` 属性，格式为 `.iloc[行, 列]`：

```{code-cell} ipython3
df.iloc[2:5, 0:4]
```

要使用整数和标签的混合选择行和列，可以以类似方式使用 `loc` 属性：

```{code-cell} ipython3
df.loc[df.index[2:5], ['country', 'tcgdp']]
```

### 按条件选择数据

除了使用整数和名称索引行和列外，我们还可以获取满足某些（可能较复杂）条件的数据子框架。

本节演示各种实现方法。

最直接的方式是使用 `[]` 运算符：

```{code-cell} ipython3
df[df.POP >= 20000]
```

要理解这里发生了什么，注意 `df.POP >= 20000` 返回一系列布尔值：

```{code-cell} ipython3
df.POP >= 20000
```

在这种情况下，`df[___]` 接受一系列布尔值，只返回值为 `True` 的行。

再看一个示例：

```{code-cell} ipython3
df[(df.country.isin(['Argentina', 'India', 'South Africa'])) & (df.POP > 40000)]
```

然而，还有另一种方法可以做到同样的事情，对于大型数据框，它可能稍快一些，且语法更自然：

```{code-cell} ipython3
# 上面的代码等价于
df.query("POP >= 20000")
```

```{code-cell} ipython3
df.query("country in ['Argentina', 'India', 'South Africa'] and POP > 40000")
```

我们还可以允许不同列之间的算术运算：

```{code-cell} ipython3
df[(df.cc + df.cg >= 80) & (df.POP <= 20000)]
```

```{code-cell} ipython3
# 上面的代码等价于
df.query("cc + cg >= 80 & POP <= 20000")
```

例如，我们可以使用条件来选择家庭消费占 GDP 份额 `cc` 最大的国家：

```{code-cell} ipython3
df.loc[df.cc == max(df.cc)]
```

当我们只想查看选定子数据框的某些列时，可以将上述条件与 `.loc[__ , __]` 命令结合使用。

第一个参数接受条件，第二个参数接受我们想要返回的列的列表：

```{code-cell} ipython3
df.loc[(df.cc + df.cg >= 80) & (df.POP <= 20000), ['country', 'year', 'POP']]
```

**应用：对数据框进行子集化**

现实世界的数据集可能[非常庞大](https://developers.google.com/machine-learning/crash-course/overfitting)。

有时需要使用数据的子集来提高计算效率并减少冗余。

假设我们只对人口（`POP`）和总 GDP（`tcgdp`）感兴趣。

将数据框 `df` 缩减为仅包含这些变量的一种方法是使用上述选择方法覆盖数据框：

```{code-cell} ipython3
df_subset = df[['country', 'POP', 'tcgdp']]
df_subset
```

然后我们可以保存较小的数据集以供进一步分析：

```{code-block} python3
:class: no-execute

df_subset.to_csv('pwt_subset.csv', index=False)
```

### Apply 方法

另一个广泛使用的 pandas 方法是 `df.apply()`。

它将函数应用于每行/列并返回一个 Series。

这个函数可以是一些内置函数（如 `max` 函数）、`lambda` 函数或用户自定义函数。

以下是使用 `max` 函数的示例：

```{code-cell} ipython3
df[['year', 'POP', 'XRAT', 'tcgdp', 'cc', 'cg']].apply(max)
```

这行代码将 `max` 函数应用于所有选定的列。

`lambda` 函数通常与 `df.apply()` 方法一起使用。

一个简单的示例是为数据框中的每一行返回其自身：

```{code-cell} ipython3
df.apply(lambda row: row, axis=1)
```

```{note}
对于 `.apply()` 方法：
- axis = 0 -- 将函数应用于每列（变量）
- axis = 1 -- 将函数应用于每行（观测值）
- axis = 0 是默认参数
```

我们可以将它与 `.loc[]` 一起使用来进行一些更高级的选择：

```{code-cell} ipython3
complexCondition = df.apply(
    lambda row: row.POP > 40000 if row.country in ['Argentina', 'India', 'South Africa'] else row.POP < 20000, 
    axis=1), ['country', 'year', 'POP', 'XRAT', 'tcgdp']
```

这里的 `df.apply()` 返回一系列布尔值，表示满足 if-else 语句中指定条件的行。

此外，它还定义了感兴趣的变量子集：

```{code-cell} ipython3
complexCondition
```

当我们将此条件应用于数据框时，结果为：

```{code-cell} ipython3
df.loc[complexCondition]
```

### 修改 DataFrames

修改数据框的能力对于生成用于未来分析的干净数据集非常重要。

**1.** 我们可以方便地使用 `df.where()` 来"保留"我们已选择的行，并用任何其他值替换其余行：

```{code-cell} ipython3
df.where(df.POP >= 20000, False)
```

**2.** 我们可以简单地使用 `.loc[]` 来指定我们想要修改的列，并赋值：

```{code-cell} ipython3
df.loc[df.cg == max(df.cg), 'cg'] = np.nan
df
```

**3.** 我们可以使用 `.apply()` 方法*整体修改行/列*：

```{code-cell} ipython3
def update_row(row):
    # 修改 POP
    row.POP = np.nan if row.POP<= 10000 else row.POP

    # 修改 XRAT
    row.XRAT = row.XRAT / 10
    return row

df.apply(update_row, axis=1)
```

**4.** 我们可以使用 `.map()` 方法一次修改数据框中所有*单个条目*：

```{code-cell} ipython3
# 将所有小数四舍五入到2位小数
df.map(lambda x : round(x,2) if type(x)!=str else x)
```

**应用：缺失值插补**

替换缺失值是数据整理的重要步骤。

让我们随机插入一些 NaN 值：

```{code-cell} ipython3
for idx in list(zip([0, 3, 5, 6], [3, 4, 6, 2])):
    df.iloc[idx] = np.nan

df
```

这里的 `zip()` 函数从两个列表中创建值对（即 [0,3]、[3,4] 等）。

我们可以再次使用 `.map()` 方法将所有缺失值替换为 0：

```{code-cell} ipython3
# 将所有NaN值替换为0
def replace_nan(x):
    if type(x)!=str:
        return  0 if np.isnan(x) else x
    else:
        return x

df.map(replace_nan)
```

pandas 还为我们提供了便捷的方法来替换缺失值。

例如，使用变量均值进行单一插补在 pandas 中可以很容易地完成：

```{code-cell} ipython3
df = df.fillna(df.iloc[:,2:8].mean())
df
```

缺失值插补是数据科学中的一个大领域，涉及各种机器学习技术。

Python 中还有更多[高级工具](https://scikit-learn.org/stable/modules/impute.html)可用于插补缺失值。

### 标准化与可视化

假设我们只对人口（`POP`）和总 GDP（`tcgdp`）感兴趣。

将数据框 `df` 缩减为仅包含这些变量的一种方法是使用上述选择方法覆盖数据框：

```{code-cell} ipython3
df = df[['country', 'POP', 'tcgdp']]
df
```

这里索引 `0, 1,..., 7` 是多余的，因为我们可以使用国家名称作为索引。

为此，我们将索引设置为数据框中的 `country` 变量：

```{code-cell} ipython3
df = df.set_index('country')
df
```

让我们给列取更好的名称：

```{code-cell} ipython3
df.columns = 'population', 'total GDP'
df
```

`population` 变量以千为单位，让我们恢复为单个单位：

```{code-cell} ipython3
df['population'] = df['population'] * 1e3
df
```

接下来，我们将添加一列显示人均实际 GDP，乘以 1,000,000，因为总 GDP 以百万为单位：

```{code-cell} ipython3
df['GDP percap'] = df['total GDP'] * 1e6 / df['population']
df
```

pandas `DataFrame` 和 `Series` 对象的一个优点是它们具有通过 Matplotlib 进行绘图和可视化的方法。

例如，我们可以轻松地生成人均 GDP 的条形图：

```{code-cell} ipython3
ax = df['GDP percap'].plot(kind='bar')
ax.set_xlabel('国家', fontsize=12)
ax.set_ylabel('人均GDP', fontsize=12)
plt.show()
```

目前数据框按国家字母顺序排列——让我们改为按人均 GDP 排列：

```{code-cell} ipython3
df = df.sort_values(by='GDP percap', ascending=False)
df
```

与之前一样进行绘图现在会产生：

```{code-cell} ipython3
ax = df['GDP percap'].plot(kind='bar')
ax.set_xlabel('国家', fontsize=12)
ax.set_ylabel('人均GDP', fontsize=12)
plt.show()
```

## 在线数据来源

```{index} single: Data Sources
```

Python 使得以编程方式查询在线数据库变得简单。

对于经济学家来说，一个重要的数据库是 [FRED](https://fred.stlouisfed.org/) —— 由圣路易斯联储维护的大量时间序列数据集合。

例如，假设我们对[失业率](https://fred.stlouisfed.org/series/UNRATE)感兴趣。

（要将数据下载为 csv，点击右上角的 `Download` 并选择 `CSV (data)` 选项。）

或者，我们可以在 Python 程序中访问 CSV 文件。

这可以通过多种方法完成。

我们从一个相对低级的方法开始，然后回到 pandas。

### 使用 {index}`requests <single: requests>` 访问数据

```{index} single: Python; requests
```

一个选择是使用 [requests](https://requests.readthedocs.io/en/latest/)，这是一个用于通过互联网请求数据的标准 Python 库。

首先，在你的计算机上尝试以下代码：

```{code-cell} ipython3
r = requests.get('https://fred.stlouisfed.org/graph/fredgraph.csv?bgcolor=%23e1e9f0&chart_type=line&drp=0&fo=open%20sans&graph_bgcolor=%23ffffff&height=450&mode=fred&recession_bars=on&txtcolor=%23444444&ts=12&tts=12&width=1318&nt=0&thu=0&trc=0&show_legend=yes&show_axis_titles=yes&show_tooltip=yes&id=UNRATE&scale=left&cosd=1948-01-01&coed=2024-06-01&line_color=%234572a7&link_values=false&line_style=solid&mark_type=none&mw=3&lw=2&ost=-99999&oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&fgst=lin&fgsnd=2020-02-01&line_index=1&transformation=lin&vintage_date=2024-07-29&revision_date=2024-07-29&nd=1948-01-01')
```

如果没有错误消息，则调用成功。

如果你确实收到了错误，那么有两种可能的原因：

1. 你没有连接到互联网——希望这不是问题所在。
1. 你的机器通过代理服务器访问互联网，而 Python 不知道这一点。

在第二种情况下，你可以：

* 切换到另一台机器
* 通过阅读[文档](https://requests.readthedocs.io/en/latest/)解决代理问题

假设一切正常，你现在可以使用调用 `requests.get('https://research.stlouisfed.org/fred2/series/UNRATE/downloaddata/UNRATE.csv')` 返回的 `source` 对象继续操作：

```{code-cell} ipython3
url = 'https://fred.stlouisfed.org/graph/fredgraph.csv?bgcolor=%23e1e9f0&chart_type=line&drp=0&fo=open%20sans&graph_bgcolor=%23ffffff&height=450&mode=fred&recession_bars=on&txtcolor=%23444444&ts=12&tts=12&width=1318&nt=0&thu=0&trc=0&show_legend=yes&show_axis_titles=yes&show_tooltip=yes&id=UNRATE&scale=left&cosd=1948-01-01&coed=2024-06-01&line_color=%234572a7&link_values=false&line_style=solid&mark_type=none&mw=3&lw=2&ost=-99999&oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&fgst=lin&fgsnd=2020-02-01&line_index=1&transformation=lin&vintage_date=2024-07-29&revision_date=2024-07-29&nd=1948-01-01'
source = requests.get(url).content.decode().split("\n")
source[0]
```

```{code-cell} ipython3
source[1]
```

```{code-cell} ipython3
source[2]
```

我们现在可以编写一些额外的代码来解析这个文本并将其存储为数组。

但这是不必要的——pandas 的 `read_csv` 函数可以为我们处理这个任务。

我们使用 `parse_dates=True`，这样 pandas 就能识别我们的日期列，从而能够进行简单的日期过滤：

```{code-cell} ipython3
data = pd.read_csv(url, index_col=0, parse_dates=True)
```

数据已被读入一个名为 `data` 的 pandas DataFrame，我们现在可以以通常的方式操作它：

```{code-cell} ipython3
type(data)
```

```{code-cell} ipython3
data.head()  # 一个有用的方法，可以快速查看数据框
```

```{code-cell} ipython3
pd.set_option('display.precision', 1)
data.describe()  # 你的输出可能略有不同
```

我们还可以如下绘制 2006 年至 2012 年的失业率：

```{code-cell} ipython3
ax = data['2006':'2012'].plot(title='美国失业率', legend=False)
ax.set_xlabel('年份', fontsize=12)
ax.set_ylabel('%', fontsize=12)
plt.show()
```

请注意，pandas 提供了许多其他文件类型的替代方案。

pandas 有[多种](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html)顶级方法，我们可以用来读取 excel、json、parquet 文件或直接连接到数据库服务器。

### 使用 {index}`wbgapi <single: wbgapi>` 和 {index}`yfinance <single: yfinance>` 访问数据

[wbgapi](https://pypi.org/project/wbgapi/) Python 库可用于从世界银行发布的众多数据库中获取数据。

```{note}
你可以在这篇[世界银行博客文章](https://blogs.worldbank.org/en/opendata/introducing-wbgapi-new-python-package-accessing-world-bank-data)以及这个[教程](https://github.com/tgherzog/wbgapi/blob/master/examples/wbgapi-quickstart.ipynb)中找到有关 [wbgapi](https://pypi.org/project/wbgapi/) 包的一些有用信息。
```

我们还将在练习中使用 [yfinance](https://pypi.org/project/yfinance/) 从 Yahoo Finance 获取数据。

现在让我们来看一个下载和绘制数据的示例——这次来自世界银行。

世界银行[收集并整理](https://data.worldbank.org/indicator)了大量指标的数据。

例如，[这里](https://data.worldbank.org/indicator/GC.DOD.TOTL.GD.ZS)是一些关于政府债务占 GDP 比率的数据。

下面的代码示例为你获取数据并绘制美国和澳大利亚的时间序列：

```{code-cell} ipython3
import wbgapi as wb
wb.series.info('GC.DOD.TOTL.GD.ZS')
```

```{code-cell} ipython3
govt_debt = wb.data.DataFrame('GC.DOD.TOTL.GD.ZS', economy=['USA','AUS'], time=range(2005,2016))
govt_debt = govt_debt.T    # 将年份从列移到行以便绘图
```

```{code-cell} ipython3
govt_debt.plot(xlabel='年份', ylabel='政府债务（占GDP的百分比）');
```

## 练习

```{exercise-start}
:label: pd_ex1
```

使用以下导入：

```{code-cell} ipython3
import datetime as dt
import yfinance as yf
```

编写一个程序，计算以下股票在 2021 年的价格百分比变化：

```{code-cell} ipython3
ticker_list = {'INTC': 'Intel',
               'MSFT': 'Microsoft',
               'IBM': 'IBM',
               'BHP': 'BHP',
               'TM': 'Toyota',
               'AAPL': 'Apple',
               'AMZN': 'Amazon',
               'C': 'Citigroup',
               'QCOM': 'Qualcomm',
               'KO': 'Coca-Cola',
               'GOOG': 'Google'}
```

以下是程序的第一部分：

```{code-cell} ipython3
def read_data(ticker_list,
          start=dt.datetime(2021, 1, 1),
          end=dt.datetime(2021, 12, 31)):
    """
    此函数从 Yahoo 读取 ticker_list 中每个
    股票代码的收盘价数据。
    """
    ticker = pd.DataFrame()

    for tick in ticker_list:
        stock = yf.Ticker(tick)
        prices = stock.history(start=start, end=end)

        # 将索引改为仅日期
        prices.index = pd.to_datetime(prices.index.date)
        
        closing_prices = prices['Close']
        ticker[tick] = closing_prices

    return ticker

ticker = read_data(ticker_list)
```

完成程序，将结果绘制为如下所示的条形图：

```{image} /_static/lecture_specific/pandas/pandas_share_prices.png
:scale: 80
:align: center
```

```{exercise-end}
```

```{solution-start} pd_ex1
:class: dropdown
```

有几种方法可以使用 pandas 计算百分比变化来解决这个问题。

首先，你可以提取数据并执行如下计算：

```{code-cell} ipython3
p1 = ticker.iloc[0]    # 获取第一组价格作为 Series
p2 = ticker.iloc[-1]   # 获取最后一组价格作为 Series
price_change = (p2 - p1) / p1 * 100
price_change
```

或者，你可以使用内置方法 `pct_change`，并使用 `periods` 参数配置它以执行正确的计算：

```{code-cell} ipython3
change = ticker.pct_change(periods=len(ticker)-1, axis='rows')*100
price_change = change.iloc[-1]
price_change
```

然后绘制图表：

```{code-cell} ipython3
price_change.sort_values(inplace=True)
price_change.rename(index=ticker_list, inplace=True)
```

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(10,8))
ax.set_xlabel('股票', fontsize=12)
ax.set_ylabel('价格变化百分比', fontsize=12)
price_change.plot(kind='bar', ax=ax)
plt.show()
```

```{solution-end}
```


```{exercise-start}
:label: pd_ex2
```

使用 {ref}`pd_ex1` 中介绍的 `read_data` 方法，编写一个程序来获取以下指数的年同比百分比变化：

```{code-cell} ipython3
indices_list = {'^GSPC': 'S&P 500',
               '^IXIC': 'NASDAQ',
               '^DJI': 'Dow Jones',
               '^N225': 'Nikkei'}
```

完成程序，显示汇总统计数据并将结果绘制为如下所示的时间序列图：

```{image} /_static/lecture_specific/pandas/pandas_indices_pctchange.png
:scale: 80
:align: center
```

```{exercise-end}
```

```{solution-start} pd_ex2
:class: dropdown
```

按照你在 {ref}`pd_ex1` 中所做的工作，你可以通过相应地更新开始和结束日期，使用 `read_data` 查询数据：

```{code-cell} ipython3
indices_data = read_data(
        indices_list,
        start=dt.datetime(1971, 1, 1),  # 共同起始日期
        end=dt.datetime(2021, 12, 31)
)
```

然后，提取每年的第一组和最后一组价格作为 DataFrames，并计算年度回报：

```{code-cell} ipython3
yearly_returns = pd.DataFrame()

for index, name in indices_list.items():
    p1 = indices_data.groupby(indices_data.index.year)[index].first()  # 获取第一组回报作为 DataFrame
    p2 = indices_data.groupby(indices_data.index.year)[index].last()   # 获取最后一组回报作为 DataFrame
    returns = (p2 - p1) / p1
    yearly_returns[name] = returns

yearly_returns
```

接下来，你可以使用 `describe` 方法获取汇总统计数据：

```{code-cell} ipython3
yearly_returns.describe()
```

然后，绘制图表：

```{code-cell} ipython3
fig, axes = plt.subplots(2, 2, figsize=(10, 8))

for iter_, ax in enumerate(axes.flatten()):            # 将二维数组展平为一维数组
    index_name = yearly_returns.columns[iter_]         # 每次迭代获取指数名称
    ax.plot(yearly_returns[index_name])                # 绘制每个指数年度回报的百分比变化
    ax.set_ylabel("百分比变化", fontsize = 12)
    ax.set_title(index_name)

plt.tight_layout()
```

```{solution-end}
```

[^mung]: 维基百科将数据整理（munging）定义为将数据从一种原始形式清理为结构化、净化形式的过程。
