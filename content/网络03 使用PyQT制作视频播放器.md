# Python应用03 使用PyQT制作视频播放器

最近研究了Python的两个GUI包，Tkinter和PyQT。这两个GUI包的底层分别是Tcl/Tk和QT。相比之下，我觉得PyQT使用起来更加方便，功能也相对丰富。这一篇用PyQT实现一个视频播放器，并借此来说明PyQT的基本用法。

## 视频播放器

先把已经完成的代码放出来。代码基于Python 3.5：
```py
import time
import sys

from PyQt4 import QtGui, QtCore
from PyQt4.phonon import Phonon


class PollTimeThread(QtCore.QThread):
    """
    This thread works as a timer.
    """
    update = QtCore.pyqtSignal()

    def __init__(self, parent):
        super(PollTimeThread, self).__init__(parent)

    def run(self):
        while True:
            time.sleep(1)
            if self.isRunning():
                # emit signal
                self.update.emit()
            else:
                return

class Window(QtGui.QWidget):
    def __init__(self):
        QtGui.QWidget.__init__(self)

        # media
        self.media = Phonon.MediaObject(self)
        self.media.stateChanged.connect(self.handleStateChanged)
        self.video = Phonon.VideoWidget(self)
        self.video.setMinimumSize(200, 200)
        self.audio = Phonon.AudioOutput(Phonon.VideoCategory, self)
        Phonon.createPath(self.media, self.audio)
        Phonon.createPath(self.media, self.video)

        # control button
        self.button = QtGui.QPushButton('选择文件', self)
        self.button.clicked.connect(self.handleButton)

        # for display of time lapse
        self.info = QtGui.QLabel(self)

        # layout
        layout = QtGui.QGridLayout(self)
        layout.addWidget(self.video, 1, 1, 3, 3)
        layout.addWidget(self.info, 4, 1, 1, 3)
        layout.addWidget(self.button, 5, 1, 1, 3)

        # signal-slot, for time lapse
        self.thread = PollTimeThread(self)
        self.thread.update.connect(self.update)

    def update(self):
        # slot
        lapse = self.media.currentTime()/1000.0
        self.info.setText("%4.2f 秒" % lapse)

    def startPlay(self):
        if self.path:
            self.media.setCurrentSource(Phonon.MediaSource(self.path))

            # use a thread as a timer
            self.thread = PollTimeThread(self)
            self.thread.update.connect(self.update)
            self.thread.start()
            self.media.play()

    def handleButton(self):
        if self.media.state() == Phonon.PlayingState:
            self.media.stop()
            self.thread.terminate()
        else:
            self.path = QtGui.QFileDialog.getOpenFileName(self, self.button.text())
            self.startPlay()

    def handleStateChanged(self, newstate, oldstate):
        if newstate == Phonon.PlayingState:
            self.button.setText('停止')
        elif (newstate != Phonon.LoadingState and
              newstate != Phonon.BufferingState):
            self.button.setText('选择文件')
            if newstate == Phonon.ErrorState:
                source = self.media.currentSource().fileName()
                print ('错误：不能播放：', source.toLocal8Bit().data())
                print ('  %s' % self.media.errorString().toLocal8Bit().data())


if __name__ == '__main__':
    app = QtGui.QApplication(sys.argv)
    app.setApplicationName('视频播放')
    window = Window()
    window.show()
    sys.exit(app.exec_())

```

代码实现了一个有GUI窗口的应用，用来播放视频文件。视频播放利用了PyQT中的Phonon模块。此外，还有一个进程每隔一秒发出一个信号。窗口在接收到信号后，更新视频播放的时间。这个应用的效果如下：

![](413416-20161206231212694-1821975955.png)

测试运行环境为Mac OSX El Capitan。

## 视图部分

写完这个代码之后，我发现这个代码虽然简单，但涉及了几个重要机制，可以用PyQT的练习题。下面对代码进行一些简要的说明，首先是主程序部分：
```
app = QtGui.QApplication(sys.argv)
...
window = Window()
window.show()
sys.exit(app.exec_())
```
在PyQT程序中，QApplication是最上层的对象，指代整个GUI应用。我们在程序的一开始创建了一个应用对象，在程序最后调用exec_()来运行这个应用。sys.exit()用来要求应用的主循环结束后干净地退出程序。PyQT程序的开始和结尾都是类似的固定套路。关键就在于其间定义的QWidget对象。

我们自定义的Window类继承自QWidget。其实QWidget是所有用户界面对象的基类，并不单单指代一个窗口。表格、输入框、按钮都继承自QWidget。在一个Window对象中，我们还组合有QPushButton和QLabel这样的对象，分别代表一个按钮和一个文本框。它们通过QGridLayout的方式，布局在Window的界面上，即下面一部分代码：

```py
# layout
layout = QtGui.QGridLayout(self)
...
layout.addWidget(self.info, 4, 1, 1, 3)
layout.addWidget(self.button, 5, 1, 1, 3)
```
QGridLayout把界面分成网格，并把某个视图对象附着在特定的网格位置。比如说，addWidget()(self.info, 4, 1, 1, 3)表示把一个文本框对象放在第4排、第1列的位置。该文本框纵向将占据1排，横向占据3列。这样，上下层视图的位置关系就通过布局确定了下来。除了网格式的布局，PyQT还支持其他形式的布局，如横向堆砌、纵向堆砌等等，可以进一步了解。

除了QWidget，PyQT还提供了常用的对话框，如：
```py
self.path = QtGui.QFileDialog.getOpenFileName(self, self.button.text())
```
这里的QFileDialog对话框用于选择文件。对话框将访问所选文件的路径。除了文件选择，对话框还有确认对话框、文件输入对话框、色彩对话框。这些对话框实现了不少常用的GUI输入功能。通过利用这些对话框，可以减少程序员从头开发的工作量。

## 多线程

GUI界面的主线程通常留给应用做主循环。其他的很多工作要通过其他的线程来完成。PyQT多线程编程很简单，只需要重写QThread的run()方法就可以了：
```py
class PollTimeThread(QtCore.QThread):
    def __init__(self, parent):
        super(PollTimeThread, self).__init__(parent) 
    def run(self):
        ...
```
创建线程后，只需要调用start()方法，就可以运行：
```py
self.thread = PollTimeThread()  
...
self.thread.start()       # 启动线程  
...
self.thread.terminate()   # 终止线程
```

## 信号与槽

GUI经常要用到异步处理。比如说点击某个按钮，然后调用相应的回调函数。QT的“信号与槽”(signal-slot)机制就是为了解决异步处理问题。我们在线程中创建了信号，并通过emit()方法来发出信号：
```py
class PollTimeThread(QtCore.QThread):
    """
    This thread works as a timer.
    """
    update = QtCore.pyqtSignal()

    def __init__(self, parent):
        super(PollTimeThread, self).__init__(parent)

    def run(self):
        while True:
            time.sleep(1)
            if self.isRunning():
                # emit signal
                self.update.emit()
            else:
                return
```
有了信号，我们就可以给该信号连接到一个“槽”，其实就是对应于该信号的回调函数：

    self.thread.update.connect(self.update)

每当信号被发出时，“槽”就会被调用。在这个例子中，就是更新视频播放时间。QT中的“信号与槽”是普遍存在的机制。一些组建如按键，预设了“点击”这样的信号，可以直接对应到“槽”。如代码中的：

    self.button.clicked.connect(self.handleButton)

此外，Phonon是一个很好用的多媒体模块，使用方法也很简单，可以参考代码本身，这里不再赘述。
