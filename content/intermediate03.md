#Python进阶03 模块



 

我们之前看到了函数和对象。从本质上来说，它们都是为了更好的组织已经有的程序，以方便重复利用。

模块(module)也是为了同样的目的。在Python中，一个.py文件就构成一个模块。通过模块，你可以调用其它文件中的程序。

 

##引入模块

我们先写一个first.py文件，内容如下：
```python
def laugh():
    print 'HaHaHaHa'
``` 

再写一个second.py，并引入first中的程序：
```python
import first

for i in range(10):
    first.laugh()
```
在second.py中，我们使用了first.py中定义的laugh()函数。

 

引入模块后，可以通过模块.对象的方式来调用引入模块中的某个对象。上面例子中，first为引入的模块，laugh()是我们所引入的对象。

Python中还有其它的引入方式,
```python
import a as b             # 引入模块a，并将模块a重命名为b

from a import function1   # 从模块a中引入function1对象。调用a中对象时，我们不用再说明模块，即直接使用function1，而不是a.function1。

from a import *           # 从模块a中引入所有对象。调用a中对象时，我们不用再说明模块，即直接使用对象，而不是a.对象。
```
这些引用方式，可以方便后面的程序书写。

 

##搜索路径

Python会在以下路径中搜索它想要寻找的模块：

程序所在的文件夹
标准库的安装路径
操作系统环境变量PYTHONPATH所包含的路径
 

如果你有自定义的模块，或者下载的模块，可以根据情况放在相应的路径，以便Python可以找到。

 

模块包

可以将功能相似的模块放在同一个文件夹（比如说this_dir）中，构成一个模块包。通过
```python
import this_dir.module
```
引入this_dir文件夹中的module模块。

 

该文件夹中必须包含一个__init__.py的文件，提醒Python，该文件夹为一个模块包。__init__.py可以是一个空文件。

 

##总结
```
import module

module.object

__init__.py
```
