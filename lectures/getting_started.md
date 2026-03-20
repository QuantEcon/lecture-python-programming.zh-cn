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
  Python in the Cloud: 云端 Python
  Local Install: 本地安装
  Local Install::The Anaconda Distribution: Anaconda 发行版
  Local Install::Installing Anaconda: 安装 Anaconda
  Local Install::Updating `conda`: 更新 `conda`
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`': '{index}`Jupyter 笔记本 <single: Jupyter Notebooks>`'
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Starting the Jupyter Notebook': 启动 Jupyter 笔记本
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Notebook Basics': 笔记本基础
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Notebook Basics::Running Cells': 运行单元格
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Notebook Basics::Modal Editing': 模式编辑
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Notebook Basics::Inserting Unicode (e.g., Greek Letters)': 插入 Unicode（例如希腊字母）
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Notebook Basics::A Test Program': 测试程序
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Working with the Notebook': 使用笔记本
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Working with the Notebook::Tab Completion': Tab 补全
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Working with the Notebook::On-Line Help': 在线帮助
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Working with the Notebook::Other Content': 其他内容
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Debugging Code': 调试代码
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::Sharing Notebooks': 共享笔记本
  '{index}`Jupyter Notebooks <single: Jupyter Notebooks>`::QuantEcon Notes': QuantEcon Notes
  Installing Libraries: 安装库
  Working with Python Files: 使用 Python 文件
  Working with Python Files::Editing and Execution: 编辑与执行
  'Working with Python Files::Editing and Execution::Option 1: {index}`JupyterLab <single: JupyterLab>`': '选项 1：{index}`JupyterLab <single: JupyterLab>`'
  'Working with Python Files::Editing and Execution::Option 2: Using a Text Editor': 选项 2：使用文本编辑器
  Exercises: 练习
---

(getting_started)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

<!-- TODO: Review this styling -->

<style>
  .auto {
    width: 70%;
    height: auto;
    } 
  .terminal{
    width: 80%;
    height: auto;
  }  
</style>


# 入门指南

```{index} single: Python
```

## 概述

在本讲座中，您将学习如何

1. 在云端使用 Python
1. 在本地搭建 Python 环境
1. 执行简单的 Python 命令
1. 运行示例程序
1. 安装支撑这些讲座的代码库

## 云端 Python

开始使用 Python 编程的最简便方式是在云端运行它。

（即使用已安装 Python 的远程服务器。）

一个既免费又可靠的选择是 [Google Colab](https://colab.research.google.com/)。

Colab 还有提供 GPU 的优势，我们将在更高级的讲座中加以利用。

有关 Google Colab 入门的教程可以通过网络和视频搜索找到。

我们大多数讲座在右上角都包含一个"启动笔记本"按钮（带有播放图标），点击后可连接到 Colab 上的可执行版本。


## 本地安装

如果您有合适的机器并计划进行大量 Python 编程，本地安装是更好的选择。

同时，本地安装比 Colab 等云端选项需要更多工作。

本讲座的其余部分将带您了解与本地安装相关的一些细节。


### Anaconda 发行版

[Python 核心包](https://www.python.org/downloads/) 易于安装，但*不是*您在这些讲座中应该选择的版本。

这些讲座需要完整的科学编程生态系统，而

* 核心安装并不提供这些
* 逐个安装各组件非常繁琐。

因此，对于我们的目的而言，最佳方法是安装一个包含以下内容的 Python 发行版：

1. Python 核心语言 **以及**
1. 最流行的科学库的兼容版本。

最好的此类发行版是 [Anaconda Python](https://www.anaconda.com/)。

Anaconda

* 非常流行
* 跨平台
* 功能全面
* 与[同名的 Nicki Minaj 歌曲](https://www.youtube.com/watch?v=LDZX4ooRsWs) 完全无关

Anaconda 还附带一个包管理系统来组织您的代码库。

**以下所有内容均假设您采纳此建议！**

(install_anaconda)=
### 安装 Anaconda

```{index} single: Python; Anaconda
```

要安装 Anaconda，请[下载](https://www.anaconda.com/download)二进制文件并按照说明操作。

重要事项：

* 请确保安装适合您操作系统的正确版本。
* 如果在安装过程中询问您是否希望将 Anaconda 设为默认 Python 安装，请选择"是"。

### 更新 `conda`

Anaconda 提供了一个名为 `conda` 的工具来管理和升级您的 Anaconda 包。

您应该定期执行的一个 `conda` 命令是更新整个 Anaconda 发行版的命令。

作为练习，请执行以下操作：

1. 打开终端
1. 输入 `conda update conda`

有关 conda 的更多信息，请在终端中输入 conda help。

(ipython_notebook)=
## {index}`Jupyter 笔记本 <single: Jupyter Notebooks>`

```{index} single: Python; IPython
```

```{index} single: IPython
```

```{index} single: Jupyter
```

[Jupyter](https://jupyter.org/) 笔记本是与 Python 及科学库进行交互的众多方式之一。

它们使用*基于浏览器*的 Python 交互界面，具有以下特点：

* 能够编写和执行 Python 命令。
* 在浏览器中格式化输出，包括表格、图形、动画等。
* 可以混入格式化文本和数学表达式。

由于这些特性，Jupyter 现已成为科学计算生态系统中的重要角色。

下图展示了在 Jupyter 笔记本中执行部分代码的示例（借自[此处](https://matplotlib.org/stable/gallery/statistics/hexbin_demo.html)）：

```{figure} /_static/lecture_specific/getting_started/jp_demo.png
:figclass: auto
```

虽然 Jupyter 并非在 Python 中编程的唯一方式，但在以下情况下它非常出色：

* 开始学习 Python 编程
* 测试新想法或与少量代码进行交互
* 使用强大的在线交互环境，如 [Google Colab](https://research.google.com/colaboratory/)
* 与学生或同事共享或协作科学想法

这些讲座专为在 Jupyter 笔记本中执行而设计。

### 启动 Jupyter 笔记本

```{index} single: Jupyter Notebook; Setup
```

安装 Anaconda 后，您可以启动 Jupyter 笔记本。

您可以：

* 在应用程序菜单中搜索 Jupyter，或者
* 打开终端并输入 `jupyter notebook`
    * Windows 用户应将上一行中的"终端"替换为"Anaconda 命令提示符"。

如果使用第二种方式，您将看到类似以下内容：

```{figure} /_static/lecture_specific/getting_started/starting_nb.png
:figclass: terminal
```

输出信息告诉我们笔记本正在 `http://localhost:8888/` 运行。

