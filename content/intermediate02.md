#Python进阶02 文本文件的输入输出



 

Python具有基本的文本文件读写功能。Python的标准库提供有更丰富的读写功能。

文本文件的读写主要通过open()所构建的文件对象来实现。

 

##创建文件对象

我们打开一个文件，并使用一个对象来表示该文件：
```python
f = open(文件名，模式)
```
 

最常用的模式有：
```python
"r"     # 只读

"w"     # 写入

"a"     # 追加
```

 

比如
```python
>>>f = open("test.txt","r")
```
 

##文件对象的方法

读取：
```python
content = f.read(N)          # 读取N bytes的数据

content = f.readline()       # 读取一行

content = f.readlines()      # 读取所有行，储存在列表中，每个元素是一行。
```

Pythonic:
读取文件更简洁的写法：
```python
datafile = open('datafile',"r")
for line in datafile:
    do_something(line)
```

写入：
```python
f.write('I like apple')      # 将'I like apple'写入文件
```
 

关闭文件：
```python
f.close()
```

##总结
```python
f    = open(name, "r")

line = f.readline()

f.write('abc')

f.close()
```
