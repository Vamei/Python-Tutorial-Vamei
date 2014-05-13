
##Python标准库11 多进程探索 (multiprocessing包)


 

在初步了解Python多进程之后，我们可以继续探索multiprocessing包中更加高级的工具。这些工具可以让我们更加便利地实现多进程。

 

##进程池

进程池 (Process Pool)可以创建多个进程。这些进程就像是随时待命的士兵，准备执行任务(程序)。一个进程池中可以容纳多个待命的士兵。

 



“三个进程的进程池”

 

 

比如下面的程序:

```python
import multiprocessing as mul

def f(x):
    return x**2

pool = mul.Pool(5)
rel  = pool.map(f,[1,2,3,4,5,6,7,8,9,10])
print(rel)
```
我们创建了一个容许5个进程的进程池 (Process Pool) 。Pool运行的每个进程都执行f()函数。我们利用map()方法，将f()函数作用到表的每个元素上。这与built-in的map()函数类似，只是这里用5个进程并行处理。如果进程运行结束后，还有需要处理的元素，那么的进程会被用于重新运行f()函数。除了map()方法外，Pool还有下面的常用方法。

apply_async(func,args)  从进程池中取出一个进程执行func，args为func的参数。它将返回一个AsyncResult的对象，你可以对该对象调用get()方法以获得结果。

close()  进程池不再创建新的进程

join()   wait进程池中的全部进程。必须对Pool先调用close()方法才能join。

 

##练习

有下面一个文件download.txt。

www.sina.com.cn
www.163.com
www.iciba.com
www.cnblogs.com
www.qq.com
www.douban.com
使用包含3个进程的进程池下载文件中网站的首页。(你可以使用subprocess调用wget或者curl等下载工具执行具体的下载任务)

 

##共享资源

我们在Python多进程初步已经提到，我们应该尽量避免多进程共享资源。多进程共享资源必然会带来进程间相互竞争。而这种竞争又会造成race condition，我们的结果有可能被竞争的不确定性所影响。但如果需要，我们依然可以通过共享内存和Manager对象这么做。

 



共享“资源”

共享内存

在Linux进程间通信中，我们已经讲述了共享内存(shared memory)的原理，这里给出用Python实现的例子:

```python
# modified from official documentation
import multiprocessing

def f(n, a):
    n.value   = 3.14
    a[0]      = 5

num   = multiprocessing.Value('d', 0.0)
arr   = multiprocessing.Array('i', range(10))

p = multiprocessing.Process(target=f, args=(num, arr))
p.start()
p.join()

print num.value
print arr[:]
```
这里我们实际上只有主进程和Process对象代表的进程。我们在主进程的内存空间中创建共享的内存，也就是Value和Array两个对象。对象Value被设置成为双精度数(d), 并初始化为0.0。而Array则类似于C中的数组，有固定的类型(i, 也就是整数)。在Process进程中，我们修改了Value和Array对象。回到主程序，打印出结果，主程序也看到了两个对象的改变，说明资源确实在两个进程之间共享。

 

Manager

Manager对象类似于服务器与客户之间的通信 (server-client)，与我们在Internet上的活动很类似。我们用一个进程作为服务器，建立Manager来真正存放资源。其它的进程可以通过参数传递或者根据地址来访问Manager，建立连接后，操作服务器上的资源。在防火墙允许的情况下，我们完全可以将Manager运用于多计算机，从而模仿了一个真实的网络情境。下面的例子中，我们对Manager的使用类似于shared memory，但可以共享更丰富的对象类型。

```python
import multiprocessing

def f(x, arr, l):
    x.value = 3.14
    arr[0] = 5
    l.append('Hello')

server = multiprocessing.Manager()
x    = server.Value('d', 0.0)
arr  = server.Array('i', range(10))
l    = server.list()

proc = multiprocessing.Process(target=f, args=(x, arr, l))
proc.start()
proc.join()

print(x.value)
print(arr)
print(l)
```
Manager利用list()方法提供了表的共享方式。实际上你可以利用dict()来共享词典，Lock()来共享threading.Lock(注意，我们共享的是threading.Lock，而不是进程的mutiprocessing.Lock。后者本身已经实现了进程共享)等。 这样Manager就允许我们共享更多样的对象。

 

我们在这里不深入讲解Manager在远程情况下的应用。有机会的话，会在网络应用中进一步探索。

 

##总结

Pool

Shared memory, Manager