* `localhost` 是本地计算机的名称
* `8888` 指的是您计算机上的[端口号](https://en.wikipedia.org/wiki/Port_%28computer_networking%29) 8888

因此，Jupyter 内核正在监听本地计算机 8888 端口上的 Python 命令。

希望您的默认浏览器也已打开一个看起来类似以下内容的网页：

```{figure} /_static/lecture_specific/getting_started/nb.png
:figclass: auto
```

您在这里看到的内容称为 Jupyter *仪表板*。

如果查看顶部的 URL，它应该是 `localhost:8888` 或类似的内容，与上方的消息一致。

假设这一切都正常运行，您现在可以点击右上角的 `New`，然后选择 `Python 3` 或类似选项。

以下是我们机器上显示的内容：

```{figure} /_static/lecture_specific/getting_started/nb2.png
:figclass: auto
```

笔记本显示一个*活动单元格*，您可以在其中输入 Python 命令。

### 笔记本基础

```{index} single: Jupyter Notebook; Basics
```

让我们从如何编辑代码和运行简单程序开始。

#### 运行单元格

注意，在上图中，单元格被绿色边框包围。

这意味着单元格处于*编辑模式*。

在此模式下，您输入的任何内容都将出现在带有闪烁光标的单元格中。

当您准备好执行单元格中的代码时，按 `Shift-Enter` 而不是通常的 `Enter`。

```{figure} /_static/lecture_specific/getting_started/nb3.png
:figclass: auto
```

```{note}
也有菜单和按钮选项可用于运行单元格中的代码，您可以通过探索来找到它们。
```

#### 模式编辑

关于 Jupyter 笔记本，下一件需要了解的事情是它使用*模式*编辑系统。

这意味着键盘输入的效果**取决于您所处的模式**。

两种模式分别是：

1. 编辑模式
    * 以一个单元格周围的绿色边框加上闪烁光标为标志
    * 您输入的任何内容都会原样出现在该单元格中

1. 命令模式
    * 绿色边框被蓝色边框替代
    * 按键被解释为命令——例如，输入 `b` 会在当前单元格下方添加新单元格

切换方式：

* 从编辑模式切换到命令模式，按 `Esc` 键或 `Ctrl-M`
* 从命令模式切换到编辑模式，按 `Enter` 或点击单元格

Jupyter 笔记本的模式行为在习惯后非常高效。

#### 插入 Unicode（例如希腊字母）

Python 支持 [unicode](https://docs.python.org/3/howto/unicode.html)，允许在代码中使用 $\alpha$ 和 $\beta$ 等字符作为名称。

在代码单元格中，尝试输入 `\alpha`，然后按键盘上的 Tab 键。

(a_test_program)=
#### 测试程序

让我们运行一个测试程序。

这是一个我们可以使用的任意程序：[https://matplotlib.org/stable/gallery/pie_and_polar_charts/polar_bar.html](https://matplotlib.org/stable/gallery/pie_and_polar_charts/polar_bar.html)。

在该页面上，您将看到以下代码：

```{code-cell} ipython
import numpy as np
import matplotlib.pyplot as plt

# 固定随机状态以保证可重现性
np.random.seed(19680801)

# 计算饼图切片
N = 20
θ = np.linspace(0.0, 2 * np.pi, N, endpoint=False)
radii = 10 * np.random.rand(N)
width = np.pi / 4 * np.random.rand(N)
colors = plt.cm.viridis(radii / 10.)

ax = plt.subplot(111, projection='polar')
ax.bar(θ, radii, width=width, bottom=0.0, color=colors, alpha=0.5)

plt.show()
```

现在不必担心细节——让我们直接运行它，看看会发生什么。

运行此代码的最简单方法是将其复制粘贴到笔记本的单元格中。

希望您能得到类似的图形。

### 使用笔记本

以下是使用 Jupyter 笔记本的更多技巧。

#### Tab 补全

在前面的程序中，我们执行了 `import numpy as np` 这一行。

* NumPy 是我们将深入使用的数值库。

执行此导入命令后，NumPy 中的函数可以通过 `np.函数名` 语法访问。

* 例如，试试 `np.random.randn(3)`。

我们可以使用 `Tab` 键来探索 `np` 的这些属性。

例如，这里我们输入 `np.random.r` 并按 Tab：

```{figure} /_static/lecture_specific/getting_started/nb6.png
:figclass: auto
```

Jupyter 为您提供了几个可供选择的补全项。

这样，Tab 键既能提醒您有哪些可用选项，又能节省您的输入时间。

(gs_help)=
#### 在线帮助

```{index} single: Jupyter Notebook; Help
```

要获取 `np.random.randn` 的帮助，我们可以执行 `np.random.randn?`。

文档会出现在浏览器的分割窗口中，如下所示：

```{figure} /_static/lecture_specific/getting_started/nb6a.png
:figclass: auto
```

点击下方分割窗口的右上角可关闭在线帮助。

我们将在{ref}`后续内容 <Docstrings>`中学习更多关于如何创建此类文档的知识！

#### 其他内容

除了执行代码外，Jupyter 笔记本还允许您在页面中嵌入文本、方程式、图形甚至视频。

例如，我们可以输入纯文本和 LaTeX 的混合内容，而不是代码。

接下来我们按 `Esc` 进入命令模式，然后输入 `m` 表示我们正在编写 [Markdown](https://daringfireball.net/projects/markdown/)，这是一种类似于（但比 LaTeX 更简单的）标记语言。

（您也可以用鼠标从菜单项列表下方的 `Code` 下拉框中选择 `Markdown`）

```{figure} /_static/lecture_specific/getting_started/nb7.png
:figclass: auto
```

现在我们按 `Shift+Enter` 生成以下内容：

```{figure} /_static/lecture_specific/getting_started/nb8.png
:figclass: auto
```

### 调试代码

```{index} single: Jupyter Notebook; Debugging
```

调试是识别和消除程序错误的过程。

您将花费大量时间调试代码，因此学会[如何有效地进行调试](https://www.freecodecamp.org/news/what-is-debugging-how-to-debug-code/)非常重要。

如果您使用的是较新版本的 Jupyter，您应该能在工具栏右端看到一个虫子图标。

```{figure} /_static/lecture_specific/getting_started/debug.png
:scale: 50%
:figclass: auto
```

点击此图标将启用 Jupyter 调试器。

<!-- IDEA: This could be turned into a margin note once supported by quantecon-book-theme -->
```{note}
您可能还需要打开调试器面板（视图 -> 调试器面板）。
```

您可以通过点击要调试的单元格的行号来设置断点。

运行单元格时，调试器将在断点处停止。

然后，您可以使用 CALLSTACK 工具栏（位于右侧窗口）上的"下一步"按钮逐行执行代码。

<!-- IDEA: add a red square around the area of interest in the image -->
```{figure} /_static/lecture_specific/getting_started/debugger_breakpoint.png
:figclass: auto
```

您可以在 [Jupyter 文档](https://jupyterlab.readthedocs.io/en/latest/user/debugger.html)中探索调试器的更多功能。

### 共享笔记本

```{index} single: Jupyter Notebook; Sharing
```

```{index} single: Jupyter Notebook; nbviewer
```

笔记本文件只是结构化为 [JSON](https://en.wikipedia.org/wiki/JSON) 格式的文本文件，通常以 `.ipynb` 为扩展名。

您可以用通常共享文件的方式共享它们——或者使用 [nbviewer](https://nbviewer.org/) 等网络服务。

您在该网站上看到的笔记本是**静态** html 表示。

要运行某个笔记本，请点击右上角的下载图标，将其下载为 `ipynb` 文件。

将其保存到某处，从 Jupyter 仪表板导航到该文件，然后按上述方式运行。

```{note}
如果您有兴趣共享包含交互内容的笔记本，可以查看 [Binder](https://mybinder.org/)。

要与他人协作处理笔记本，您可以了解以下工具：

- [Google Colab](https://colab.research.google.com/)
- [Kaggle](https://www.kaggle.com/code)

要保持代码私密并使用熟悉的 JupyterLab 和笔记本界面，请了解 [JupyterLab 实时协作扩展](https://jupyterlab-realtime-collaboration.readthedocs.io/en/latest/)。
```

### QuantEcon Notes

QuantEcon 有自己的网站，用于共享与经济学相关的 Jupyter 笔记本——[QuantEcon Notes](http://notes.quantecon.org/)。

提交到 QuantEcon Notes 的笔记本可以通过链接共享，并对社区的评论和投票开放。

## 安装库

(gs_qe)=
```{index} single: QuantEcon
```

我们需要的大多数库都包含在 Anaconda 中。

其他库可以用 `pip` 或 `conda` 安装。

我们将使用的一个库是 [QuantEcon.py](https://quantecon.org/quantecon-py/)。

(gs_install_qe)=
您可以通过启动 Jupyter 并在单元格中输入以下内容来安装 [QuantEcon.py](https://quantecon.org/quantecon-py/)：

```{code-block} ipython3
:class: no-execute

!conda install quantecon
```

或者，您可以在终端中输入以下内容：

```{code-block} bash
:class: no-execute

conda install quantecon
```

更多说明可以在[库页面](https://quantecon.org/quantecon-py/)找到。

要升级到最新版本（您应该定期执行此操作），请使用：

```{code-block} bash
:class: no-execute

conda upgrade quantecon
```

我们将使用的另一个库是 [interpolation.py](https://github.com/EconForge/interpolation.py)。

可以在 Jupyter 中输入以下内容来安装：

```{code-block} ipython3
:class: no-execute

!conda install -c conda-forge interpolation
```

## 使用 Python 文件

到目前为止，我们专注于执行输入到 Jupyter 笔记本单元格中的 Python 代码。

传统上，大多数 Python 代码以不同的方式运行。

代码首先保存在本地计算机的文本文件中。

按照惯例，这些文本文件具有 `.py` 扩展名。

我们可以按如下方式创建此类文件的示例：

```{code-cell} ipython
%%writefile foo.py

print("foobar")
```

这将 `print("foobar")` 这一行写入本地目录中名为 `foo.py` 的文件。

这里 `%%writefile` 是[单元格魔法命令](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cell-magics)的一个示例。

### 编辑与执行

如果您遇到保存在 `*.py` 文件中的代码，需要考虑以下问题：

1. 应该如何执行它？
1. 应该如何修改或编辑它？

#### 选项 1：{index}`JupyterLab <single: JupyterLab>`

```{index} single: JupyterLab
```

[JupyterLab](https://github.com/jupyterlab/jupyterlab) 是一个建立在 Jupyter 笔记本之上的集成开发环境。

使用 JupyterLab，您可以编辑和运行 `*.py` 文件以及 Jupyter 笔记本。

要启动 JupyterLab，请在应用程序菜单中搜索它，或在终端中输入 `jupyter-lab`。

现在您应该能够通过在 JupyterLab 中打开上面创建的 `foo.py` 文件来编辑和运行它。

请阅读文档或搜索最近的 YouTube 视频以获取更多信息。

#### 选项 2：使用文本编辑器

也可以使用文本编辑器编辑文件，然后在 Jupyter 笔记本中运行它们。

文本编辑器是专门为处理文本文件（如 Python 程序）而设计的应用程序。

对于处理程序文本而言，一个好的文本编辑器的功能和效率无可比拟。

好的文本编辑器将提供：

* 高效的文本编辑命令（例如，复制、粘贴、查找和替换）
* 语法高亮等

目前，用于编码的极为流行的文本编辑器是 [VS Code](https://code.visualstudio.com/)。

VS Code 开箱即用，且拥有许多高质量的扩展。

另外，如果您想要一个出色的免费文本编辑器，且不介意看似陡峭的学习曲线以及在所有神经通路被重新布线期间漫长的痛苦煎熬，可以尝试 [Vim](https://www.vim.org/)。

## 练习

```{exercise-start}
:label: gs_ex1
```

如果 Jupyter 仍在运行，请在启动它的终端中使用 `Ctrl-C` 退出。

现在重新启动，但这次使用 `jupyter notebook --no-browser`。

这应该会在不启动浏览器的情况下启动内核。

同时注意启动消息：它应该给您一个 URL，例如 `http://localhost:8888`，表示笔记本正在运行的地址。

现在：

1. 启动您的浏览器——如果已在运行，则打开一个新标签页。
1. 在顶部的地址栏中输入上述 URL（例如 `http://localhost:8888`）。

您现在应该能够运行标准的 Jupyter 笔记本会话。

这是启动笔记本的另一种方式，也非常方便。

只要内核仍在运行，这在您不小心关闭网页时也能正常工作。

```{exercise-end}
```