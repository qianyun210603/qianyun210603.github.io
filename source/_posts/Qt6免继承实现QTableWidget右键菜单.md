---
title: Qt6免继承实现QTableWidget右键菜单
date: 2025-02-13T15:24:29+08:00
toc: true
tags: 
- C++
categories: 
- Programming
- Qt
---

通过事件过滤器（`eventFilter`）来实现右键菜单并显示行号。这种方法更加灵活，适合在已有的 `QTableWidget` 上添加功能，而无需修改或继承它。

以下是完整的示例代码：

```cpp
#include <QApplication>
#include <QTableWidget>
#include <QMenu>
#include <QContextMenuEvent>
#include <QMessageBox>
#include <QHBoxLayout>
#include <QWidget>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    // 创建主窗口
    QWidget window;
    QHBoxLayout *layout = new QHBoxLayout(&window);

    // 创建 QTableWidget
    QTableWidget *tableWidget = new QTableWidget(5, 3);
    tableWidget->setHorizontalHeaderLabels({"Column 1", "Column 2", "Column 3"});

    // 填充一些数据
    for (int row = 0; row < 5; ++row) {
        for (int col = 0; col < 3; ++col) {
            QTableWidgetItem *item = new QTableWidgetItem(tr("Item %1-%2").arg(row).arg(col));
            tableWidget->setItem(row, col, item);
        }
    }

    // 将 QTableWidget 添加到布局中
    layout->addWidget(tableWidget);
    window.setLayout(layout);

    // 安装事件过滤器
    tableWidget->viewport()->installEventFilter(&window);

    // 事件过滤器实现
    window.installEventFilter(tableWidget);
    QObject::connect(tableWidget, &QTableWidget::customContextMenuRequested, [tableWidget](const QPoint &pos) {
        QTableWidgetItem *item = tableWidget->itemAt(pos);
        if (item) {
            int row = item->row();

            // 创建右键菜单
            QMenu menu;
            QAction *showRowAction = menu.addAction(tr("Show Row Number"));
            QObject::connect(showRowAction, &QAction::triggered, [row]() {
                QMessageBox::information(nullptr, tr("Row Number"), tr("You clicked on row: %1").arg(row));
            });

            // 显示菜单
            menu.exec(tableWidget->viewport()->mapToGlobal(pos));
        }
    });

    // 启用右键菜单
    tableWidget->setContextMenuPolicy(Qt::CustomContextMenu);

    window.show();
    return app.exec();
}
```

### 代码说明：
1. **事件过滤器**：
   - 我们通过 `installEventFilter` 将事件过滤器安装到 `QTableWidget` 的视口（`viewport()`）上。
   - 在事件过滤器中，我们可以捕获右键点击事件并显示菜单。

2. **右键菜单**：
   - 使用 `QMenu` 创建一个右键菜单。
   - 通过 `QTableWidget::itemAt` 获取当前点击的单元格，并提取行号。
   - 添加一个菜单项来显示行号，点击后会弹出一个消息框。

3. **`setContextMenuPolicy`**：
   - 设置 `QTableWidget` 的上下文菜单策略为 `Qt::CustomContextMenu`，以便触发 `customContextMenuRequested` 信号。

4. **信号与槽**：
   - 连接 `customContextMenuRequested` 信号，在右键点击时显示菜单。

### 运行效果：
- 右键点击表格中的任意单元格，会弹出一个菜单。
- 选择 "Show Row Number" 菜单项，会弹出一个消息框显示当前点击的行号。

### 优点：
- 不需要继承 `QTableWidget`，适合在已有代码基础上扩展功能。
- 更加灵活，可以轻松应用到其他控件上。

如果不想修改现有代码结构，这种方法是非常合适的！