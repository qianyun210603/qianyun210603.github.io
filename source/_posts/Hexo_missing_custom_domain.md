---
title: Hexo部署到GitHub Pages后设置的自定义域名丢失
date: 2023-03-23 21:02:14
categories:
- Hexo
---

### 问题
基于Hexo的静态博客托管在GitHub Pages上，并且在`Settings->Pages`里面绑定了自定义域名。
![](Hexo_missing_custom_domain\1.png)


但每次部署（运行`hexo deploy`后设置的自定义域名都会被自动清空。

### 解决
在`source`文件夹下建立文本文件`CNAME`（无后缀名），内容为自定义域名。