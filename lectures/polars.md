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
  title: Polars
  headings:
    Overview: 概述
    Series: Series
    DataFrames: DataFrames
    DataFrames::Selecting data: 选择数据
    DataFrames::Filtering by conditions: 按条件过滤
    DataFrames::Column expressions: 列表达式
    DataFrames::Missing values: 缺失值
    DataFrames::Visualization: 可视化
    Lazy evaluation: 惰性求值
    Lazy evaluation::Eager vs lazy: 立即执行与惰性执行
    Lazy evaluation::Query optimization: 查询优化
    Lazy evaluation::Performance comparison: 性能比较
    On-line data sources: 在线数据源
    Exercises: 练习
---

(pl)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# Polars

```{index} single: Python; Polars
```

除了 Anaconda 中已有的库之外，本讲座还需要以下库：

```{code-cell} ipython3
:tags: [hide-output]

!pip install --upgrade polars yfinance
```

## 概述

[Polars](https://pola.rs/) 是一个用 Rust 编写的快速 Python 数据处理库。

由于其性能优势，它作为 {doc}`pandas <pandas>` 的现代替代品已经获得了广泛的关注。

Polars 在设计时充分考虑了性能和内存效率，主要利用了以下技术：

* [Apache Arrow 列式存储格式](https://arrow.apache.org/docs/format/Columnar.html)，实现快速的数据访问
* [惰性求值（lazy evaluation）](https://en.wikipedia.org/wiki/Lazy_evaluation)，用于优化查询执行
* 并行处理，充分利用所有可用的 CPU 核心
* 围绕列表达式构建的富有表现力的 API

```{tip}
*为什么要考虑用 Polars 替代 pandas？*

* **内存**：pandas 通常需要相当于数据集大小 5--10 倍的内存；Polars 只需 2--4 倍
* **速度**：对于许多常见操作，Polars 的速度快 10--100 倍
* **参见**：[Polars TPC-H 基准测试](https://pola.rs/benchmarks/)，可查看最新的性能对比结果
```

在整个讲座中，我们假设已经执行了以下导入语句

```{code-cell} ipython3
import polars as pl
import numpy as np
import matplotlib.pyplot as plt
```

与 {doc}`pandas` 类似，Polars 定义了两种重要的数据类型：`Series` 和 `DataFrame`。

你可以将 `Series` 理解为一列数据，例如某个变量的一组观测值。

`DataFrame` 是一个二维对象，用于存储若干相关联的数据列。

## Series

```{index} single: Polars; Series
```

我们先从 Series 开始。

首先创建一个由四个随机观测值组成的 series

```{code-cell} ipython3
rng = np.random.default_rng()
s = pl.Series(name='daily returns', values=rng.standard_normal(4))
s
```

```{note}
与 {doc}`pandas <pandas>` 的 Series 不同，Polars 的 Series 没有行索引。
Polars 是以列为中心的——数据访问是通过列表达式和布尔掩码来管理的，而不是通过行标签。
更多详情请参阅 [面向 pandas 用户的 Polars 迁移指南](https://docs.pola.rs/user-guide/migration/pandas/)。
```

Polars 的 `Series` 建立在 [Apache Arrow](https://arrow.apache.org/) 数组之上，并支持许多我们熟悉的操作

```{code-cell} ipython3
s * 100
```

绝对值可以通过一个方法来获得

```{code-cell} ipython3
s.abs()
```

我们也可以快速获取汇总统计信息

```{code-cell} ipython3
s.describe()
```

由于 Polars 没有行索引，带标签的数据需要用 `DataFrame` 来表示。

例如，要将股票代码与收益关联起来：

```{code-cell} ipython3
df = pl.DataFrame({
    'company': ['AMZN', 'AAPL', 'MSFT', 'GOOG'],
    'daily returns': rng.standard_normal(4)
})
df
```

我们通过对某一列进行过滤来访问某个值

```{code-cell} ipython3
df.filter(
    pl.col('company') == 'AMZN'
).select('daily returns').item()
```

更新数据同样使用表达式，而不是按索引赋值

```{code-cell} ipython3
df = df.with_columns(
    pl.when(pl.col('company') == 'AMZN')
    .then(0)
    .otherwise(pl.col('daily returns'))
    .alias('daily returns')
)
df
```

我们还可以检查成员关系

```{code-cell} ipython3
'AAPL' in df['company']
```

## DataFrames

```{index} single: Polars; DataFrames
```

`Series` 是单独一列数据，而 `DataFrame` 则包含多个列，每个变量对应一列。

和 {doc}`pandas` 一样，我们用 [Penn World Tables](https://www.rug.nl/ggdc/productivity/pwt/pwt-releases/pwt-7.0) 中的数据来进行操作。

我们使用 `pl.read_csv` 来读取数据

```{code-cell} ipython3
url = ('https://raw.githubusercontent.com/QuantEcon/'
       'lecture-python-programming/main/lectures/_static/'
       'lecture_specific/pandas/data/test_pwt.csv')
df = pl.read_csv(url)
df
```

### 选择数据

我们可以通过切片来选择行，通过列名来选择列

```{code-cell} ipython3
df[2:5]
```

要选择特定的列，可以向 `select` 传入一个名称列表

```{code-cell} ipython3
df.select(['country', 'tcgdp'])
```

这些操作也可以组合使用

```{code-cell} ipython3
df[2:5].select(['country', 'tcgdp'])
```

### 按条件过滤

`filter` 方法接受由 `pl.col` 构建的布尔表达式

```{code-cell} ipython3
df.filter(pl.col('POP') >= 20000)
```

可以使用 `&`（与）和 `|`（或）来组合多个条件

```{code-cell} ipython3
df.filter(
    (pl.col('country').is_in(['Argentina', 'India', 'South Africa'])) &
    (pl.col('POP') > 40000)
)
```

表达式可以涉及跨列的算术运算

```{code-cell} ipython3
df.filter(
    (pl.col('cc') + pl.col('cg') >= 80) & (pl.col('POP') <= 20000)
)
```

选择家庭消费占比最大的国家

```{code-cell} ipython3
df.filter(pl.col('cc') == pl.col('cc').max())
```

### 列表达式

与 pandas 的一个关键区别在于，Polars 使用**列表达式**来进行转换，而不是逐元素调用 `apply`。

下面是一个计算每个数值列最大值的示例

```{code-cell} ipython3
df.select(
    pl.col(['year', 'POP', 'XRAT', 'tcgdp', 'cc', 'cg'])
    .max()
    .name.suffix('_max')
)
```

表达式可以在 `with_columns` 内部使用，用于添加或修改列

```{code-cell} ipython3
df.with_columns(
    (pl.col('XRAT') / 10).alias('XRAT_scaled'),
    pl.col(pl.Float64).round(2)
)
```

条件逻辑使用 `pl.when(...).then(...).otherwise(...)`

```{code-cell} ipython3
df.with_columns(
    pl.when(pl.col('POP') >= 20000)
    .then(pl.col('POP'))
    .otherwise(None)
    .alias('POP_filtered')
).select(['country', 'POP', 'POP_filtered'])
```

```{note}
Polars 提供了 `map_elements` 作为逐行应用任意 Python 函数的应急方案，
但它会绕过经过优化的表达式引擎，因此只要存在原生表达式，就应该避免使用它。
```

### 缺失值

让我们插入一些空值来演示插补技术

```{code-cell} ipython3
df_nulls = df.with_row_index().with_columns(
    pl.when(pl.col('index') == 0)
    .then(None).otherwise(pl.col('XRAT')).alias('XRAT'),
    pl.when(pl.col('index') == 3)
    .then(None).otherwise(pl.col('cc')).alias('cc'),
    pl.when(pl.col('index') == 5)
    .then(None).otherwise(pl.col('tcgdp')).alias('tcgdp'),
    pl.when(pl.col('index') == 6)
    .then(None).otherwise(pl.col('POP')).alias('POP'),
).drop('index')
df_nulls
```

将所有空值填充为零

```{code-cell} ipython3
df_nulls.fill_null(0)
```

或者用列的均值来填充

```{code-cell} ipython3
cols = ['cc', 'tcgdp', 'POP', 'XRAT']
df_nulls.with_columns(
    pl.col(cols).fill_null(pl.col(cols).mean())
)
```

Polars 还支持向前填充（`fill_null(strategy='forward')`）以及插值。

在 scikit-learn 中还有更多[高级插补工具](https://scikit-learn.org/stable/modules/impute.html)可供使用。

### 可视化

让我们构建一个人均 GDP 列，并绘制出来

```{code-cell} ipython3
df = (df
    .select(['country', 'POP', 'tcgdp'])
    .rename({'POP': 'population', 'tcgdp': 'total GDP'})
    .with_columns(
        (pl.col('population') * 1e3).alias('population')
    )
    .with_columns(
        (pl.col('total GDP') * 1e6 / pl.col('population'))
        .alias('GDP percap')
    )
    .sort('GDP percap', descending=True)
)
df
```

我们可以直接提取列用于 matplotlib

```{note}
Polars 还提供了基于 Altair 的内置[绘图 API](https://docs.pola.rs/user-guide/misc/visualization/)
（例如 `df.plot.bar(x=..., y=...)`）。
为了与本系列讲座的其他部分保持一致，我们在这里使用 matplotlib。
```

```{code-cell} ipython3
fig, ax = plt.subplots()
ax.bar(df['country'].to_list(), df['GDP percap'].to_list())
ax.set_xlabel('country', fontsize=12)
ax.set_ylabel('GDP per capita', fontsize=12)
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

## 惰性求值

```{index} single: Polars; Lazy Evaluation
```

Polars 最强大的特性之一是**惰性求值（lazy evaluation）**。

惰性模式并不会立即执行每一个操作，而是先收集完整的查询计划，再在运行前对其进行优化。

### 立即执行与惰性执行

```{code-cell} ipython3
# 重新加载数据集
url = ('https://raw.githubusercontent.com/QuantEcon/'
       'lecture-python-programming/main/lectures/_static/'
       'lecture_specific/pandas/data/test_pwt.csv')
df_full = pl.read_csv(url)
```

**立即执行（eager）** API 会立即执行（就像 pandas 那样）

```{code-cell} ipython3
result_eager = (df_full
    .filter(pl.col('tcgdp') > 1000)
    .select(['country', 'year', 'tcgdp'])
    .sort('tcgdp', descending=True)
)
result_eager.head()
```

**惰性（lazy）** API 则会构建一个查询计划

```{code-cell} ipython3
lazy_query = (df_full.lazy()
    .filter(pl.col('tcgdp') > 1000)
    .select(['country', 'year', 'tcgdp'])
    .sort('tcgdp', descending=True)
)
print(lazy_query.explain())
```

调用 `collect` 来执行该计划

```{code-cell} ipython3
result_lazy = lazy_query.collect()
result_lazy.head()
```

### 查询优化

惰性执行引擎会自动应用若干优化：

* **谓词下推**——过滤操作会被尽可能提前应用
* **投影下推**——只从数据源读取所需的列
* **公共子表达式消除**——重复的计算会被合并

让我们看看 Polars 是如何重写一个多步骤查询的

```{code-cell} ipython3
optimized = (df_full.lazy()
    .select(['country', 'year', 'tcgdp', 'POP'])
    .filter(pl.col('tcgdp') > 500)
    .with_columns(
        (pl.col('tcgdp') / pl.col('POP')).alias('gdp_per_capita')
    )
    .filter(pl.col('gdp_per_capita') > 10)
    .select(['country', 'year', 'gdp_per_capita'])
)

print("Optimized plan:")
print(optimized.explain())
```

执行该计划便可得到最终结果

```{code-cell} ipython3
optimized.collect()
```

### 性能比较

让我们在同一任务上比较 pandas、Polars 立即执行模式和 Polars 惰性执行模式。

我们先从一个小数据集（上面使用过的 Penn World Tables 数据）开始，说明对于小数据而言，各方案之间的差异微不足道

```{code-cell} ipython3
import pandas as pd
import time

# 小数据集 -- Penn World Tables（约 8 行）
url = ('https://raw.githubusercontent.com/QuantEcon/'
       'lecture-python-programming/main/lectures/_static/'
       'lecture_specific/pandas/data/test_pwt.csv')
small_pd = pd.read_csv(url)
small_pl = pl.read_csv(url)
```

现在我们对每个库中相同的过滤-选择-排序操作进行计时

```{code-cell} ipython3
# pandas
start = time.perf_counter()
_ = (small_pd
     .query('tcgdp > 500')
     [['country', 'year', 'tcgdp', 'POP']]
     .assign(gdp_pc=lambda d: d['tcgdp'] / d['POP'])
     .sort_values('gdp_pc', ascending=False))
pd_small = time.perf_counter() - start

# Polars 立即执行模式
start = time.perf_counter()
_ = (small_pl
     .filter(pl.col('tcgdp') > 500)
     .select(['country', 'year', 'tcgdp', 'POP'])
     .with_columns((pl.col('tcgdp') / pl.col('POP')).alias('gdp_pc'))
     .sort('gdp_pc', descending=True))
pl_small = time.perf_counter() - start

print(f"Small data  --  pandas: {pd_small:.4f}s | Polars eager: {pl_small:.4f}s")
```

对于寥寥数行的数据，速度差异并不重要——你可以选择自己更习惯使用的 API。

现在让我们把数据规模扩大到 500 万行，此时差异就会变得明显。

我们的任务是：过滤出 `value > 0` 的行，计算加权乘积
`value * weight`，然后对每个分组内该乘积取均值——
也就是一个分组加权平均。

```{code-cell} ipython3
n = 5_000_000
rng = np.random.default_rng(42)

groups = rng.choice(['A', 'B', 'C', 'D'], n)
values = rng.standard_normal(n)
weights = rng.random(n)
extra1 = rng.standard_normal(n)
extra2 = rng.standard_normal(n)

big_pd = pd.DataFrame({
    'group': groups, 'value': values,
    'weight': weights, 'extra1': extra1, 'extra2': extra2
})
big_pl = pl.DataFrame({
    'group': groups, 'value': values,
    'weight': weights, 'extra1': extra1, 'extra2': extra2
})
```

首先是 pandas 基准测试

```{code-cell} ipython3
start = time.perf_counter()
tmp = big_pd[big_pd['value'] > 0][['group', 'value', 'weight']].copy()
tmp['weighted'] = tmp['value'] * tmp['weight']
_ = tmp.groupby('group')['weighted'].mean()
pd_time = time.perf_counter() - start
print(f"pandas:       {pd_time:.4f}s")
```

接下来是 Polars 的立即执行模式

```{code-cell} ipython3
start = time.perf_counter()
_ = (big_pl
    .filter(pl.col('value') > 0)
    .select(['group', 'value', 'weight'])
    .with_columns(
        (pl.col('value') * pl.col('weight')).alias('weighted'))
    .group_by('group')
    .agg(pl.col('weighted').mean()))
eager_time = time.perf_counter() - start
print(f"Polars eager: {eager_time:.4f}s")
```

最后是 Polars 的惰性执行模式

```{code-cell} ipython3
start = time.perf_counter()
_ = (big_pl.lazy()
    .filter(pl.col('value') > 0)
    .select(['group', 'value', 'weight'])
    .with_columns(
        (pl.col('value') * pl.col('weight')).alias('weighted'))
    .group_by('group')
    .agg(pl.col('weighted').mean())
    .collect())
lazy_time = time.perf_counter() - start
print(f"Polars lazy:  {lazy_time:.4f}s")
```

由此可以得出以下结论：

* 对于**小数据**（数千行），pandas 与 Polars 的表现相似——
  可以根据 API 偏好和生态系统兼容性来选择。
* 对于**中大型数据**（数十万行及以上），得益于其 Rust 引擎、
  并行执行以及（在惰性模式下的）查询优化，Polars 可以显著更快。

当从磁盘读取数据时，惰性 API 尤其强大——`scan_csv` 会直接返回一个 `LazyFrame`，因此过滤和投影操作会被下推到文件读取器中执行。

```{tip}
在处理大型 CSV 文件时，使用 `pl.scan_csv(path)` 而不是 `pl.read_csv(path)`。
这样只有你实际需要的列和行才会从磁盘中被读取。
参见 [Polars I/O 文档](https://docs.pola.rs/user-guide/io/csv/)。
```

## 在线数据源

```{index} single: Data Sources
```

与 {doc}`pandas` 一样，Python 可以非常方便地查询在线数据库。

对经济学家而言，一个重要的数据库是 [FRED](https://fred.stlouisfed.org/)——由圣路易斯联储维护的庞大时间序列数据集。

Polars 的 `read_csv` 可以直接从 URL 获取数据。

我们使用 `try_parse_dates=True` 来自动解析日期列

```{code-cell} ipython3
fred_url = ('https://fred.stlouisfed.org/graph/fredgraph.csv?'
            'bgcolor=%23e1e9f0&chart_type=line&drp=0&'
            'fo=open%20sans&graph_bgcolor=%23ffffff&'
            'height=450&mode=fred&recession_bars=on&'
            'txtcolor=%23444444&ts=12&tts=12&width=1318&'
            'nt=0&thu=0&trc=0&show_legend=yes&'
            'show_axis_titles=yes&show_tooltip=yes&'
            'id=UNRATE&scale=left&cosd=1948-01-01&'
            'coed=2024-06-01&line_color=%234572a7&'
            'link_values=false&line_style=solid&'
            'mark_type=none&mw=3&lw=2&ost=-99999&'
            'oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&'
            'fgst=lin&fgsnd=2020-02-01&line_index=1&'
            'transformation=lin&vintage_date=2024-07-29&'
            'revision_date=2024-07-29&nd=1948-01-01')
data = pl.read_csv(fred_url, try_parse_dates=True)
```

让我们查看前几行

```{code-cell} ipython3
data.head()
```

以及获取汇总统计信息

```{code-cell} ipython3
data.describe()
```

绘制 2006 年到 2012 年的失业率

```{code-cell} ipython3
filtered = data.filter(
    (pl.col('observation_date') >= pl.date(2006, 1, 1)) &
    (pl.col('observation_date') <= pl.date(2012, 12, 31))
)

fig, ax = plt.subplots()
ax.plot(filtered['observation_date'].to_list(),
        filtered['UNRATE'].to_list())
ax.set_title('US Unemployment Rate')
ax.set_xlabel('year', fontsize=12)
ax.set_ylabel('%', fontsize=12)
plt.show()
```

Polars 支持[多种文件格式](https://docs.pola.rs/user-guide/io/)，包括 Excel、JSON、Parquet，以及直接连接数据库。

## 练习

```{exercise-start}
:label: pl_ex1
```

使用以下导入：

```{code-cell} ipython3
import datetime as dt
import yfinance as yf
```

编写一个程序，计算以下几只股票在 2021 年间价格的百分比变化：

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

下面是一个将收盘价读入 Polars DataFrame 的函数：

```{code-cell} ipython3
def read_data_polars(ticker_list,
                     start=dt.datetime(2021, 1, 1),
                     end=dt.datetime(2021, 12, 31)):
    """
    Read closing price data from Yahoo Finance
    and return a Polars DataFrame.
    """
    dataframes = []

    for tick in ticker_list:
        stock = yf.Ticker(tick)
        prices = stock.history(start=start, end=end)
        df = pl.DataFrame({
            'Date': list(prices.index.date),
            tick: prices['Close'].values
        }).with_columns(pl.col('Date').cast(pl.Date))
        dataframes.append(df)

    result = dataframes[0]
    for df in dataframes[1:]:
        result = result.join(
            df, on='Date', how='full', coalesce=True
        )
    return result.sort('Date')

ticker = read_data_polars(ticker_list)
```

```{note}
Polars 的连接（join）操作不保证输出行的顺序——
只匹配一侧的键值会被追加，而不是被插入到相应位置。
这和前面提到的"没有索引、没有自动对齐"的主题是一致的：
由于没有行标签可供对齐，排序需要我们显式地去请求。
因此在返回结果之前会调用 `sort('Date')`，
后续任何 `first()`/`last()` 计算都依赖于这一排序结果。
```

请补充完整该程序，将结果绘制为柱状图。

```{exercise-end}
```

```{solution-start} pl_ex1
:class: dropdown
```

使用 Polars 表达式计算百分比变化：

```{code-cell} ipython3
price_change = ticker.select([
    ((pl.col(tick).drop_nulls().last() / pl.col(tick).drop_nulls().first() - 1) * 100)
    .alias(tick)
    for tick in ticker_list.keys()
]).transpose(
    include_header=True,
    header_name='ticker',
    column_names=['pct_change']
).with_columns(
    pl.col('ticker')
    .replace_strict(ticker_list, default=pl.col('ticker'))
    .alias('company')
).sort('pct_change')

print(price_change)
```

直接使用 matplotlib 绘制结果：

```{code-cell} ipython3
companies = price_change['company'].to_list()
changes = price_change['pct_change'].to_list()
colors = ['red' if x < 0 else 'blue' for x in changes]

fig, ax = plt.subplots(figsize=(10, 8))
ax.bar(companies, changes, color=colors)
ax.set_xlabel('stock', fontsize=12)
ax.set_ylabel('percentage change in price', fontsize=12)
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

```{solution-end}
```


```{exercise-start}
:label: pl_ex2
```

使用 {ref}`pl_ex1` 中的 `read_data_polars`，求出以下指数的同比百分比变化：

```{code-cell} ipython3
indices_list = {'^GSPC': 'S&P 500',
               '^IXIC': 'NASDAQ',
               '^DJI': 'Dow Jones',
               '^N225': 'Nikkei'}
```

将结果绘制为时间序列图。

```{exercise-end}
```

```{solution-start} pl_ex2
:class: dropdown
```

```{code-cell} ipython3
indices_data = read_data_polars(
    indices_list,
    start=dt.datetime(1971, 1, 1),
    end=dt.datetime(2021, 12, 31)
)

indices_data = indices_data.with_columns(
    pl.col('Date').dt.year().alias('year')
)
```

使用分组操作计算逐年收益：

```{code-cell} ipython3
yearly_returns = indices_data.group_by('year').agg([
    *[pl.col(idx).drop_nulls().first().alias(f'{idx}_first')
      for idx in indices_list],
    *[pl.col(idx).drop_nulls().last().alias(f'{idx}_last')
      for idx in indices_list]
])

for idx, name in indices_list.items():
    yearly_returns = yearly_returns.with_columns(
        ((pl.col(f'{idx}_last') - pl.col(f'{idx}_first'))
         / pl.col(f'{idx}_first') * 100).alias(name)
    )

yearly_returns = (yearly_returns
    .select(['year', *indices_list.values()])
    .sort('year')
)
print(yearly_returns)
```

汇总统计信息：

```{code-cell} ipython3
yearly_returns.select(list(indices_list.values())).describe()
```

将每个指数绘制在一个子图中：

```{code-cell} ipython3
fig, axes = plt.subplots(2, 2, figsize=(12, 10))
years = yearly_returns['year'].to_list()

for iter_, ax in enumerate(axes.flatten()):
    name = list(indices_list.values())[iter_]
    values = yearly_returns[name].to_list()
    ax.plot(years, values, 'o-', linewidth=2, markersize=4)
    ax.axhline(y=0, color='k', linestyle='--', alpha=0.3)
    ax.set_ylabel('yearly return (%)', fontsize=12)
    ax.set_xlabel('year', fontsize=12)
    ax.set_title(name, fontsize=12)

plt.tight_layout()
plt.show()
```

```{solution-end}
```
