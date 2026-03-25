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
  title: 执行统计
---

# 执行统计

此表包含最新的执行统计信息。

```{nb-exec-table}
```

(status:machine-details)=

这些讲座通过 `github actions` 在 `linux` 实例上构建。

这些讲座使用以下 Python 版本

```{code-cell} ipython
!python --version
```

以及以下软件包版本

```{code-cell} ipython
:tags: [hide-output]
!conda list
```

本讲座系列可以访问以下 GPU

```{code-cell} ipython
!nvidia-smi
```

您可以使用以下方式检查 JAX 使用的后端：

```{code-cell} ipython3
import jax
# 检查 JAX 是否正在使用 GPU
print(f"JAX backend: {jax.devices()[0].platform}")
```