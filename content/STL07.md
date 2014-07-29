#Python标准库07 信号（signal包、部分os包）

在了解了Linux的信号基础之后，Python标准库中的`signal`包就很容易学习和理解。`signal`包负责在Python程序内部处理信号，典型的操作包括预设信号处理函数，暂停并等待信号，以及定时发出`SIGALRM`等。要注意，`signal`包主要是针对UNIX平台（比如Linux、OS X），而Windows内核中由于对信号机制的支持不充分，所以在Windows上的Python不能发挥信号系统的功能。

##定义信号名

`signal`包定义了各个信号名及其对应的整数，比如：

```python
import signal
print signal.SIGALRM
print signal.SIGCONT
```

Python所用的信号名和Linux一致。你可以通过

```bash
$man 7 signal
```

来查询。

##预设信号处理函数

`signal`包的核心是使用`signal.signal()`函数来预设（register）信号处理函数，如下所示：

```python
singnal.signal(signalnum, handler)
```

`signalnum`为某个信号，`handler`为该信号的处理函数。我们在信号基础里提到，进程可以无视信号，可以采取默认操作，还可以自定义操作。当`handler`为`signal.SIG_IGN`时，信号被无视（ignore）。当`handler`为`singal.SIG_DFL`，进程采取默认操作（default）。当`handler`为一个函数名时，进程采取函数中定义的操作。

```python
import signal
# Define signal handler function
def myHandler(signum, frame):
    print('I received: ', signum)

# register signal.SIGTSTP's handler 
signal.signal(signal.SIGTSTP, myHandler)
signal.pause()
print('End of Signal Demo')
```

在主程序中，我们首先使用`signal.signal()`函数来预设信号处理函数。然后我们执行`signal.pause()`来让该进程暂停以等待信号，以等待信号。当信号`SIGUSR1`被传递给该进程时，进程从暂停中恢复，并根据预设，执行`SIGTSTP`的信号处理函数`myHandler()`。`myHandler`的两个参数一个用来识别信号（`signum`），另一个用来获得信号发生时，进程栈的状况（stack frame）。这两个参数都是由`signal.singnal()`函数来传递的。

上面的程序可以保存在一个文件中（比如test.py）。我们使用如下方法运行：

```bash
$python test.py
```

以便让进程运行。当程序运行到`signal.pause()`的时候，进程暂停并等待信号。此时，通过按下`CTRL+Z`向该进程发送`SIGTSTP`信号。我们可以看到，进程执行了`myHandle()`函数，随后返回主程序，继续执行。（当然，也可以用`$ps`查询process ID，再使用`$kill`来发出信号。）

（进程并不一定要使用`signal.pause()`暂停以等待信号，它也可以在进行工作中接受信号，比如将上面的`signal.pause()`改为一个需要长时间工作的循环。）

我们可以根据自己的需要更改`myHandler()`中的操作，以针对不同的信号实现个性化的处理。

##定时发出SIGALRM信号

一个有用的函数是`signal.alarm()`，它被用于在一定时间之后，向进程自身发送`SIGALRM`信号：

```python
import signal
# Define signal handler function
def myHandler(signum, frame):
    print("Now, it's the time")
    exit()

# register signal.SIGALRM's handler 
signal.signal(signal.SIGALRM, myHandler)
signal.alarm(5)
while True:
    print('not yet')
```

我们这里用了一个无限循环以便让进程持续运行。在`signal.alarm()`执行`5`秒之后，进程将向自己发出`SIGALRM`信号，随后，信号处理函数`myHandler`开始执行。

##发送信号

`signal`包的核心是设置信号处理函数。除了`signal.alarm()`向自身发送信号之外，并没有其他发送信号的功能。但在`os`包中，有类似于linux的`kill`命令的函数，分别为：

```python
os.kill(pid, sid)
os.killpg(pgid, sid)
```

分别向进程和进程组（见Linux进程关系）发送信号。`sid`为信号所对应的整数或者`singal.SIG*`。

实际上`signal`、`pause、`kill`和`alarm`都是Linux应用编程中常见的C库函数，在这里，我们只不过是用Python语言来实现了一下。实际上，Python 的解释器是使用C语言来编写的，所以有此相似性也并不意外。此外，在Python 3.4中，`signal`包被增强，信号阻塞等功能被加入到该包中。我们暂时不深入到该包中。

##总结

`signal.SIG*`

`signal.signal()`

`signal.pause()`

`signal.alarm()`