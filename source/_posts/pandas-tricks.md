---
title: 【Python知识库】pandas tricks
date: 2018-06-26 21:02:12
tags: python, pandas
categories:
- Programming
- Python
---
1. Pandas显示设置（最大行数等）
   ```python
    pd.set_option('max_rows', 500)
    #param choices
    # display.height
    # width
    # max_rows
    # max_columns
    ```

2. 将Timestamp和Timedelta相加时报AttributeError: 'NoneType' object has no attribute 'total_seconds'
解决：重新安装pandas
```bash
pip install --force-reinstall pandas
```
