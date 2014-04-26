#Python标准库05 存储对象 (pickle包，cPickle包)



 

在之前对Python对象的介绍中 (面向对象的基本概念，面向对象的进一步拓展)，我提到过Python“一切皆对象”的哲学，在Python中，无论是变量还是函数，都是一个对象。当Python运行时，对象存储在内存中，随时等待系统的调用。然而，内存里的数据会随着计算机关机和消失，如何将对象保存到文件，并储存在硬盘上呢？

计算机的内存中存储的是二进制的序列 (当然，在Linux眼中，是文本流)。我们可以直接将某个对象所对应位置的数据抓取下来，转换成文本流 (这个过程叫做serialize)，然后将文本流存入到文件中。由于Python在创建对象时，要参考对象的类定义，所以当我们从文本中读取对象时，必须在手边要有该对象的类定义，才能懂得如何去重建这一对象。从文件读取时，对于Python的内建(built-in)对象 (比如说整数、词典、表等等)，由于其类定义已经载入内存，所以不需要我们再在程序中定义类。但对于用户自行定义的对象，就必须要先定义类，然后才能从文件中载入对象 (比如面向对象的基本概念中的对象那个summer)。

 

##pickle包

对于上述过程，最常用的工具是Python中的pickle包。

1) 将内存中的对象转换成为文本流：

```pyhton
import pickle

# define class
class Bird(object):
    have_feather = True
    way_of_reproduction  = 'egg'

summer       = Bird()                 # construct an object
picklestring = pickle.dumps(summer)   # serialize object
```
使用pickle.dumps()方法可以将对象summer转换成了字符串 picklestring(也就是文本流)。随后我们可以用普通文本的存储方法来将该字符串储存在文件(文本文件的输入输出)。

 

当然，我们也可以使用pickle.dump()的方法，将上面两部合二为一:

```python
import pickle

# define class
class Bird(object):
    have_feather = True
    way_of_reproduction  = 'egg'

summer       = Bird()                        # construct an object
fn           = 'a.pkl'
with open(fn, 'w') as f:                     # open file with write-mode
    picklestring = pickle.dump(summer, f)   # serialize and save object
```
对象summer存储在文件a.pkl

 

2) 重建对象

首先，我们要从文本中读出文本，存储到字符串 (文本文件的输入输出)。然后使用pickle.loads(str)的方法，将字符串转换成为对象。要记得，此时我们的程序中必须已经有了该对象的类定义。

 

此外，我们也可以使用pickle.load()的方法，将上面步骤合并:

```python
import pickle

# define the class before unpickle
class Bird(object):
    have_feather = True
    way_of_reproduction  = 'egg'

fn     = 'a.pkl'
with open(fn, 'r') as f:
    summer = pickle.load(f)   # read file and build object
```
 

 

##cPickle包

cPickle包的功能和用法与pickle包几乎完全相同 (其存在差别的地方实际上很少用到)，不同在于cPickle是基于c语言编写的，速度是pickle包的1000倍。对于上面的例子，如果想使用cPickle包，我们都可以将import语句改为:

import cPickle as pickle
就不需要再做任何改动了。

 

##总结

对象 -> 文本 -> 文件

pickle.dump(), pickle.load(), cPickle
