# pyQtGraph

## pyQtGraph

PyQtGraph 是基于 PyQt / PySide 和 numpy 构建的纯 Python 图形和 GUI 库。Qt图形界面框架完美融合。目前打算使用它

```bash
pip install pyqt6 -i https://pypi.mirrors.ustc.edu.cn/simple/
```

```python
import pyqtgraph.examples
pyqtgraph.examples.run()
```

```bash
https://github.com/pyqtgraph/pyqtgraph/tree/master/pyqtgraph/examples
```

## 线程

Qt应用程序基于事件（用户交互、信号和定时器）运行。发生的任何事件会进入事件队列中，然后按照顺序执行。执行事件循环的代码为:

```c
app = QApplication([])
window = MainWindow()
app.exec()
```

默认情况下，触发任何事件的执行程序和都在GUI线程中，导致执行程序运行和GUI的交互不能同时进行。

您正在做的事情很简单，并且快速将控制权返回给 GUI 循环，则用户将察觉不到这种冻结。但是，如果您需要执行长时间运行的任务，例如打开/写入大文件、下载一些数据或渲染一些复杂的图像，就会出现问题。对于您的用户来说，应用程序将显得没有响应（因为确实如此）

qt 提供两种类来实现多线程运行`QRunnable`，`QThreadPool`

* `QRunnable`：具体实现
* `QThreadPool`：传递到qt中

传回线程运行的状态和数据

![result](../picture/PyQtGraph/result.gif)
