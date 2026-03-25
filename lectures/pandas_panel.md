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
  title: Pandas 面板数据
  headings:
    Overview: 概述
    Slicing and Reshaping Data: 数据切片与重塑
    Merging Dataframes and Filling NaNs: 合并数据框与填充 NaN 值
    Grouping and Summarizing Data: 数据分组与汇总
    Final Remarks: 结语
    Exercises: 练习
---

```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

(ppd)=
# {index}`Pandas 面板数据 <single: Pandas for Panel Data>`

```{index} single: Python; Pandas
```

除 Anaconda 中已有的库外，本讲座还需要以下库：

```{code-cell} ipython3
:tags: [hide-output]

!pip install --upgrade seaborn
```

我们使用以下导入语句。

```{code-cell} ipython3
import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n
import seaborn as sns
sns.set_theme()
```

## 概述

在{doc}`早先关于 pandas 的讲座 <pandas>`中，我们研究了如何处理简单的数据集。

计量经济学家通常需要处理更复杂的数据集，例如面板数据。

常见任务包括：

* 导入数据、清洗数据并跨多个轴对其进行重塑。
* 从面板中选择时间序列或横截面。
* 对数据进行分组和汇总。

`pandas`（源自"panel"和"data"）包含强大且易于使用的工具，专门用于解决此类问题。

在下文中，我们将使用经合组织（OECD）的实际最低工资面板数据集来创建：

* 跨多个数据维度的汇总统计
* 数据集中各国平均最低工资的时间序列
* 按大洲划分的工资核密度估计

我们将首先从 CSV 文件中读取长格式面板数据，然后使用 `pivot_table` 对生成的 `DataFrame` 进行重塑，以构建 `MultiIndex`。

我们将使用 pandas 的 `merge` 函数向 `DataFrame` 添加更多细节，并使用 `groupby` 函数对数据进行汇总。

## 数据切片与重塑

我们将读入一个来自 OECD 的数据集，该数据集包含 32 个国家的实际最低工资，并将其赋值给 `realwage`。

可通过以下链接访问该数据集：

```{code-cell} ipython3
url1 = 'https://raw.githubusercontent.com/QuantEcon/lecture-python/master/source/_static/lecture_specific/pandas_panel/realwage.csv'
```

```{code-cell} ipython3
import pandas as pd

# 出于显示目的，设置最多显示 6 列
pd.set_option('display.max_columns', 6)

# 将小数位数减少到 2 位
pd.options.display.float_format = '{:,.2f}'.format

realwage = pd.read_csv(url1)
```

让我们看看数据的内容

```{code-cell} ipython3
realwage.head()  # 显示前 5 行
```

数据目前为长格式，当数据有多个维度时，这种格式难以分析。

我们将使用 `pivot_table` 创建宽格式面板，并使用 `MultiIndex` 来处理高维数据。

`pivot_table` 的参数应指定数据（values）、索引（index）以及结果数据框中所需的列（columns）。

通过在 columns 中传入列表，我们可以在列轴上创建 `MultiIndex`

```{code-cell} ipython3
realwage = realwage.pivot_table(values='value',
                                index='Time',
                                columns=['Country', 'Series', 'Pay period'])
realwage.head()
```

为了便于后续过滤时间序列数据，我们将索引转换为 `DateTimeIndex`

```{code-cell} ipython3
realwage.index = pd.to_datetime(realwage.index)
type(realwage.index)
```

列包含多层索引，称为 `MultiIndex`，层级按层次排列（国家 > 系列 > 支付周期）。

`MultiIndex` 是在 pandas 中管理面板数据最简单、最灵活的方式

```{code-cell} ipython3
type(realwage.columns)
```

```{code-cell} ipython3
realwage.columns.names
```

与之前一样，我们可以选择国家（`MultiIndex` 的顶层）

```{code-cell} ipython3
realwage['United States'].head()
```

在本讲座中，对 `MultiIndex` 层级进行堆叠和展开将被频繁使用，以将数据框重塑为所需格式。

`.stack()` 将列 `MultiIndex` 的最低层级旋转到行索引（`.unstack()` 方向相反——可以尝试一下）

```{code-cell} ipython3
realwage.stack(future_stack=True).head()
```

我们也可以传入参数来选择要堆叠的层级

```{code-cell} ipython3
realwage.stack(level='Country', future_stack=True).head()  # pandas>3.0 之前需要 future_stack=True
```

使用 `DatetimeIndex` 可以方便地选择特定时间段。

选择某一年并堆叠 `MultiIndex` 的两个较低层级，可以创建面板数据的横截面

```{code-cell} ipython3
realwage.loc['2015'].stack(level=(1, 2), future_stack=True).transpose().head() # pandas>3.0 之前需要 future_stack=True
```

在本讲座的其余部分，我们将使用一个包含各国和各时间段每小时实际最低工资的数据框，以 2015 年美元计价。

为了创建过滤后的数据框（`realwage_f`），我们可以使用 `xs` 方法在多级索引的较低层级中选择值，同时保留较高层级（此处为国家）

```{code-cell} ipython3
realwage_f = realwage.xs(('Hourly', 'In 2015 constant prices at 2015 USD exchange rates'),
                         level=('Pay period', 'Series'), axis=1)
