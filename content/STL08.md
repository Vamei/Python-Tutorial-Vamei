##Python标准库08 多线程与同步 (threading包)


 

Python主要通过标准库中的`threading`包来实现多线程。在当今网络时代，每个服务器都会接收到大量的请求。服务器可以利用多线程的方式来处理这些请求，以提高对网络端口的读写效率。Python是一种网络服务器的后台工作语言 (比如豆瓣网)，所以多线程也就很自然被Python语言支持。

(关于多线程的原理和C实现方法，请参考我之前写的Linux多线程与同步，要了解race condition, mutex和condition variable的概念)

 

##多线程售票以及同步

我们使用Python来实现Linux多线程与同步文中的售票程序。我们使用`mutex`(也就是Python中的`Lock`类对象) 来实现线程的同步:

```python
# A program to simulate selling tickets in multi-thread way
# Written by Vamei

import threading
import time
import os

# This function could be any function to do other chores.
def doChore():
    time.sleep(0.5)

# Function for each thread
def booth(tid):
    global i
    global lock
    while True:
        lock.acquire()                # Lock; or wait if other thread is holding the lock
        if i != 0:
            i = i - 1                 # Sell tickets
            print(tid,':now left:',i) # Tickets left
            doChore()                 # Other critical operations
        else:
            print("Thread_id",tid," No more tickets")
            os._exit(0)              # Exit the whole process immediately
        lock.release()               # Unblock
        doChore()                    # Non-critical operations

# Start of the main function
i    = 100                           # Available ticket number 
lock = threading.Lock()              # Lock (i.e., mutex)

# Start 10 threads
for k in range(10):
    new_thread = threading.Thread(target=booth,args=(k,))   # Set up thread; target: the callable (function) to be run, args: the argument for the callable 
    new_thread.start()                                      # run the thread
```

我们使用了两个全局变量，一个是`i`，用以储存剩余票数；一个是`lock`对象，用于同步线程对`i`的修改。此外，在最后的`for`循环中，我们总共设置了`10`个线程。每个线程都执行`booth()`函数。线程在调用`start()`方法的时候正式启动 (实际上，计算机中最多会有11个线程，因为主程序本身也会占用一个线程)。Python使用`threading.Thread`对象来代表线程，用`threading.Lock`对象来代表一个互斥锁 (mutex)。

有两点需要注意:

我们在函数中使用`global`来声明变量为全局变量，从而让多线程共享`i`和`lock` (在C语言中，我们通过将变量放在所有函数外面来让它成为全局变量)。如果不这么声明，由于`i`和`lock`是不可变数据对象，它们将被当作一个局部变量(参看Python动态类型)。如果是可变数据对象的话，则不需要`global`声明。我们甚至可以将可变数据对象作为参数来传递给线程函数。这些线程将共享这些可变数据对象。
我们在`booth`中使用了两个`doChore()`函数。可以在未来改进程序，以便让线程除了进行`i = i - 1`之外，做更多的操作，比如打印剩余票数，找钱，或者喝口水之类的。第一个`doChore()`依然在`Lock`内部，所以可以安全地使用共享资源 (critical operations, 比如打印剩余票数)。第二个`doChore()`时，`Lock`已经被释放，所以不能再去使用共享资源。这时候可以做一些不使用共享资源的操作 (non-critical operation, 比如找钱、喝水)。我故意让`doChore()`等待了`0.5`秒，以代表这些额外的操作可能花费的时间。你可以定义的函数来代替`doChore()`。
 

##OOP创建线程

上面的Python程序非常类似于一个面向过程的C程序。我们下面介绍如何通过面向对象 (OOP， object-oriented programming，参看Python面向对象的基本概念和Python面向对象的进一步拓展) 的方法实现多线程，其核心是继承`threading.Thread`类。我们上面的`for`循环中已经利用了`threading.Thread()`的方法来创建一个`Thread`对象，并将函数`booth()`以及其参数传递给改对象，并调用`start()`方法来运行线程。OOP的话，通过修改`Thread`类的`run()`方法来定义线程所要执行的命令。

