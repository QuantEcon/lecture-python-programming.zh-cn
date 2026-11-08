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
  title: 故障排除
  headings:
    Fixing Your Local Environment: 修复本地环境
    Reporting an Issue: 提交问题报告
---

(troubleshooting)=
```{raw} jupyter
<div id="qe-notebook-header" align="right" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" width="250px" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# 故障排除

本页面适用于在运行讲座代码时遇到错误的读者。

## 修复本地环境

本系列讲座的基本假设是：满足以下条件时，讲座中的代码都应能正常运行：

1. 在 Jupyter Notebook 中运行；并且
1. Notebook 运行在安装了最新版 Anaconda Python 的机器上。

您已经按照{doc}`本讲座 <getting_started>`中的说明安装了 Anaconda，对吗？

假设您已安装，我们的读者最常见的问题来源是其 Anaconda 发行版未能保持最新。

[这里有一篇有用的文章](https://www.anaconda.com/blog/keeping-anaconda-date)
介绍了如何更新 Anaconda。

另一个选择是直接卸载 Anaconda 并重新安装。

您还需要及时更新外部依赖库，例如 [QuantEcon.py](https://quantecon.org/quantecon-py/)。

为此，您可以：

* 在命令行中使用 conda upgrade quantecon，或
* 在 Jupyter notebook 中执行 !conda upgrade quantecon。

如果您的本地环境仍然无法正常工作，您可以采取以下两种措施。

首先，您可以转而使用远程机器，点击每个讲座中提供的"启动 Notebook"图标即可。

```{image} _static/lecture_specific/troubleshooting/launch.png

```

其次，您可以提交问题报告，以便我们尝试修复您的本地环境。

我们很乐意收到关于讲座的反馈，请随时与我们联系。

## 提交问题报告

提供反馈的一种方式是通过我们的[问题追踪器](https://github.com/QuantEcon/lecture-python-programming/issues)提交问题。

请尽量具体地描述问题，指出问题出现的位置，并尽可能提供本地环境的详细信息。

最后，您也可以直接通过 [contact@quantecon.org](mailto:contact@quantecon.org) 与我们联系。