realwage_f.head()
```

## 合并数据框与填充 NaN 值

与 SQL 等关系型数据库类似，pandas 内置了合并数据集的方法。

使用来自 [WorldData.info](https://www.worlddata.info/downloads/) 的国家信息，我们将通过 `merge` 函数向 `realwage_f` 添加每个国家所属的大洲。

可通过以下链接访问该数据集：

```{code-cell} ipython3
url2 = 'https://raw.githubusercontent.com/QuantEcon/lecture-python/master/source/_static/lecture_specific/pandas_panel/countries.csv'
```

```{code-cell} ipython3
worlddata = pd.read_csv(url2, sep=';')
worlddata.head()
```

首先，我们从 `worlddata` 中只选取国家和大洲变量，并将列重命名为"Country"

```{code-cell} ipython3
worlddata = worlddata[['Country (en)', 'Continent']]
worlddata = worlddata.rename(columns={'Country (en)': 'Country'})
worlddata.head()
```

我们希望将新的数据框 `worlddata` 与 `realwage_f` 合并。

pandas 的 `merge` 函数允许按行连接数据框。

我们的数据框将通过国家名称进行合并，因此需要对 `realwage_f` 进行转置，使两个数据框中的行均对应国家名称

```{code-cell} ipython3
realwage_f.transpose().head()
```

我们可以使用左连接、右连接、内连接或外连接来合并数据集：

* 左连接只包含左侧数据集中的国家
* 右连接只包含右侧数据集中的国家
* 外连接包含左侧或右侧数据集中的所有国家
* 内连接只包含两个数据集共有的国家

默认情况下，`merge` 使用内连接。

这里我们将传入 `how='left'`，以保留 `realwage_f` 中的所有国家，但丢弃 `worlddata` 中在 `realwage_f` 中没有对应数据条目的国家。

下图中的红色阴影部分说明了这一点

```{figure} /_static/lecture_specific/pandas_panel/venn_diag.png
```

我们还需要指定每个数据框中国家名称的位置，这将是用于合并数据框的 `key`。

我们的"左"数据框（`realwage_f.transpose()`）在索引中包含国家名称，因此设置 `left_index=True`。

我们的"右"数据框（`worlddata`）在"Country"列中包含国家名称，因此设置 `right_on='Country'`

```{code-cell} ipython3
merged = pd.merge(realwage_f.transpose(), worlddata,
                  how='left', left_index=True, right_on='Country')
merged.head()
```

出现在 `realwage_f` 中但未出现在 `worlddata` 中的国家，其 Continent 列将显示 `NaN`。

为检查是否存在这种情况，我们可以对 Continent 列使用 `.isnull()` 并过滤合并后的数据框

```{code-cell} ipython3
merged[merged['Continent'].isnull()]
```

有三个缺失值！

处理 NaN 值的一种方法是创建一个包含这些国家及其对应大洲的字典。

`.map()` 将把 `merged['Country']` 中的国家与字典中的大洲进行匹配。

注意，不在字典中的国家将被映射为 `NaN`

```{code-cell} ipython3
missing_continents = {'Korea': 'Asia',
                      'Russian Federation': 'Europe',
                      'Slovak Republic': 'Europe'}

merged['Country'].map(missing_continents)
```

我们不想用此映射覆盖整个系列。

`.fillna()` 只用映射填充 `merged['Continent']` 中的 `NaN` 值，而保持列中其他值不变

```{code-cell} ipython3
merged['Continent'] = merged['Continent'].fillna(merged['Country'].map(missing_continents))

# 检查大洲是否被正确映射