```python
# A program to simulate selling tickets in multi-thread way
# Written by Vamei

import threading
import time
import os

# This function could be any function to do other chores.
def doChore():
    time.sleep(0.5)

# Function for each thread
class BoothThread(threading.Thread):
    def __init__(self, tid, monitor):
        self.tid          = tid
        self.monitor = monitor
        threading.Thread.__init__(self)
    def run(self):
        while True:
            monitor['lock'].acquire()                          # Lock; or wait if other thread is holding the lock
            if monitor['tick'] != 0:
                monitor['tick'] = monitor['tick'] - 1          # Sell tickets
                print(self.tid,':now left:',monitor['tick'])   # Tickets left
                doChore()                                      # Other critical operations
            else:
                print("Thread_id",self.tid," No more tickets")
                os._exit(0)                                    # Exit the whole process immediately
            monitor['lock'].release()                          # Unblock
            doChore()                                          # Non-critical operations

# Start of the main function
monitor = {'tick':100, 'lock':threading.Lock()}

# Start 10 threads
for k in range(10):
    new_thread = BoothThread(k, monitor)
    new_thread.start()
```
我们自己定义了一个类`BoothThread`, 这个类继承自`thread.Threading`类。然后我们把上面的`booth()`所进行的操作统统放入到`BoothThread`类的`run()`方法中。注意，我们没有使用全局变量声明`global`，而是使用了一个词典`monitor`存放全局变量，然后把词典作为参数传递给线程函数。由于词典是可变数据对象，所以当它被传递给函数的时候，函数所使用的依然是同一个对象，相当于被多个线程所共享。这也是多线程乃至于多进程编程的一个技巧 (应尽量避免上面的`global`声明的用法，因为它并不适用于windows平台)。

上面OOP编程方法与面向过程的编程方法相比，并没有带来太大实质性的差别。

 

##其他

`threading.Thread`对象： 我们已经介绍了该对象的`start()`和`run()`, 此外：

`join()`方法，调用该方法的线程将等待直到该`Thread`对象完成，再恢复运行。这与进程间调用`wait()`函数相类似。
 

下面的对象用于处理多线程同步。对象一旦被建立，可以被多个线程共享，并根据情况阻塞某些进程。请与Linux多线程与同步中的同步工具参照阅读。

`threading.Lock`对象: mutex, 有`acquire()`和`release()`方法。

`threading.Condition`对象: condition variable，建立该对象时，会包含一个`Lock`对象 (因为condition variable总是和mutex一起使用)。可以对`Condition`对象调用`acquire()`和`release()`方法，以控制潜在的`Lock`对象。此外:

`wait()`方法，相当于`cond_wait()`
`notify_all()`，相当与`cond_broadcast()`
`nofify()`，与`notify_all()`功能类似，但只唤醒一个等待的线程，而不是全部
`threading.Semaphore`对象: semaphore，也就是计数锁(也称信号量。semaphore传统意义上是一种进程间同步工具，见Linux进程间通信)。创建对象的时候，可以传递一个整数作为计数上限 (`sema = threading.Semaphore(5)`)。它与`Lock`类似，也有`Lock`的两个方法。
`threading.Event`对象: 与`threading.Condition`相类似，相当于没有潜在的`Lock`保护的condition variable。对象有`True`和`False`两个状态。可以多个线程使用`wait()`等待，直到某个线程调用该对象的`set()`方法，将对象设置为`True`。线程可以调用对象的`clear()`方法来重置对象为`False`状态。
 
 
练习
参照Linux多线程与同步中的condition variable的例子，使用Python实现。同时考虑使用面向过程和面向对象的编程方法。
更多的threading的内容请参考:
http://docs.python.org/library/threading.html

 

##总结

`threading.Thread`

`Lock`, `Condition`, `Semaphore`, `Event`

