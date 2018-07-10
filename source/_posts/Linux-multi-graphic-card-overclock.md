---
title: Linux系统下多Nvidia显卡超频
date: 2018-07-10 19:56:58
categories:
- Linux
- Cuda
---

1. 安装驱动
```bash
apt install nvidia-384 nvidia-384-dev
```

2. 下载一个edid文件用来模拟显示器以使得没有连接显示器的显卡可以被设置。

3. 用`xconfig`生成`xorg.conf`文件(生成后位于`/etc/X11/`)
```bash
nvidia-xconfig -a --cool-bits=28 --custom-edid="DFP-0:<path to edid file>" --connected-monitor="DFP-0" --no-use-display-device
```

4. 打开`/etc/X11/xorg.conf`文件，可以看到对应每块显卡，均生成了`Monitor`，`Device`，`Screen`三个Section，其中`Screen` Section如下
```
Section "Screen"
    Identifier     "Screen1"
    Device         "Device1"
    Monitor        "Monitor1"
    DefaultDepth    24
    Option         "CustomEDID" "DFP-0:/etc/X11/delledid.bin"
    Option         "ConnectedMonitor" "DFP-0"
    Option         "UseDisplayDevice" "none"
    Option         "Coolbits" "28"
    SubSection     "Display"
        Depth       24
    EndSubSection
EndSection
```
可以看到刚才设置的各个条目都在`Option`里面。找到并删除掉实际连接显示器的Section中除了`Coolbits`之外的其他`Option`。

5. 重启电脑。