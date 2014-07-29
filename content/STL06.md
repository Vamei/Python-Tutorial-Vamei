#Python标准库06 子进程（subprocess包）

这里的内容以Linux进程基础和Linux文本流为基础。`subprocess`包主要功能是执行外部的命令和程序。比如说，我需要使用`wget`下载文件。我在Python中调用`wget`程序。从这个意义上来说，`subprocess`的功能与shell类似。

##subprocess以及常用的封装函数

当我们运行python的时候，我们都是在创建并运行一个进程。正如我们在Linux进程基础中介绍的那样，一个进程可以`fork`一个子进程，并让这个子进程`exec`另外一个程序。在Python中，我们通过标准库中的`subprocess`包来`fork`一个子进程，并运行一个外部的程序（`fork`，`exec`见Linux进程基础）。

`subprocess`包中定义有数个创建子进程的函数，这些函数分别以不同的方式创建子进程，所以我们可以根据需要来从中选取一个使用。另外`subprocess`还提供了一些管理标准流（standard stream）和管道（pipe）的工具，从而在进程间使用文本通信。

使用`subprocess`包中的函数创建子进程的时候，要注意：

1) 在创建子进程之后，父进程是否暂停，并等待子进程运行。

2) 函数返回什么？

3) 当`returncode`不为`0`时，父进程如何处理。

```python
subprocess.call()
```

父进程等待子进程完成。返回退出信息（`returncode`，相当于exit code，见Linux进程基础）。

```python
subprocess.check_call()
```

父进程等待子进程完成。返回`0`。

检查退出信息，如果`returncode`不为`0`，则举出错误`subprocess.CalledProcessError`，该对象包含有`returncode`属性，可用`try...except...`来检查（见Python错误处理）。

```python
subprocess.check_output()
```

父进程等待子进程完成。返回子进程向标准输出的输出结果。

检查退出信息，如果`returncode`不为`0`，则举出错误`subprocess.CalledProcessError`，该对象包含有`returncode`属性和`output`属性，`output`属性为标准输出的输出结果，可用`try...except...`来检查。

这三个函数的使用方法相类似，我们以`subprocess.call()`来说明：

```python
import subprocess
rc = subprocess.call(["ls", "-l"])
```

我们将程序名（`ls`）和所带的参数（`-l`）一起放在一个表中传递给`subprocess.call()`

可以通过一个shell来解释一整个字符串：

```python
import subprocess
out = subprocess.call("ls -l", shell=True)
out = subprocess.call("cd ..", shell=True)
```

我们使用了`shell=True`这个参数。这个时候，我们使用一整个字符串，而不是一个表来运行子进程。Python将先运行一个shell，再用这个shell来解释这整个字符串。

shell命令中有一些是shell的内建命令，这些命令必须通过shell运行，`$cd`。`shell=True`允许我们运行这样一些命令。

##Popen()

实际上，我们上面的三个函数都是基于`Popen()`的封装（wrapper）。这些封装的目的在于让我们容易使用子进程。当我们想要更个性化我们的需求的时候，就要转向`Popen`类，该类生成的对象用来代表子进程。

与上面的封装不同，`Popen`对象创建后，主程序不会自动等待子进程完成。我们必须调用对象的`wait()`方法，父进程才会等待（也就是阻塞block）：

```python
import subprocess
child = subprocess.Popen(["ping", "-c", "5", "www.google.com"])
print("parent process")
```

从运行结果中看到，父进程在开启子进程之后并没有等待`child`的完成，而是直接运行`print`。

对比等待的情况：

```python
import subprocess
child = subprocess.Popen(["ping", "-c", "5", "www.google.com"])
child.wait()
print("parent process")
``` 

此外，你还可以在父进程中对子进程进行其它操作，比如我们上面例子中的`child`对象：

```python
child.poll()           # 检查子进程状态
child.kill()           # 终止子进程
child.send_signal()    # 向子进程发送信号
child.terminate()      # 终止子进程
```

子进程的PID存储在`child.pid`。

##子进程的文本流控制

（沿用`child`子进程）子进程的标准输入，标准输出和标准错误也可以通过如下属性表示：

`child.stdin`

`child.stdout`

`child.stderr`

我们可以在`Popen()`建立子进程的时候改变标准输入、标准输出和标准错误，并可以利用`subprocess.PIPE`将多个子进程的输入和输出连接在一起，构成管道（pipe）：

```python
import subprocess
child1 = subprocess.Popen(["ls", "-l"], stdout=subprocess.PIPE)
child2 = subprocess.Popen(["wc"], stdin=child1.stdout, stdout=subprocess.PIPE)
out = child2.communicate()
print(out)
```

`subprocess.PIPE`实际上为文本流提供一个缓存区。`child1`的`stdout`将文本输出到缓存区，随后`child2`的`stdin`从该`PIPE`中将文本读取走。`child2`的输出文本也被存放在`PIPE`中，直到`communicate()`方法从`PIPE`中读取出`PIPE`中的文本。

要注意的是，`communicate()`是`Popen`对象的一个方法，该方法会阻塞父进程，直到子进程完成。

我们还可以利用`communicate()`方法来使用`PIPE`给子进程输入：

```python
import subprocess
child = subprocess.Popen(["cat"], stdin=subprocess.PIPE)
child.communicate("vamei")
```

我们启动子进程之后，`cat`会等待输入，直到我们用`communicate()`输入`"vamei"`。

通过使用`subprocess`包，我们可以运行外部程序。这极大的拓展了Python的功能。如果你已经了解了操作系统的某些应用，你可以从Python中直接调用该应用（而不是完全依赖Python），并将应用的结果输出给Python，并让Python继续处理。shell的功能（比如利用文本流连接各个应用），就可以在Python中实现。

##总结

`subprocess.call`、`subprocess.check_call()`、`subprocess.check_output()`

`subprocess.Popen()`、`subprocess.PIPE`

`Popen.wait()`、`Popen.communicate()`