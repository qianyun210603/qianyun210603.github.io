---
title: 嵌入的Qt对话框输入一些东西相关(.exec/.show etc)
date: 2024-11-09T12:24:29+08:00
toc: true
tags: 
categories: 
- Programming
- Python
---

基本框架
```python

# some content load from remote data service
# ... code ignored ...

app = QApplication([])
dialog = ContentEditDialog(original_content)
dialog.exec()
# dialog.show()
# app.exec()

updated_content = dialog.collect_edited_content()

print(updated_content)
```

- 只有`dialog.exec()`：程序阻塞，编辑生效。
- 只有`dialog.show()`: 程序不阻塞，对话框一闪即逝，无法编辑。
- `dialog.exec()`和`app.exec()`: 对话框关闭后程序卡死。
- `dialog.show()`和`app.exec()`: 程序阻塞，编辑生效。