merged[merged['Country'] == 'Korea']
```

我们还将把美洲合并为单一大洲——这将使后续可视化效果更好。

为此，我们将使用 `.replace()` 并遍历要替换的大洲值列表

```{code-cell} ipython3
replace = ['Central America', 'North America', 'South America']
merged['Continent'] = merged['Continent'].replace(to_replace=replace, value='America')
```

现在我们已将所需的全部数据整合到一个 `DataFrame` 中，我们将使用 `MultiIndex` 将其重塑回面板形式。

我们还应使用 `.sort_index()` 对索引进行排序，以便后续能高效过滤数据框。

默认情况下，层级将从上到下排序

```{code-cell} ipython3
merged = merged.set_index(['Continent', 'Country']).sort_index()
merged.head()
```

合并过程中，由于合并的列不是日期时间格式，我们丢失了 `DatetimeIndex`

```{code-cell} ipython3
merged.columns
```

现在我们已将合并后的列设置为索引，可以使用 `.to_datetime()` 重新创建 `DatetimeIndex`

```{code-cell} ipython3
merged.columns = pd.to_datetime(merged.columns)
merged.columns = merged.columns.rename('Time')
merged.columns
```

`DatetimeIndex` 在行轴中工作更为顺畅，因此我们将对 `merged` 进行转置

```{code-cell} ipython3
merged = merged.transpose()
merged.head()
```

## 数据分组与汇总

对于理解大型面板数据集而言，数据分组和汇总特别有用。

汇总数据的一种简单方法是对数据框调用[聚合方法](https://pandas.pydata.org/pandas-docs/stable/getting_started/intro_tutorials/06_calculate_statistics.html)，例如 `.mean()` 或 `.max()`。

例如，我们可以计算 2006 年至 2016 年间每个国家的平均实际最低工资（默认按行聚合）

```{code-cell} ipython3
merged.mean().head(10)
```

利用此系列，我们可以绘制数据集中每个国家过去十年的平均实际最低工资

```{code-cell} ipython3
merged.mean().sort_values(ascending=False).plot(kind='bar',
                                                title="2006 - 2016 年平均实际最低工资")

# 设置国家标签
country_labels = merged.mean().sort_values(ascending=False).index.get_level_values('Country').tolist()
plt.xticks(range(0, len(country_labels)), country_labels)
plt.xlabel('国家')

plt.show()
```

向 `.mean()` 传入 `axis=1` 将按列聚合（给出所有国家随时间变化的平均最低工资）

```{code-cell} ipython3
merged.mean(axis=1).head()
```

我们可以将此时间序列绘制为折线图

```{code-cell} ipython3
merged.mean(axis=1).plot()
plt.title('2006 - 2016 年平均实际最低工资')
plt.ylabel('2015 年美元')
plt.xlabel('年份')
plt.show()
```

我们还可以指定 `MultiIndex` 的某个层级（在列轴上）进行聚合。

对于 `groupby`，由于 pandas 已弃用在 `groupby` 方法中使用 `axis=1`，我们需要使用 `.T` 将列转置为行。

```{code-cell} ipython3
merged.T.groupby(level='Continent').mean().head()
```

我们可以将各大洲的平均最低工资绘制为时间序列

```{code-cell} ipython3
merged.T.groupby(level='Continent').mean().T.plot()
plt.title('平均实际最低工资')
plt.ylabel('2015 年美元')
plt.xlabel('年份')
plt.show()
```

出于绘图目的，我们将删除澳大利亚这一大洲

```{code-cell} ipython3
merged = merged.drop('Australia', level='Continent', axis=1)
merged.T.groupby(level='Continent').mean().T.plot()
plt.title('平均实际最低工资')
plt.ylabel('2015 年美元')
plt.xlabel('年份')
plt.show()
```

`.describe()` 可用于快速检索多个常用汇总统计量

```{code-cell} ipython3
merged.stack(future_stack=True).describe()
```

这是使用 `groupby` 的一种简化方式。

使用 `groupby` 通常遵循"拆分-应用-合并"流程：

* 拆分：根据一个或多个键对数据进行分组
* 应用：对每个组独立调用函数
* 合并：将函数调用的结果合并到新的数据结构中

`groupby` 方法完成此流程的第一步，创建一个新的 `DataFrameGroupBy` 对象，其中数据被分成若干组。

让我们再次按大洲对 `merged` 进行拆分，这次使用 `groupby` 函数，并将结果对象命名为 `grouped`

```{code-cell} ipython3
grouped = merged.T.groupby(level='Continent')
grouped
```

对对象调用聚合方法会对每个组应用该函数，结果将合并到新的数据结构中。

例如，我们可以使用 `.size()` 返回数据集中每个大洲的国家数量。

在此情况下，新的数据结构是一个 `Series`

```{code-cell} ipython3
grouped.size()
```

通过调用 `.get_group()` 只返回单个组中的国家，我们可以为每个大洲创建 2016 年实际最低工资分布的核密度估计。

`grouped.groups.keys()` 将返回 `groupby` 对象中的键

```{code-cell} ipython3
continents = grouped.groups.keys()

