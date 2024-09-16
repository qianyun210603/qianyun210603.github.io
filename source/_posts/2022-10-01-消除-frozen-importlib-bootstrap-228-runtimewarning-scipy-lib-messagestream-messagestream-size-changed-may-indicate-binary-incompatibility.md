---
layout: "post"
title: 【Python Debug】"消除&lt;frozen importlib._bootstrap&gt;:228: RuntimeWarning: scipy._lib.messagestream.MessageStream size changed, may indicate binary incompatibility."
date: "2022-10-01 14:08"
categories:
- Programming
- Python
---

### 问题
在`import scipy.stats`的时候报警告：
```python
Python 3.9.12 (main, Apr  5 2022, 06:56:58)
[GCC 7.5.0] :: Anaconda, Inc. on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from scipy.stats import percentileofscore
<frozen importlib._bootstrap>:228: RuntimeWarning: scipy._lib.messagestream.MessageStream size changed, may indicate binary incompatibility. Expected 56 from C header, got 64 from PyObject
>>> exit()
```

### 解决
这个的原因之一是pip在升级numpy，scipy的时候(`pip install -U numpy scipy`)，对旧的package的删除可能不干净的问题。需要多次用`uninstall`直至无残留：

以numpy为例
```bash
(base) $ pip uninstall numpy
Found existing installation: numpy 1.23.3
Uninstalling numpy-1.23.3:
  Would remove:
    /anaconda3/bin/f2py
    /anaconda3/bin/f2py3
    /anaconda3/bin/f2py3.9
    /anaconda3/lib/python3.9/site-packages/numpy-1.23.3.dist-info/*
    /anaconda3/lib/python3.9/site-packages/numpy.libs/libgfortran-040039e1.so.5.0.0
    /anaconda3/lib/python3.9/site-packages/numpy.libs/libopenblas64_p-r0-742d56dc.3.20.so
    /anaconda3/lib/python3.9/site-packages/numpy.libs/libquadmath-96973f99.so.0.0.0
    /anaconda3/lib/python3.9/site-packages/numpy/*
  Would not remove (might be manually added):
    /anaconda3/lib/python3.9/site-packages/numpy/distutils/site.cfg
Proceed (Y/n)? y
  Successfully uninstalled numpy-1.23.3
(base) $ pip install numpy
Requirement already satisfied: numpy in /anaconda3/lib/python3.9/site-packages (1.21.5)
(base) $ pip uninstall numpy
Found existing installation: numpy 1.21.5
Uninstalling numpy-1.21.5:
  Would remove:
    /anaconda3/lib/python3.9/site-packages/numpy-1.21.5.dist-info/*
    /anaconda3/lib/python3.9/site-packages/numpy/distutils/site.cfg
Proceed (Y/n)? y
  Successfully uninstalled numpy-1.21.5
```

可见在重新安装的时候，发现卸载掉1.23.3后，残余的1.21.5暴露出来了。所以需要再次卸载，直至完全卸载干净。

对scipy也是同样操作。
