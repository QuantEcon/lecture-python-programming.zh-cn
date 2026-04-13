---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.2
kernelspec:
  name: python3
  display_name: Python 3 (ipykernel)
  language: python
translation:
  title: 编写较长的程序
  headings:
    Overview: 概述
    Working with Python files: 使用 Python 文件
    Development environments: 开发环境
    'A step forward from Jupyter Notebooks: JupyterLab': Jupyter Notebook 的进阶：JupyterLab
    'A step forward from Jupyter Notebooks: JupyterLab::Using magic commands': 使用魔法命令
    'A step forward from Jupyter Notebooks: JupyterLab::Using the terminal': 使用终端
    A walk through Visual Studio Code: Visual Studio Code 使用指南
    A walk through Visual Studio Code::Using the run button: 使用运行按钮
    A walk through Visual Studio Code::Using the terminal: 使用终端
    Git your hands dirty: 动手实践 Git
---

(workspace)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# 编写较长的程序

## 概述

到目前为止，我们已经探索了在 Jupyter Notebook 中编写和执行 Python 代码的使用方法。

虽然在处理短代码时 Notebook 高效且灵活，但对于较长的程序和脚本而言，Notebook 并不是最佳选择。

Jupyter Notebook 非常适合交互式计算（即数据科学工作流），可以帮助逐块执行代码。

文本文件和脚本则允许将较长的代码一次性编写并执行。

我们将探索使用 Python 脚本作为替代方案。

随后介绍 JupyterLab 和 Visual Studio Code（VS Code）开发环境，以及版本控制（Git）入门。

在本讲座中，您将学习：
- 使用 Python 脚本
- 配置各种开发环境
- 开始使用 GitHub

```{note}
在接下来的内容中，假设您已经安装并运行了 Anaconda 环境。

如果您还没有这样做，可能需要[创建一个新的 conda 环境](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands)。
```

## 使用 Python 文件

Python 文件用于编写较长的、可复用的代码块——按照惯例，它们具有 `.py` 后缀。

让我们从以下示例开始。

```{code-cell} ipython3
:caption: sine_wave.py
:lineno-start: 1

import matplotlib.pyplot as plt
import matplotlib as mpl  # i18n
import matplotlib.font_manager  # i18n
import numpy as np

FONTPATH = "_fonts/SourceHanSerifSC-SemiBold.otf"  # i18n
mpl.font_manager.fontManager.addfont(FONTPATH)  # i18n
mpl.rcParams['font.family'] = ['Source Han Serif SC']  # i18n

x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.plot(x, y)
plt.xlabel('x')
plt.ylabel('y')
plt.title('正弦波')
plt.show()
```

由于有多种方式可以执行代码，我们将在不同的开发环境中探索这些方式。

使用 Python 脚本的一大优势在于，您可以从其他脚本中将功能"导入"到当前脚本或 Jupyter Notebook 中。

让我们将之前的代码重写为一个函数，并将其写入名为 `sine_wave.py` 的文件中。

```{code-cell} ipython3
:caption: sine_wave.py
:lineno-start: 1

%%writefile sine_wave.py

import matplotlib.pyplot as plt
import numpy as np

# 定义 plot_wave 函数
def plot_wave(title : str = '正弦波'):
  x = np.linspace(0, 10, 100)
  y = np.sin(x)

  plt.plot(x, y)
  plt.xlabel('x')
  plt.ylabel('y')
  plt.title(title)
  plt.show()
```

```{code-cell} ipython3
:caption: second_script.py
:lineno-start: 1

import sine_wave # 导入 sine_wave 脚本
 
# 调用 plot_wave 函数
sine_wave.plot_wave("正弦波 - 从第二个脚本调用")
```

这使您可以将代码分割成若干块，并更好地组织代码库。

