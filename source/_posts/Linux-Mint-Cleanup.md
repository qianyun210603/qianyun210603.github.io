---
title: Linux Mint Cleanup
date: 2018-06-26 21:02:09
tags:
---
Clean up old kernel
```bash
uname -r #check current using kernel
dpkg --get-selections |grep linux # list installed kernels
apt purge xxx # remove
```