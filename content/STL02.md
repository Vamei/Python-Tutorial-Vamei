#Python标准库02 时间与日期 (time, datetime包)

 

Python具有良好的时间和日期管理功能。实际上，计算机只会维护一个挂钟时间(wall clock time)，这个时间是从某个固定时间起点到现在的时间间隔。时间起点的选择与计算机相关，但一台计算机的话，这一时间起点是固定的。其它的日期信息都是从这一时间计算得到的。此外，计算机还可以测量CPU实际上运行的时间，也就是处理器时间(processor clock time)，以测量计算机性能。当CPU处于闲置状态时，处理器时间会暂停。

 

##time包

time包基于C语言的库函数(library functions)。Python的解释器通常是用C编写的，Python的一些函数也会直接调用C语言的库函数。
```python
import time
print(time.time())   # wall clock time, unit: second
print(time.clock())  # processor clock time, unit: second
``` 

time.sleep()可以将程序置于休眠状态，直到某时间间隔之后再唤醒程序，让程序继续运行。
```python
import time
print('start')
time.sleep(10)     # sleep for 10 seconds
print('wake up')
```
当我们需要定时地查看程序运行状态时，就可以利用该方法。

 

time包还定义了struct_time对象。该对象实际上是将挂钟时间转换为年、月、日、时、分、秒……等日期信息，存储在该对象的各个属性中(tm_year, tm_mon, tm_mday...)。下面方法可以将挂钟时间转换为struct_time对象:
```python
st = time.gmtime()      # 返回struct_time格式的UTC时间
st = time.localtime()   # 返回struct_time格式的当地时间, 当地时区根据系统环境决定。

s  = time.mktime(st)    # 将struct_time格式转换成wall clock time
``` 

##datetime包

1) 简介

datetime包是基于time包的一个高级包， 为我们提供了多一层的便利。

datetime可以理解为date和time两个组成部分。date是指年月日构成的日期(相当于日历)，time是指时分秒微秒构成的一天24小时中的具体时间(相当于手表)。你可以将这两个分开管理(datetime.date类，datetime.time类)，也可以将两者合在一起(datetime.datetime类)。由于其构造大同小异，我们将只介绍datetime.datetime类。

比如说我现在看到的时间，是2012年9月3日21时30分，我们可以用如下方式表达：
```python
import datetime
t = datetime.datetime(2012,9,3,21,30)
print(t)
```
所返回的t有如下属性:

hour, minute, second, microsecond

year, month, day, weekday   # weekday表示周几

 

2) 运算

datetime包还定义了时间间隔对象(timedelta)。一个时间点(datetime)加上一个时间间隔(timedelta)可以得到一个新的时间点(datetime)。比如今天的上午3点加上5个小时得到今天的上午8点。同理，两个时间点相减会得到一个时间间隔。

```python
import datetime
t      = datetime.datetime(2012,9,3,21,30)
t_next = datetime.datetime(2012,9,5,23,30)
delta1 = datetime.timedelta(seconds = 600)
delta2 = datetime.timedelta(weeks = 3)
print(t + delta1)
print(t + delta2)
print(t_next - t)
```
在给datetime.timedelta传递参数（如上的seconds和weeks）的时候，还可以是days, hours, milliseconds, microseconds。

 

两个datetime对象还可以进行比较。比如使用上面的t和t_next:
```python
print(t > t_next)
``` 

3) datetime对象与字符串转换

假如我们有一个的字符串，我们如何将它转换成为datetime对象呢？

一个方法是用上一讲的正则表达式来搜索字符串。但时间信息实际上有很明显的特征，我们可以用格式化读取的方式读取时间信息。
```python
from datetime import datetime
format = "output-%Y-%m-%d-%H%M%S.txt" 
str    = "output-1997-12-23-030000.txt" 
t      = datetime.strptime(str, format)
strptime, p = parsing
```

我们通过format来告知Python我们的str字符串中包含的日期的格式。在format中，%Y表示年所出现的位置, %m表示月份所出现的位置……。

反过来，我们也可以调用datetime对象的strftime()方法，来将datetime对象转换为特定格式的字符串。比如上面所定义的t_next,
```python
print(t_next.strftime(format))
```
strftime, f = formatting

具体的格式写法可参阅官方文档。 如果是Linux系统，也可查阅date命令的手册($man date)，两者相通。

 

##总结

时间，休眠

datetime, timedelta

格式化时间