for continent in continents:
    sns.kdeplot(grouped.get_group(continent).T.loc['2015'].unstack(), label=continent, fill=True)

plt.title('2015 年实际最低工资')
plt.xlabel('美元')
plt.legend()
plt.show()
```

## 结语

本讲座介绍了 pandas 的一些高级功能，包括多级索引、合并、分组和绘图。

在面板数据分析中，其他可能有用的工具包括 [xarray](https://docs.xarray.dev/en/stable/)，这是一个将 pandas 扩展到 N 维数据结构的 Python 包。

## 练习

```{exercise-start}
:label: pp_ex1
```

在这些练习中，你将使用来自 [Eurostat](https://ec.europa.eu/eurostat/data/database) 的欧洲按年龄和性别划分的就业率数据集。

可通过以下链接访问该数据集：

```{code-cell} ipython3
url3 = 'https://raw.githubusercontent.com/QuantEcon/lecture-python/master/source/_static/lecture_specific/pandas_panel/employ.csv'
```

读入 CSV 文件会返回长格式的面板数据集。使用 `.pivot_table()` 构建列中带有 `MultiIndex` 的宽格式数据框。

首先探索数据框以及 `MultiIndex` 层级中可用的变量。

编写一个程序，快速返回 `MultiIndex` 中的所有值。

```{exercise-end}
```

```{solution-start} pp_ex1
:class: dropdown
```

```{code-cell} ipython3
employ = pd.read_csv(url3)
employ = employ.pivot_table(values='Value',
                            index=['DATE'],
                            columns=['UNIT','AGE', 'SEX', 'INDIC_EM', 'GEO'])
employ.index = pd.to_datetime(employ.index) # 确保日期为 datetime 格式
employ.head()
```

这是一个大型数据集，因此探索可用的层级和变量非常有用

```{code-cell} ipython3
employ.columns.names
```

层级中的变量可以通过循环快速获取

```{code-cell} ipython3
for name in employ.columns.names:
    print(name, employ.columns.get_level_values(name).unique())
```

```{solution-end}
```

```{exercise-start}
:label: pp_ex2
```

对上述数据框进行过滤，只保留就业占"活跃人口"百分比的数据。

使用 `seaborn` 创建 2015 年按年龄组和性别划分的就业率分组箱线图。

```{hint}
:class: dropdown

`GEO` 包含区域和国家两类地理单元。
```

```{exercise-end}
```

```{solution-start} pp_ex2
:class: dropdown
```

为便于按国家过滤，将 `GEO` 交换到顶层并对 `MultiIndex` 进行排序

```{code-cell} ipython3
employ.columns = employ.columns.swaplevel(0,-1)
employ = employ.sort_index(axis=1)
```

我们需要去掉 `GEO` 中一些非国家的条目。

快速去掉欧盟区域的方法是使用列表推导式，找到 `GEO` 中以"Euro"开头的层级值

```{code-cell} ipython3
geo_list = employ.columns.get_level_values('GEO').unique().tolist()
countries = [x for x in geo_list if not x.startswith('Euro')]
employ = employ[countries]
employ.columns.get_level_values('GEO').unique()
```

从数据框中只选取活跃人口就业百分比

```{code-cell} ipython3
employ_f = employ.xs(('Percentage of total population', 'Active population'),
                     level=('UNIT', 'INDIC_EM'),
                     axis=1)
employ_f.head()
```

在创建分组箱线图之前删除"Total"值

```{code-cell} ipython3
employ_f = employ_f.drop('Total', level='SEX', axis=1)
```

```{code-cell} ipython3
box = employ_f.loc['2015'].unstack().reset_index()
sns.boxplot(x="AGE", y=0, hue="SEX", data=box, palette=("husl"), showfliers=False)
plt.xlabel('')
plt.xticks(rotation=35)
plt.ylabel('占人口百分比（%）')
plt.title('欧洲就业状况（2015 年）')
plt.legend(bbox_to_anchor=(1,0.5))
plt.show()
```

```{solution-end}
```