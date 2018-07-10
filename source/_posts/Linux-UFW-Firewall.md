---
title: Linux上使用ufw进行端口管理
date: 2018-07-05 21:37:49
categories:
- Linux
---

#### 安装
```
apt-get install ufw
```

### 启用
```
ufw enable
```

### 增加规则允许访问端口
```
ufw allow 80
```

### 删除规则允许访问端口
```
ufw delete allow 80
```




