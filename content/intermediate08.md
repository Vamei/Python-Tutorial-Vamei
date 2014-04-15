#Python进阶08 异常处理


 

##异常处理

在项目开发中，异常处理是不可或缺的。异常处理帮助人们debug，通过更加丰富的信息，让人们更容易找到bug的所在。异常处理还可以提高程序的容错性。

我们之前在讲循环对象的时候，曾提到一个StopIteration的异常，该异常是在循环对象穷尽所有元素时的报错。

我们以它为例，来说明基本的异常处理。

一个包含异常的程序:
```python
re = iter(range(5))

for i in range(100):
    print re.next()

print 'HaHaHaHa'
```
首先，我们定义了一个循环对象re，该循环对象将进行5次循环，每次使用序列的一个元素。

在随后的for循环中，我们手工调用next()函数。当循环进行到第6次的时候，re.next()不会再返回元素，而是抛出(raise)StopIteration的异常。整个程序将会中断。

 

我们可以修改以上异常程序，直到完美的没有bug。但另一方面，如果我们在写程序的时候，知道这里可能犯错以及可能的犯错类型，我们可以针对该异常类型定义好”应急预案“。
```python
re = iter(range(5))

try:
    for i in range(100):
        print re.next()
except StopIteration:
    print 'here is end ',i

print 'HaHaHaHa'
```
在try程序段中，我们放入容易犯错的部分。我们可以跟上except，来说明如果在try部分的语句发生StopIteration时，程序该做的事情。如果没有发生异常，则except部分被跳过。

随后，程序将继续运行，而不是彻底中断。

 

完整的语法结构如下：

```python
try:
    ...
except exception1:
    ...
except exception2:
    ...
except:
    ...
else:
    ...
finally:
    ...
```
 

如果try中有异常发生时，将执行异常的归属，执行except。异常层层比较，看是否是exception1, exception2...，直到找到其归属，执行相应的except中的语句。如果except后面没有任何参数，那么表示所有的exception都交给这段程序处理。比如:
```python
try:
    print(a*2)
except TypeError:
    print("TypeError")
except:
    print("Not Type Error & Error noted")
```
由于a没有定义，所以是NameError。异常最终被except:部分的程序捕捉。

 

如果无法将异常交给合适的对象，异常将继续向上层抛出，直到被捕捉或者造成主程序报错。比如下面的程序

```python
def test_func():
    try:
        m = 1/0
    except NameError:
        print("Catch NameError in the sub-function")

try:
    test_func()
except ZeroDivisionError:
    print("Catch error in the main program")
```
子程序的try...except...结构无法处理相应的除以0的错误，所以错误被抛给上层的主程序。

 

如果try中没有异常，那么except部分将跳过，执行else中的语句。

finally是无论是否有异常，最后都要做的一些事情。

流程如下，

try->异常->except->finally

try->无异常->else->finally

 

##抛出异常

我们也可以自己写一个抛出异常的例子:
```python
print 'Lalala'
raise StopIteration
print 'Hahaha'
```
这个例子不具备任何实际意义。只是为了说明raise语句的作用。

StopIteration是一个类。抛出异常时，会自动有一个中间环节，就是生成StopIteration的一个对象。Python实际上抛出的，是这个对象。当然，也可以自行声场对象:
```python
raise StopIteration()
``` 

##总结
```python
try: ... except exception: ... else: ... finally: ...
raise exception
```
