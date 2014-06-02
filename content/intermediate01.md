#Python进阶01 词典

基础教程介绍了基本概念，特别是对象和类。

进阶教程对基础教程的进一步拓展，说明Python的细节。希望在进阶教程之后，你对Python有一个更全面的认识。

 

之前我们说了，列表是Python里的一个类。一个特定的表，比如说nl = [1,3,8]，就是这个类的一个对象。我们可以调用这个对象的一些方法，比如 nl.append(15)。
我们要介绍一个新的类，词典 (dictionary)。与列表相似，词典也可以储存多个元素。这种储存多个元素的对象称为容器(container)。

 

##基本概念

常见的创建词典的方法:
```python
>>>dic = {'tom':11, 'sam':57,'lily':100}

>>>print type(dic)
```
词典和表类似的地方，是包含有多个元素，每个元素以逗号分隔。但词典的元素包含有两部分，键和值，常见的是以字符串来表示键，也可以使用数字或者真值来表示键（不可变的对象可以作为键）。值可以是任意对象。键和值两者一一对应。

 

比如上面的例子中，'tom'对应11，'sam'对应57，'lily'对应100

 

与表不同的是，词典的元素没有顺序。你不能通过下标引用元素。词典是通过键来引用。
```python
>>>print dic['tom']

>>>dic['tom'] = 30

>>>print dic
```
 

构建一个新的空的词典：
```python
>>>dic = {}

>>>print dic
```
 

在词典中增添一个新元素的方法：
```python
>>>dic['lilei'] = 99

>>>print dic
```
这里，我们引用一个新的键，并赋予它对应的值。

 

##词典元素的循环调用
```python
dic = {'lilei': 90, 'lily': 100, 'sam': 57, 'tom': 90}
for key in dic:
    print dic[key]
```

在循环中，dict的每个键，被提取出来，赋予给key变量。

通过print的结果，我们可以再次确认，dic中的元素是没有顺序的。


词典的遍历也可以同时取出键和值：
```python
for key,value in dict_od.items():
    print key , value , '\n'
```

##词典的常用方法
```python
>>>print dic.keys()           # 返回dic所有的键

>>>print dic.values()         # 返回dic所有的值

>>>print dic.items()          # 返回dic所有的元素（键值对）

>>>dic.clear()                # 清空dic，dict变为{}
```
 

另外有一个很常用的用法：
```python
>>>del dic['tom']             # 删除 dic 的‘tom’元素
```
del是Python中保留的关键字，用于删除对象。

 

与表类似，你可以用len()查询词典中的元素总数。
```python
>>>print(len(dic))
```

## 词典排序
前面我们已经谈到，词典是无序的，那么，如果需要词典内容有序，就需要使用排序方法：

词典按key排序，排序后的结果存放到列表中:（这里用到了列表推导式)

```python
listRes = sorted([(k, v) for k, v in dict_rate.items()], reverse=True)
```

词典按value排序，排序后的结果存放到列表中：
```python
listRes = sorted([(v, k) for k, v in dict_rate.items()], reverse=True
```

##总结

词典的每个元素是键值对。元素没有顺序。
```python
dic = {'tom':11, 'sam':57,'lily':100}

dic['tom'] = 99

for key in dic: ...

del, len()
```
