---
title: properly_terminate_qthread_in_pyqt
date: 2024-09-16T15:42:13+08:00
toc: true
tags: 
categories: 
 - Programming
 - Python
---

如果程序中使用了`QThread`，需要正确销毁，否则会在程序退出时崩溃。

1. 定义`finished`信号在worker类里面；
2. 在`finished`对应的槽函数里面调用`QThread`的`quit()`和`wait()`函数；
3. `QThread`自带`finished`函数，将其链接到`QThread`对应的`deleteLater()`函数上。

示例：
```python
import sys
from PyQt6.QtCore import QThread, pyqtSignal, QObject, pyqtSlot, QMetaObject
from PyQt6.QtWidgets import QApplication, QWidget, QPushButton, QVBoxLayout
from pydev_ipython.qt import QtCore


class Worker(QObject):

    finished = pyqtSignal()

    def __init__(self, parent=None):
        super(Worker, self).__init__(parent)

    def run(self):
        import time
        for i in range(1, 11):
            time.sleep(1)
            print(i)
        self.finished.emit()


class MainApp(QWidget):
    def __init__(self):
        super(MainApp, self).__init__()
        self.setWindowTitle("QThread Example")
        layout = QVBoxLayout()
        self.setLayout(layout)
        self.setGeometry(100, 100, 400, 200)
        self.btn = QPushButton("Start Thread")
        layout.addWidget(self.btn)
        self.btn.clicked.connect(self.start_thread)

    def start_thread(self):
        self.worker = Worker(parent=self)
        self.worker.setObjectName("worker1")
        self.thread = QThread()
        self.worker.moveToThread(self.thread)
        self.thread.started.connect(self.worker.run)
        # self.worker.finished.connect(self.on_worker_finished)
        self.worker.finished.connect(self.worker.deleteLater)
        self.thread.finished.connect(self.thread.deleteLater)
        QMetaObject.connectSlotsByName(self)
        self.thread.start()

    @pyqtSlot()
    def on_worker1_finished(self):
        print("Worker finished")
        self.thread.quit()
        self.thread.wait()
        print("Thread finished")


if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = MainApp()
    window.show()
    sys.exit(app.exec())
```



## 参考资料
> - []()
> - []()
