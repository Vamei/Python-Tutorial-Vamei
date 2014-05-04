##Python标准库12 数学与随机数 (math包，random包)


 

我们已经在Python运算中看到Python最基本的数学运算功能。此外，math包补充了更多的函数。当然，如果想要更加高级的数学功能，可以考虑选择标准库之外的numpy和scipy项目，它们不但支持数组和矩阵运算，还有丰富的数学和物理方程可供使用。

此外，random包可以用来生成随机数。随机数不仅可以用于数学用途，还经常被嵌入到算法中，用以提高算法效率，并提高程序的安全性。

 

##math包

math包主要处理数学相关的运算。math包定义了两个常数:

math.e   # 自然常数e

math.pi  # 圆周率pi

 

此外，math包还有各种运算函数 (下面函数的功能可以参考数学手册)：

math.ceil(x)       # 对x向上取整，比如x=1.2，返回2

math.floor(x)      # 对x向下取整，比如x=1.2，返回1

math.pow(x,y)      # 指数运算，得到x的y次方

math.log(x)        # 对数，默认基底为e。可以使用base参数，来改变对数的基地。比如math.log(100,base=10)

math.sqrt(x)       # 平方根

 

三角函数: math.sin(x), math.cos(x), math.tan(x), math.asin(x), math.acos(x), math.atan(x)

这些函数都接收一个弧度(radian)为单位的x作为参数。

 

角度和弧度互换: math.degrees(x), math.radians(x)

 

双曲函数: math.sinh(x), math.cosh(x), math.tanh(x), math.asinh(x), math.acosh(x), math.atanh(x)

 

特殊函数： math.erf(x), math.gamma(x)

 

##random包

如果你已经了解伪随机数(psudo-random number)的原理，那么你可以使用如下:

random.seed(x)

来改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。

 

1) 随机挑选和排序

random.choice(seq)   # 从序列的元素中随机挑选一个元素，比如random.choice(range(10))，从0到9中随机挑选一个整数。

random.sample(seq,k) # 从序列中随机挑选k个元素

random.shuffle(seq)  # 将序列的所有元素随机排序

 

2）随机生成实数

下面生成的实数符合均匀分布(uniform distribution)，意味着某个范围内的每个数字出现的概率相等:

random.random()          # 随机生成下一个实数，它在[0,1)范围内。

random.uniform(a,b)      # 随机生成下一个实数，它在[a,b]范围内。

 

下面生成的实数符合其它的分布 (你可以参考一些统计方面的书籍来了解这些分布):

random.gauss(mu,sigma)    # 随机生成符合高斯分布的随机数，mu,sigma为高斯分布的两个参数。 

random.expovariate(lambd) # 随机生成符合指数分布的随机数，lambd为指数分布的参数。

此外还有对数分布，正态分布，Pareto分布，Weibull分布，可参考下面链接:

http://docs.python.org/library/random.html

 

假设我们有一群人参加舞蹈比赛，为了公平起见，我们要随机排列他们的出场顺序。我们下面利用random包实现：

import random
all_people = ['Tom', 'Vivian', 'Paul', 'Liya', 'Manu', 'Daniel', 'Shawn']
random.shuffle(all_people)
for i,name in enumerate(all_people):
    print(i,':'+name)
 

##练习

设计下面两种彩票号码生成器:

1. 从1到22中随机抽取5个整数 （这5个数字不重复）

2. 随机产生一个8位数字，每位数字都可以是1到6中的任意一个整数。 

 

##总结

math.floor(), math.sqrt(), math.sin(), math.degrees()

random.random(), random.choice(), random.shuffle()
