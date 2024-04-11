---
title: 发布Python包到PyPi上
date: 2024-01-21T15:30:21+08:00
toc: true
tags: 
categories: 
- Programming
- Python
---

1. 确保Python包至少要有写好的`setup.py`(或`setup.cfg`)、`README.md`和一个协议文件`LICENSE`。同时包名不得与Pypi上已有的包冲突。
2. 安装必要的工具：
   ```bash
   pip install setuptools wheel twine
   ```
   如果使用`setup.cfg`还需要`build`包：
   ```bash
   pip install build
   ```
3. 更新`setup.py`或`setup.cfg`里的版本号，PyPi不允许重复版本号。
   ```
   version = X.Y.Z
   ```
4. 将版本提交到版本控制系统（如Github）,并创建相应的`tag`
   ```bash
   git commit -am "Release version X.Y.Z"
   git tag X.Y.Z
   git push origin X.Y.Z
   ```
5. 创建源代码和wheel文件发布
    ```bash
    python -m build
    ```

6. 注册PyPi账号于https://pypi.org/，按提示步骤操作，并开启2FA验证（可以用Google或微软的Authenticator APP）。
7. PyPi现在强制使用token验证，所以需要在"Account Setting"里面生成一个API token。然后在用户目录里面创建一个`.pypirc`文件填充如下内容（`username`就是那个带前后下划线的"__token__",`password`换成系统生成的API token）。
    ```
    [pypi]
    username = __token__
    password = pypi-**********
    ```

8. 使用`twine`上传
   ```bash
    twine upload dist/*
   ```