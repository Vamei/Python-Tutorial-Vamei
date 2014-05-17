#Python进阶05 循环设计


 

在“循环”一节，我们已经讨论了Python基本的循环语法。这一节，我们将接触更加灵活的循环方式。

 

##range()

在Python中，for循环后的in跟随一个序列的话，循环每次使用的序列元素，而不是序列的下标。

之前我们已经使用过range()来控制for循环。现在，我们继续开发range的功能，以实现下标对循环的控制：
```python
S = 'abcdefghijk'
for i in range(0,len(S),2):
    print S[i]
```
在该例子中，我们利用len()函数和range()函数，用i作为S序列的下标来控制循环。在range函数中，分别定义上限，下限和每次循环的步长。这就和C语言中的for循环相类似了。

 

##enumerate()

利用enumerate()函数，可以在每次循环中同时得到下标和元素：
```python
S = 'abcdefghijk'
for (index,char) in enumerate(S):
    print index
    print char
```
实际上，enumerate()在每次循环中，返回的是一个包含两个元素的定值表(tuple)，两个元素分别赋予index和char

增加下标的访问设计
在python 3.2中，支持enumerate带第二个参数，用来指定循环的开始位置，比如，从第一个下标位置开始循环：
```python
array = [1, 2, 3, 4, 5]

for i, e in enumerate(array,1):
    print i, e

```
 

##zip()

如果你多个等长的序列，然后想要每次循环时从各个序列分别取出一个元素，可以利用zip()方便地实现：
```python
ta = [1,2,3]
tb = [9,8,7]
tc = ['a','b','c']
for (a,b,c) in zip(ta,tb,tc):
    print(a,b,c)
```    
每次循环时，从各个序列分别从左到右取出一个元素，合并成一个tuple，然后tuple的元素赋予给a,b,c

zip()函数的功能，就是从多个列表中，依次各取出一个元素。每次取出的(来自不同列表的)元素合成一个元组，合并成的元组放入zip()返回的列表中。zip()函数起到了聚合列表的功能。

 

我们可以分解聚合后的列表，如下:
```python
ta = [1,2,3]
tb = [9,8,7]

# cluster
zipped = zip(ta,tb)
print(zipped)

# decompose
na, nb = zip(*zipped)
print(na, nb)
```


##总结

range()

enumerate()

zip()
