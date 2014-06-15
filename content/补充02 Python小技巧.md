##Python补充02 Python小技巧


 

在这里列举一些我使用Python时积累的小技巧。这些技巧是我在使用Python过程中经常使用的。之前很零碎的记在笔记本中，现在整理出来，和大家分享，也作为Python快速教程的一个补充。

 

##import模块

在Python经常使用import声明，以使用其他模块(也就是其它.py文件)中定义的对象。

1) 使用__name__

当我们编写Python库模块的时候，我们往往运行一些测试语句。当这个程序作为库被import的时候，我们并不需要运行这些测试语句。一种解决方法是在import之前，将模块中的测试语句注释掉。Python有一种更优美的解决方法，就是使用__name__。

下面是一个简单的库程序TestLib.py。当直接运行TestLib.py时，__name__为"__main__"。如果被import的话，__name__为"TestLib"。

```python
def lib_func(a):
    return a + 10

def lib_func_another(b):
    return b + 20

if __name__ == '__main__':
    test = 101
    print(lib_func(test))
```
 我们在user.py中import上面的TestLib。
 
```python
import TestLib
print(TestLib.lib_func(120))
```
 你可以尝试不在TestLib.py中使用if __name__=='__main__'， 并对比运行结果。

 

2) 更多import使用方式

import TestLib as test         # 引用TestLib模块，并将它改名为t

比如:
```pyhton
import TestLib as t
print(t.lib_func(120))
from TestLib import lib_func   # 只引用TestLib中的lib_func对象，并跳过TestLib引用字段
```
这样的好处是减小所引用模块的内存占用。

比如：
```ython
from TestLib import lib_func
print(lib_func(120))
from TestLib import *          # 引用所有TestLib中的对象，并跳过TestLib引用字段
```
比如:
```python
from TestLib import *
print(lib_func(120))
```

##查询

1) 查询函数的参数

当我们想要知道某个函数会接收哪些参数的时候，可以使用下面方法查询。
```python
import inspect
print(inspect.getargspec(func))
``` 

2) 查询对象的属性

除了使用dir()来查询对象的属性之外，我们可以使用下面内置(built-in)函数来确认一个对象是否具有某个属性：

hasattr(obj, attr_name)   # attr_name是一个字符串

例如：
```python
a = [1,2,3]
print(hasattr(a,'append'))
``` 

2) 查询对象所属的类和类名称
```python
a = [1, 2, 3]
print a.__class__
print a.__class__.__name__
``` 

3) 查询父类

我们可以用__base__属性来查询某个类的父类：

cls.__base__

例如：
```python
print(list.__base__)
``` 

##使用中文(以及其它非ASCII编码)

在Python程序的第一行加入#coding=utf8，例如:
```python
#coding=utf8
print("你好吗？")
```
也能用以下方式：
```python
#-*- coding: UTF-8 -*-
print("你好吗？")
``` 

##表示2进制，8进制和16进制数字

在2.6以上版本，以如下方式表示
```python
print(0b1110)     # 二进制，以0b开头
print(0o10)       # 八进制，以0o开头
print(0x2A)       # 十六进制，以0x开头
```
如果是更早版本，可以用如下方式：
```python
print(int("1110", 2))
print(int("10", 8))
print(int("2A", 16))
``` 

##注释

一行内的注释可以以#开始

多行的注释可以以'''开始，以'''结束，比如

```python
'''
This is demo
'''

def func():
    # print something
    print("Hello world!")  # use print() function

# main
func()
```
注释应该和所在的程序块对齐。

 

##搜索路径

当我们import的时候，Python会在搜索路径中查找模块(module)。比如上面import TestLib，就要求TestLib.py在搜索路径中。

我们可以通过下面方法来查看搜索路径：
```python
import sys
print(sys.path)
```
我们可以在Python运行的时候增加或者删除sys.path中的元素。另一方面，我们可以通过在shell中增加PYTHONPATH环境变量，来为Python增加搜索路径。

下面我们增加/home/vamei/mylib到搜索路径中：

$export PYTHONPATH=$PYTHONPATH:/home/vamei/mylib

你可以将正面的这行命令加入到～/.bashrc中。这样，我们就长期的改变了搜索路径。

 

##脚本与命令行结合

可以使用下面方法运行一个Python脚本，在脚本运行结束后，直接进入Python命令行。这样做的好处是脚本的对象不会被清空，可以通过命令行直接调用。
```python
$python -i script.py
```
 

##安装非标准包

Python的标准库随着Python一起安装。当我们需要非标准包时，就要先安装。

1) 使用Linux repository (Linux环境)

这是安装Python附加包的一个好的起点。你可以在Linux repository中查找可能存在的Python包 (比如在Ubuntu Software Center中搜索matplot)。

2) 使用pip。pip是Python自带的包管理程序，它连接Python repository，并查找其中可能存在的包。

比如使用如下方法来安装、卸载或者升级web.py：

$pip install web.py

$pip uninstall web.py

$pip install --upgrade web.py

如果你的Python安装在一个非标准的路径(使用$which python来确认python可执行文件的路径)中，比如/home/vamei/util/python/bin中，你可以使用下面方法设置pip的安装包的路径:

$pip install --install-option="--prefix=/home/vamei/util/" web.py

3) 从源码编译

如果上面方法都没法找到你想要的库，你可能需要从源码开始编译。Google往往是最好的起点。
