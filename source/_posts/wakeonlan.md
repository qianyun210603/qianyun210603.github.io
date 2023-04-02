---
layout: post
title: 设置局域网和广域网远程开机（Wake on Lan）
date: '2022-11-22 17:25'
categories:
  - 其他
---

## 需求
因为需要随时远程访问家里计算机，因此希望出远门后即便出现家里停电等情况，家里计算机意外关机，也能够通过远程唤醒。

## 网上的资料
网上的解决方案：
1. Wake on LAN (WOL)[^fn1]: 本文的方案。通过向目标机发送一个魔幻数据包（Magic Packet）进行远程唤醒，需要主板和网卡支持。本意设计是在局域网内，通过端口映射等方法也可以在广域网生效（需要公网静态IP或者公网动态IP+DDNS）。
2. 各种开机棒等： 同样需要主板和网卡的支持，同时需要购买格外硬件，还要在硬件厂商的网站注册以让硬件厂商能直接访问对应硬件，安全性也是个顾虑。
3. 智能插座，一些DIY的能在线控制的继电器之类的设备。

考虑需要远程的连接的PC都支持WOL，研究采用WOL的方案。

## 实现

### 局域网唤醒

#### 主板BIOS设置
- 开机按`Del`（根据主板品牌可能是其他功能键，建议查阅主板说明书）进入BIOS设置，打开设备唤醒中的PCIE设备唤醒选项。根据品牌可能被称为
  - Boot on LAN
  - Wake on LAN
  - PME Event WakeUp,
  - Resume by MAC LAN
  - Wake-Up by PCI card
  - Wake Up On PCI PME
  - Power On by PCI Card
  - WakeUp by PME of PCI
  - Power On By PCI Devices
  - WakeUp by Onborad LAN
  - Resume By PCI or PCI-E Ddevice
  等等。
- 如果有的话，关闭Fast Boot或类似选项。
- 如果有的话，关闭EMP。

#### 目标机系统设置
###### Windows 10
- 首先，更新网卡驱动到最新版本。
  - 建议去主板/网卡的官网下载。Windows集成的更新并不及时。
- 搜索框搜“设备管理器（Device Manager）”并打开。
- 在“网络适配器（Network adapters）”下面找到有线网卡，右键菜单“属性（Properties）”。
  - 进入“高级（Advanced）”选项卡，开启“Wait for Link”, “Wake on Link Settings”, “Wake on Magic Packet”, “Wake on Pattern Match”。
  - 进入“电源管理（Power Management）”选项卡，把三个复选框全部选上。

###### Ubuntu 22.04
- 首先查看本机的各网卡名称和MAC地址
  ```bash
  $ ifconfig
  enp5s0: flags=4099<UP,BROADCAST,MULTICAST>  mtu xxxx
          ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
          RX packets 0  bytes 0 (0.0 B)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 0  bytes 0 (0.0 B)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
          device memory 0xfc000000-fc01ffff

  enpxxx: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 192.168.x.xxx  netmask 255.255.255.0  broadcast 192.168.x.255
          inet6 xxxx:1b43  prefixlen 64  scopeid 0x0<global>
          inet6 fe80::271e:d40:2e18:9c6  prefixlen 64  scopeid 0x20<link>
          inet6 xxxx:7bb2  prefixlen 64  scopeid 0x0<global>
          inet6 xxxx:9b94  prefixlen 64  scopeid 0x0<global>
          inet6 xxxx:99f1  prefixlen 64  scopeid 0x0<global>
          ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
          RX packets 53552819  bytes 30767884934 (30.7 GB)
          RX errors 0  dropped 671  overruns 0  frame 0
          TX packets 72797481  bytes 71048847776 (71.0 GB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

  ...
  ```
  从上面信息查出本机有线网网卡名称为`enpxxx`。
- 然后使用`ethtool`查看是否开启了WOL。
  ```bash
  $ sudo ethtool enpxxx | grep Wake
	Supports Wake-on: pumbg
	Wake-on: g
  ```
  这样表示支持WOL且已经开启。
  - 如果显示`Wake-on: d`则表示WOL被禁用，需使用命令`sudo ethtool -s enpxxx wol g`开启。
  - 如未安装`ethtool`可以`sudo apt install ethtool`安装。

#### 控制软件

###### Android平台
笔者使用的是Wake on LAN v1.35，官网已不可考，吾爱破解有个[帖子](https://www.52pojie.cn/thread-1609028-1-1.html)提供了apk文件的下载。
点"+"按钮新建，选择“ENTER MANUALLY”，起个名字填入MAC地址即可。填写完成后`ADD DEVICE`，就会显示在列表里。
![](/images/2022/11/wol_lan_control_android.jpg)![](/images/2022/11/wol_mainlist_control_android.jpg)

将手机连接到目标机同一局域网内的WiFi上，点击条目即可唤醒。

### 广域网唤醒

#### 公网网址
国内运营商对家用网络一般给的都是内网IP，具体可以到运营商给的光猫里查看。如果申请了桥接，那么需要到连接光猫的路由器里查看。
 - 如果网址以10.或者172.开头，则多半是内网IP。
 - 完全确认可以去[ip138](https://www.ip138.com/)等网站上查看本机对外的IP，如果和光猫、路由的IP一致则是公网IP，否则是内网IP。

#### 动态域名
获取公网IP之后，需要到路由器所支持的动态域名商那里申请一个动态域名。如对联通的光猫，看他支持ORAY（其实是向日葵、花生壳等几家的合体）就去ORAY注册个账户，然后申请个花生壳的动态域名，申请后在路由里面按要求填写申请到的域名，ORAY的用户名，密码。

#### 路由器设置
- 设备的MAC地址与路由器分配的IP绑定。
![](/images/2022/11/ipmac.png)
- 路由器需要设置端口转发，将外部发送过来的Magic Packet转发到目标机的端口9。


[^fn1]: [wiki.wireshak: WakeOnLAN](https://wiki.wireshark.org/WakeOnLAN)
