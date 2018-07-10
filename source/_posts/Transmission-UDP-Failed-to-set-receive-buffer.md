---
title: Linux下Transmission的“UDP Failed to set receive buffer”错误
date: 2018-07-09 21:09:06
tags:
categories:
- Linux
- transmission
---
在Ubuntu 18.04的VPS上运行`transmission-daemon`报如下错误：
```
Jul 06 13:50:43 ubuntu-s-1vcpu-2gb-tor1-01 systemd[1]: Started Transmission BitTorrent Daemon.
Jul 06 13:50:44 ubuntu-s-1vcpu-2gb-tor1-01 transmission-daemon[29663]: [2018-07-06 13:50:44.429] UDP Failed to set receive buffer: requested 4194304, got 425984 (tr-udp.c:84)
Jul 06 13:50:44 ubuntu-s-1vcpu-2gb-tor1-01 transmission-daemon[29663]: [2018-07-06 13:50:44.430] UDP Failed to set send buffer: requested 1048576, got 425984 (tr-udp.c:95)
Jul 06 13:50:44 ubuntu-s-1vcpu-2gb-tor1-01 transmission-daemon[29663]: [2018-07-06 13:50:44.430] UDP Failed to set receive buffer: requested 4194304, got 425984 (tr-udp.c:84)
Jul 06 13:50:44 ubuntu-s-1vcpu-2gb-tor1-01 transmission-daemon[29663]: [2018-07-06 13:50:44.430] UDP Failed to set send buffer: requested 1048576, got 425984 (tr-udp.c:95)
```

解决方案如下：
```bash
sysctl -w net.core.rmem_max=8388608
sysctl -w net.core.wmem_max=8388608
```