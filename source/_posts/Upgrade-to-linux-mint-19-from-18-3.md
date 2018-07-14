---
title: Linux Mint从18.3升级到19
date: 2018-07-12 18:54:37
categories:
- Linux
---

#####一、检查升级需求是否满足

1. 只有Linux Mint 18.3 Cinnamon，MATE和Xfce版本可以用升级包升级。
    * 使用18、18.1、18.2的需要先用Update Manager升级到18.3。

2. 使用Timeshift保存一个系统镜像。
	* 使用`apt install timeshift`安装Timeshift。
	* 进入"Menu -> Administration -> Timeshift"打开Timeshift。
	* 按提示创建镜像并设置自动创建频率，一切按默认选项即可。

3. 确定当前X Display Manager是LightDM。
```
cat /etc/X11/default-display-manager
```
* 如果得到`/usr/sbin/lightdm`，满足条件。
* 如果是`/usr/sbin/mdm`，则需要切换到LightDM。
```
sudo apt install lightdm lightdm-settings slick-greeter # 要求选择LDM或MDM时选择LDM
sudo apt remove --purge mdm mint-mdm-themes*
sudo dpkg-reconfigure lightdm
sudo reboot
```

#### 升级步骤

1. 更新现有Linux Mint 18.3系统。
```
sudo apt update
sudo apt upgrade
```

2. 设置终端可以滚动无限多行以方便在失败后通过输出查找原因。
打开终端，在"Edit"菜单中"Profile Preferences"中"Scrolling"选项卡，把“Limit scrollback to XX lines”前面的复选框去掉。 

3. 安装升级工具。
```
apt install mintupgrade
```

4. 检查安装
使用`mintupgrade check`命令模拟升级，遵照显示的说明。此命令临时将系统指向Linux Mint 19并估计升级会带来的影响，但不会真执行升级操作所以不会对现有系统造成任何影响。如果检查过程中显示有软件包有冲突，用`apt purge`命令删除掉该软件包并记下包名，此外也要注意记录重要但会在升级时被自动删除掉的包，以备升级成功后重新安装。
重复执行本步骤直至输出符合预期（意味着没有冲突和重要的包会受到升级影响）。

5. 下载更新并按装
```
mintupgrade download
mintupgrade upgrade
```

##### 参考资料
> - [How to upgrade to Linux Mint 19](https://community.linuxmint.com/tutorial/view/2416)

