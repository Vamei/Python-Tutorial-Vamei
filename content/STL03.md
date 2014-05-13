#Python标准库03 路径与文件 (os.path包, glob包)



 

路径与文件的简介请参看Linux文件系统

 

##os.path包

os.path包主要是处理路径字符串，比如说'/home/vamei/doc/file.txt'，提取出有用信息。

```python
import os.path
path = '/home/vamei/doc/file.txt'

print(os.path.basename(path))    # 查询路径中包含的文件名
print(os.path.dirname(path))     # 查询路径中包含的目录

info = os.path.split(path)       # 将路径分割成文件名和目录两个部分，放在一个表中返回
path2 = os.path.join('/', 'home', 'vamei', 'doc', 'file1.txt')  # 使用目录名和文件名构成一个路径字符串

p_list = [path, path2]
print(os.path.commonprefix(p_list))    # 查询多个路径的共同部分
```
此外，还有下面的方法：

os.path.normpath(path)   # 去除路径path中的冗余。比如'/home/vamei/../.'被转化为'/home'

 

os.path还可以查询文件的相关信息(metadata)。文件的相关信息不存储在文件内部，而是由操作系统维护的，关于文件的一些信息(比如文件类型，大小，修改时间)。

```python
import os.path 
path = '/home/vamei/doc/file.txt'

print(os.path.exists(path))    # 查询文件是否存在

print(os.path.getsize(path))   # 查询文件大小
print(os.path.getatime(path))  # 查询文件上一次读取的时间
print(os.path.getmtime(path))  # 查询文件上一次修改的时间

print(os.path.isfile(path))    # 路径是否指向常规文件
print(os.path.isdir(path))     # 路径是否指向目录文件
```
 (实际上，这一部份类似于Linux中的ls命令的某些功能)

 

##glob包

glob包最常用的方法只有一个, glob.glob()。该方法的功能与Linux中的ls相似(参看Linux文件管理命令)，接受一个Linux式的文件名格式表达式(filename pattern expression)，列出所有符合该表达式的文件（与正则表达式类似），将所有文件名放在一个表中返回。所以glob.glob()是一个查询目录下文件的好方法。

该文件名表达式的语法与Python自身的正则表达式不同 (你可以同时看一下fnmatch包，它的功能是检测一个文件名是否符合Linux的文件名格式表达式)。 如下：

Filename Pattern Expression      Python Regular Expression

*                                .*

?                                .

[0-9]                            same

[a-e]                            same

[^mnp]                           same

 

我们可以用该命令找出/home/vamei下的所有文件:
```python
import glob
print(glob.glob('/home/vamei/*'))
``` 

##总结

文件系统

os.path

glob.glob
