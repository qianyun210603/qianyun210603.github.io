---
layout: "post"
title: "Ubuntu 20.04安装VNC服务（Xfce4）"
date: "2023-02-04 17:35"
categories:
  - Linux
---


#### 目标

给VPS安装VNC服务以能使用图形化界面软件。

#### 步骤

1. 安装Xfce桌面环境

   ```sudo apt install xfce4```

    <font size=1>
    a. 中途会让选择是用`gdm3`还是`lightdm`。根据网上描述[^fn1]，`lightdm`占用资源更少，支持的桌面系统更多，同时可定制性更好；而`gdm3`是专门为`GNOME`开发的。所以这里选`lightdm`。
    </font>

1. 安装VNC Server

    ```sudo apt install tigervnc-standalone-server```

2. 初始化
3. 运行命令`vncserver`，会提示输入密码，这个密码是后期连接vnc的密码，一定要记住。
4. 运行`vncserver -kill :1`杀掉刚刚的服务进程。
5. `vi ~/.vnc/xstartup`建立`xstartup`文件，并将以下内容复制进去：

    ```bash
    #!/bin/sh
    # Uncomment the following two lines for normal desktop:
    # unset SESSION_MANAGER
    # exec /etc/X11/xinit/xinitrc

    [ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
    [ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
    xsetroot -solid grey
    vncconfig -iconic &
    x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
    x-window-manager &

    #gnome-terminal &

    sesion-manager & xfdesktop & xfce4-panel &
    xfce4-menu-plugin &
    xfsettingsd &
    xfconfd &
    xfwm4 &
    ```

6. 再次开启`vncserver`: `vncserver -localhost no`
     `tigervncserver`默认只能本机访问，`-localhost no`参数关闭此限制。

#### 其他

- VNC服务默认使用从5901开始的端口，一台VPS上可以开启多个VNC服务，连接时`ip:1`对应5901端口的服务，连接时`ip:2`对应5902端口的服务，以此类推。
- 如VPS有防火墙需要打开对应的端口。

^fn1: <https://www.linuxfordevices.com/tutorials/linux/gdm3-vs-lightdm>
