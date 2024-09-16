---
title: 【Python Debug】"解决报错: qt.qpa.plugin Could not find the Qt platform plugin \"xcb\" in \"\""
date: 2023-04-30 10:41:32
toc: false
categories: 
 - Programming
 - Python
---

## 问题

使用`plotly`绘图生成图片的时候遇到如下报错：
```
qt.qpa.plugin: Could not find the Qt platform plugin "xcb" in ""
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.
```

## 解决过程

#### 尝试一
参考[这篇Askubuntu文章](https://askubuntu.com/questions/1228495/cannot-open-qcreator-qt-qpa-plugin-could-not-find-the-qt-platform-plugin-xcb)
```
指定环境变量`QT_QPA_PLATFORM_PLUGIN_PATH`。
```bash
export QT_QPA_PLATFORM_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/qt5/plugins/platforms
```
报错变为：
```
qt.core.plugin.loader: In /usr/lib/x86_64-linux-gnu/qt5/plugins/platforms/libqeglfs.so:
  Plugin uses incompatible Qt library (5.15.0) [release]
qt.core.plugin.loader: In /usr/lib/x86_64-linux-gnu/qt5/plugins/platforms/libqlinuxfb.so:
  Plugin uses incompatible Qt library (5.15.0) [release]
qt.core.plugin.loader: In /usr/lib/x86_64-linux-gnu/qt5/plugins/platforms/libqminimal.so:
  Plugin uses incompatible Qt library (5.15.0) [release]
qt.core.plugin.loader: In /usr/lib/x86_64-linux-gnu/qt5/plugins/platforms/libqminimalegl.so:
  Plugin uses incompatible Qt library (5.15.0) [release]
qt.core.plugin.loader: In /usr/lib/x86_64-linux-gnu/qt5/plugins/platforms/libqoffscreen.so:
  Plugin uses incompatible Qt library (5.15.0) [release]
qt.core.plugin.loader: In /usr/lib/x86_64-linux-gnu/qt5/plugins/platforms/libqvnc.so:
  Plugin uses incompatible Qt library (5.15.0) [release]
qt.core.plugin.loader: In /usr/lib/x86_64-linux-gnu/qt5/plugins/platforms/libqxcb.so:
  Plugin uses incompatible Qt library (5.15.0) [release]
qt.qpa.plugin: Could not find the Qt platform plugin "xcb" in "/usr/lib/x86_64-linux-gnu/qt5/plugins/platforms"
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.
```

#### 尝试二
考虑`python`程序用了`PySide6`包和QT6相关，考虑是不是因为这个版本不匹配，于是安装了Qt6并将环境变量指向Qt6的位置：
```bash
sudo apt install qt6-base-dev qt6-base-dev-tools
export QT_QPA_PLATFORM_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/qt6/plugins/platforms
```
报错变为：
```
qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/usr/lib/x86_64-linux-gnu/qt6/plugins/platforms" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vkkhrdisplay, vnc, xcb.
```

再考虑是不是python包版本不匹配，于是强制升级了`PySide6`和`PyQt6`。
报错变为：
```
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/_api/deprecation.py", line 454, in wrapper
    return func(*args, **kwargs)
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/pyplot.py", line 771, in figure
    manager = new_figure_manager(
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/pyplot.py", line 346, in new_figure_manager
    _warn_if_gui_out_of_main_thread()
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/pyplot.py", line 336, in _warn_if_gui_out_of_main_thread
    if (_get_required_interactive_framework(_get_backend_mod()) and
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/pyplot.py", line 206, in _get_backend_mod
    switch_backend(dict.__getitem__(rcParams, "backend"))
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/pyplot.py", line 251, in switch_backend
    switch_backend(candidate)
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/pyplot.py", line 264, in switch_backend
    backend_mod = importlib.import_module(
  File "/anaconda3/lib/python3.9/importlib/__init__.py", line 127, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "<frozen importlib._bootstrap>", line 1030, in _gcd_import
  File "<frozen importlib._bootstrap>", line 1007, in _find_and_load
  File "<frozen importlib._bootstrap>", line 986, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 680, in _load_unlocked
  File "<frozen importlib._bootstrap_external>", line 850, in exec_module
  File "<frozen importlib._bootstrap>", line 228, in _call_with_frames_removed
  File "/anaconda3/lib/pythoadd n3.9/site-packages/matplotlib/backends/backend_qtagg.py", line 12, in <module>
    from .backend_qt import QtCore, QtGui, _BackendQT, FigureCanvasQT
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/backends/backend_qt.py", line 72, in <module>
    _MODIFIER_KEYS = [
  File "/anaconda3/lib/python3.9/site-packages/matplotlib/backends/backend_qt.py", line 73, in <listcomp>
    (_to_int(getattr(_enum("QtCore.Qt.KeyboardModifier"), mod)),
TypeError: int() argument must be a string, a bytes-like object or a number, not 'KeyboardModifier'

```
发现涉及到`matplotlib`，同样升级之，错误变为：
```
qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/usr/lib/x86_64-linux-gnu/qt6/plugins/platforms" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, offscreen, minimal, xcb, wayland-egl, linuxfb, vkkhrdisplay, wayland, vnc, minimalegl.
```
百思不得其解。

#### 解决
网上找到设置`export QT_DEBUG_PLUGINS=1`后可以让Qt输出详细debug信息，设置后运行程序输出：
```
qt.core.plugin.factoryloader: Got keys from plugin meta data QList("minimalegl")
qt.core.library: "/anaconda3/lib/python3.9/site-packages/PyQt6/Qt6/plugins/platforms/libqxcb.so" cannot load: Cannot load library /anaconda3/lib/python3.9/site-packages/PyQt6/Qt6/plugins/platforms/libqxcb.so: (libxcb-cursor.so.0: cannot open shared object file: No such file or directory)
qt.core.plugin.loader: QLibraryPrivate::loadPlugin failed on "/anaconda3/lib/python3.9/site-packages/PyQt6/Qt6/plugins/platforms/libqxcb.so" : "Cannot load library /anaconda3/lib/python3.9/site-packages/PyQt6/Qt6/plugins/platforms/libqxcb.so: (libxcb-cursor.so.0: cannot open shared object file: No such file or directory)"
qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/usr/lib/x86_64-linux-gnu/qt6/plugins/platforms" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.
```
发现
- 实际上被使用的是PyQt6自带的插件动态库，并没有用`QT_QPA_PLATFORM_PLUGIN_PATH`指定的系统库。
- 错误原因是缺失`libxcb-cursor.so.0`。
于是安装`libxcb-cursor`
```bash
sudo apt install libxcb-cursor*
```
再次运行发现不再报错。问题解决。

删除掉`QT_DEBUG_PLUGINS`和`QT_QPA_PLATFORM_PLUGIN_PATH`环境变量（因为实际上并没有用系统的Qt6，所以`QT_QPA_PLATFORM_PLUGIN_PATH`删掉不影响什么）。
