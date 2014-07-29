#Python标准库10 多进程初步（multiprocessing包）

我们已经见过了使用`subprocess`包来创建子进程，但这个包有两个很大的局限性：

1) 我们总是让subprocess运行外部的程序，而不是运行一个Python脚本内部编写的函数。

2) 进程间只通过管道进行文本交流。以上限制了我们将`subprocess`包应用到更广泛的多进程任务。

（这样的比较实际是不公平的，因为`subprocessing`本身就是设计成为一个shell，而不是一个多进程管理包）

##threading和multiprocessing

（请尽量先阅读Python多线程与同步）

`multiprocessing`包是Python中的多进程管理包。与`threading.Thread`类似，它可以利用`multiprocessing.Process`对象来创建一个进程。该进程可以运行在Python程序内部编写的函数。该`Process`对象与Thread对象的用法相同，也有`start()`、`run()`、`join()`的方法。

此外`multiprocessing`包中也有`Lock`/`Event`/`Semaphore`/`Condition`类（这些对象可以像多线程那样，通过参数传递给各个进程），用以同步进程，其用法与`threading`包中的同名类一致。所以，`multiprocessing`的很大一部份与`threading`使用同一套API，只不过换到了多进程的情境。

但在使用这些共享API的时候，我们要注意以下几点：

在UNIX平台上，当某个进程终结之后，该进程需要被其父进程调用`wait`，否则进程成为僵尸进程（Zombie）。所以，有必要对每个`Process`对象调用`join()`方法（实际上等同于`wait`）。对于多线程来说，由于只有一个进程，所以不存在此必要性。

`multiprocessing`提供了`threading`包中没有的IPC（比如Pipe和Queue），效率上更高。应优先考虑Pipe和Queue，避免使用`Lock`/`Event`/`Semaphore`/`Condition`等同步方式（因为它们占据的不是用户进程的资源）。

多进程应该避免共享资源。在多线程中，我们可以比较容易地共享资源，比如使用全局变量或者传递参数。在多进程情况下，由于每个进程有自己独立的内存空间，以上方法并不合适。此时我们可以通过共享内存和`Manager`的方法来共享资源。但这样做提高了程序的复杂度，并因为同步的需要而降低了程序的效率。

`Process.PID`中保存有PID，如果进程还没有`start()`，则PID为`None`。

我们可以从下面的程序中看到`Thread`对象和`Process`对象在使用上的相似性与结果上的不同。各个线程和进程都做一件事：打印PID。但问题是，所有的任务在打印的时候都会向同一个标准输出（stdout）输出。这样输出的字符会混合在一起，无法阅读。使用`Lock`同步，在一个任务输出完成之后，再允许另一个任务输出，可以避免多个任务同时向终端输出。

```python
# Similarity and difference of multi thread vs. multi process
# Written by Vamei

import os
import threading
import multiprocessing

# worker function
def worker(sign, lock):
    lock.acquire()
    print(sign, os.getpid())
    lock.release()

# Main
print('Main:', os.getpid())

# Multi-thread
record = []
lock  = threading.Lock()
for i in range(5):
    thread = threading.Thread(target=worker, args=('thread', lock))
    thread.start()
    record.append(thread)

for thread in record:
    thread.join()

# Multi-process
record = []
lock = multiprocessing.Lock()
for i in range(5):
    process = multiprocessing.Process(target=worker, args=('process', lock))
    process.start()
    record.append(process)

for process in record:
    process.join()
```

所有`Thread`的PID都与主程序相同，而每个`Process`都有一个不同的PID。

（练习：使用`mutiprocessing`包将Python多线程与同步中的多线程程序更改为多进程程序）

##Pipe和Queue

正如我们在Linux多线程中介绍的管道PIPE和消息队列message queue，`multiprocessing`包中有`Pipe`类和`Queue`类来分别支持这两种IPC机制。`Pipe`和`Queue`可以用来传送常见的对象。

1) `Pipe`可以是单向（half-duplex），也可以是双向（duplex）。我们通过`mutiprocessing.Pipe(duplex=False)`创建单向管道（默认为双向）。一个进程从`Pipe`一端输入对象，然后被`Pipe`另一端的进程接收，单向管道只允许管道一端的进程输入，而双向管道则允许从两端输入。

下面的程序展示了`Pipe`的使用：

```python
# Multiprocessing with Pipe
# Written by Vamei

import multiprocessing as mul

def proc1(pipe):
    pipe.send('hello')
    print('proc1 rec:', pipe.recv())

def proc2(pipe):
    print('proc2 rec:', pipe.recv())
    pipe.send('hello, too')

# Build a pipe
pipe = mul.Pipe()

# Pass an end of the pipe to process 1
p1   = mul.Process(target=proc1, args=(pipe[0],))
# Pass the other end of the pipe to process 2
p2   = mul.Process(target=proc2, args=(pipe[1],))
p1.start()
p2.start()
p1.join()
p2.join()
```

这里的`Pipe`是双向的。

`Pipe`对象建立的时候，返回一个含有两个元素的表，每个元素代表`Pipe`的一端（`Connection`对象）。我们对`Pipe`的某一端调用`send()`方法来传送对象，在另一端使用`recv()`来接收。

2) `Queue`与`Pipe`相类似，都是先进先出的结构。但`Queue`允许多个进程放入，多个进程从队列取出对象。`Queue`使用`mutiprocessing.Queue(maxsize)`创建，`maxsize`表示队列中可以存放对象的最大数量。
下面的程序展示了`Queue`的使用：

```python
# Written by Vamei
import os
import multiprocessing
import time
#==================
# input worker
def inputQ(queue):
    info = str(os.getpid()) + '(put):' + str(time.time())
    queue.put(info)

# output worker
def outputQ(queue, lock):
    info = queue.get()
    lock.acquire()
    print (str(os.getpid()) + '(get):' + info)
    lock.release()
#===================
# Main
record1 = []   # store input processes
record2 = []   # store output processes
lock  = multiprocessing.Lock()    # To prevent messy print
queue = multiprocessing.Queue(3)

# input processes
for i in range(10):
    process = multiprocessing.Process(target=inputQ, args=(queue,))
    process.start()
    record1.append(process)

# output processes
for i in range(10):
    process = multiprocessing.Process(target=outputQ, args=(queue, lock))
    process.start()
    record2.append(process)

for p in record1:
    p.join()

queue.close()  # No more object will come, close the queue

for p in record2:
    p.join()
```

一些进程使用`put()`在`Queue`中放入字符串，这个字符串中包含PID和时间。另一些进程从`Queue`中取出，并打印自己的PID以及`get()`的字符串。

##总结

`Process`、`Lock`、`Event`、`Semaphore`、`Condition`

`Pipe`、`Queue`