有关导入功能的更多信息，请查阅[模块](https://docs.python.org/3/tutorial/modules.html)和[包](https://docs.python.org/3/tutorial/modules.html#packages)的使用。

## 开发环境

开发环境是一个一站式工作空间，您可以在其中：
- 编辑和运行代码
- 测试和调试
- 管理项目文件

本讲座将带您了解两种开发环境的使用方法。

## Jupyter Notebook 的进阶：JupyterLab

JupyterLab 是一个基于浏览器的开发环境，支持 Jupyter Notebook、代码脚本和数据文件。

如果您想在本地安装之前先试用，可以[在浏览器中试用 JupyterLab](https://jupyter.org/try-jupyter/lab/)。

您可以使用 pip 安装 JupyterLab：

```
> pip install jupyterlab
``` 

然后像 Jupyter Notebook 一样在浏览器中启动它：

```
> jupyter-lab
```

```{figure} /_static/lecture_specific/workspace/jupyter_lab_cmd.png
:figclass: auto
```

您可以看到 Jupyter 服务器正在本地主机的 8888 端口上运行。

以下界面应会自动在您的默认浏览器中打开——如果没有，请按住 CTRL 并单击服务器 URL。

```{figure} /_static/lecture_specific/workspace/jupyter_lab.png
:figclass: auto
```

点击：

- Notebooks 下的 Python 3（ipykernel）按钮以打开新的 Jupyter Notebook
- Python File 按钮以打开新的 Python 脚本（.py）

您可以随时通过点击顶部的"+"按钮来打开此启动器标签页。

工作目录中的所有文件和文件夹可以在文件浏览器（左侧标签页）中找到。

您可以使用文件浏览器标签页顶部的按钮创建新文件和文件夹。

```{figure} /_static/lecture_specific/workspace/file_browser.png
:figclass: auto
```

您可以通过访问扩展标签页来安装增加 JupyterLab 功能的扩展。

```{figure} /_static/lecture_specific/workspace/extensions.png
:figclass: auto
```

回到之前的示例脚本，在 JupyterLab 中有两种方式可以使用它们：

- 使用魔法命令
- 使用终端

### 使用魔法命令

Jupyter Notebook 和 JupyterLab 支持使用[魔法命令](https://ipython.readthedocs.io/en/stable/interactive/magics.html)——这些命令扩展了标准 Jupyter Notebook 的功能。

`%run` 魔法命令允许您在 Notebook 中运行 Python 脚本。

这是一种便捷的方式，可以运行与 Notebook 处于同一目录下的脚本，并在 Notebook 中显示输出结果。

```{figure} /_static/lecture_specific/workspace/jupyter_lab_py_run.png
:figclass: auto
```

### 使用终端

但是，如果您只是想运行 `.py` 文件，有时使用终端会更方便。

从启动器打开终端并运行以下命令：

```
> python <path to file.py>
``` 

```{figure} /_static/lecture_specific/workspace/jupyter_lab_py_run_term.png
:figclass: auto
```

```{note}
您也可以通过打开 ipykernel 控制台逐行运行脚本，可以：
- 从启动器打开
- 在 Notebook 中右键单击并选择 Create Console for Editor

使用 Shift + Enter 运行一行代码。
```

## Visual Studio Code 使用指南

Visual Studio Code（VS Code）是一个代码编辑器和开发工作空间，可以：
- 在[浏览器](https://vscode.dev/)中运行
- 作为本地[安装](https://code.visualstudio.com/docs/?dv=win)运行

两种界面完全相同。

启动 VS Code 时，您将看到以下界面：

```{figure} /_static/lecture_specific/workspace/vs_code_home.png
:figclass: auto
```

通过引导式演练探索如何按照您的喜好自定义 VS Code。

```{figure} /_static/lecture_specific/workspace/vs_code_walkthrough.png
:figclass: auto
```

出现以下提示时，继续安装所有推荐的扩展。

```{figure} /_static/lecture_specific/workspace/vs_code_install_ext.png
:figclass: auto
```

您也可以从扩展标签页安装扩展。

```{figure} /_static/lecture_specific/workspace/vs_code_extensions.png
:figclass: auto
```

Jupyter Notebook（`.ipynb` 文件）可以在 VS Code 中使用。

在尝试打开 Jupyter Notebook 之前，请确保从扩展标签页安装了 Jupyter 扩展。

创建一个新文件（在文件资源管理器标签页中）并以 `.ipynb` 扩展名保存。

通过点击编辑器右上角的 Select Kernel 按钮选择运行 Notebook 的内核/环境。

```{figure} /_static/lecture_specific/workspace/vs_code_kernels.png
:figclass: auto
```

VS Code 还通过源代码管理标签页提供出色的版本控制功能。

```{figure} /_static/lecture_specific/workspace/vs_code_git.png
:figclass: auto
```

将您的 GitHub 账户链接到 VS Code，以便向您的仓库推送和拉取更改。

有关版本控制的进一步讨论可以在下一节中找到。

要在 VS Code 中打开新终端，请点击终端标签页并选择新建终端。

VS Code 会在您当前工作的目录中打开一个新终端——在 Windows 上是 PowerShell，在 Linux 上是 Bash。

您可以通过终端标签页右端的下拉菜单更改 shell 或打开新实例。

```{figure} /_static/lecture_specific/workspace/vs_code_terminal_opts.png
:figclass: auto
```

VS Code 可以帮助您在不使用命令行的情况下管理 conda 环境。

打开命令面板（CTRL + SHIFT + P 或从视图标签页的下拉菜单中打开）并搜索 ```Python: Select Interpreter```。

这将加载现有环境。

您也可以在命令面板中使用 ```Python: Create Environment``` 创建新环境。

新环境（.conda 文件夹）将在当前工作目录中创建。

回到之前的示例脚本，在 VS Code 中同样有两种方式可以使用它们：

- 使用运行按钮
- 使用终端

### 使用运行按钮

您可以通过点击编辑器右上角的运行按钮来运行脚本。

```{figure} /_static/lecture_specific/workspace/vs_code_run.png
:figclass: auto
```

您也可以通过从下拉菜单中选择 **Run Current File in Interactive Window** 选项来交互式地运行脚本。

```{figure} /_static/lecture_specific/workspace/vs_code_run_button.png
:figclass: auto
```

这将创建一个 ipykernel 控制台并运行脚本。

### 使用终端

命令 `python <path to file.py>` 在您选择的控制台上执行。

如果您使用的是 Windows 机器，可以使用 Anaconda Prompt 或命令提示符——但通常不使用 PowerShell。

以下是之前代码的执行示例：

```{figure} /_static/lecture_specific/workspace/sine_wave_import.png
:figclass: auto
```

```{note}
如果您想使用 Python 开发包和构建工具，可能需要了解[Docker 容器与 VS Code 的使用](https://github.com/RamiKrispin/vscode-python)。

但是，这超出了这些讲座的重点范围。
```

## 动手实践 Git

本节将帮助您熟悉 Git 和 GitHub。

[Git](https://git-scm.com/) 是一个*版本控制系统*——一种用于管理数字项目（如代码库）的软件。

在许多情况下，相关的文件集合——称为*仓库*——存储在 [GitHub](https://github.com/) 上。

GitHub 是协作编程项目的宝藏之地。

例如，它托管了我们稍后将使用的许多科学库，例如[这个](https://github.com/pandas-dev/pandas)。

Git 是用于管理这些项目的底层软件。

Git 是一个极其强大的分布式协作工具——例如，我们用它来共享和同步这些讲座的所有源文件。

Git 主要有两种形式：

1. 纯粹的[命令行 Git](https://git-scm.com/downloads/) 版本
2. 各种点击式 GUI 版本
    * 例如，参见 [GitHub 版本](https://github.com/apps/desktop)或集成到您的 IDE 中的 Git GUI。

如果您还没有，请尝试：

1. 安装 Git。
1. 使用 Git 获取 [QuantEcon.py](https://github.com/QuantEcon/QuantEcon.py) 的副本。

例如，如果您已安装命令行版本，打开终端并输入：

```bash
git clone https://github.com/QuantEcon/QuantEcon.py
```

（这只是在仓库 URL 前面加上 `git clone`）

此命令将下载重建您正在阅读的讲座所需的所有组件。

作为第二个任务：

1. 注册 [GitHub](https://github.com/)。
1. 了解"分叉"GitHub 仓库的方法（分叉意味着创建一个存储在 GitHub 上的 GitHub 仓库的自己的副本）。
1. 分叉 [QuantEcon.py](https://github.com/QuantEcon/QuantEcon.py)。
1. 将您的分叉克隆到某个本地目录，进行编辑，提交更改，并将其推送回您分叉的 GitHub 仓库。
1. 如果您做出了有价值的改进，请向我们发送[拉取请求](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)！

有关这些及其他主题的阅读资料，请参阅：

* [Git 官方文档](https://git-scm.com/doc)。
* 阅读 [GitHub](https://docs.github.com/en) 上的文档。
* Scott Chacon 和 Ben Straub 所著的 [Pro Git Book](https://git-scm.com/book)。
* 网络上数以千计的 Git 教程之一。