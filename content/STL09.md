
##Python标准库09 当前进程信息 (部分os包)


 

我们在Linux的概念与体系，多次提及进程的重要性。Python的os包中有查询和修改进程信息的函数。学习Python的这些工具也有助于理解Linux体系。

 

##进程信息

os包中相关函数如下：

uname() 返回操作系统相关信息。类似于Linux上的uname命令。

umask() 设置该进程创建文件时的权限mask。类似于Linux上的umask命令，见Linux文件管理背景知识

 

get*() 查询 (*由以下代替)

    uid, euid, resuid, gid, egid, resgid ：权限相关，其中resuid主要用来返回saved UID。相关介绍见Linux用户与“最小权限”原则

    pid, pgid, ppid, sid                 ：进程相关。相关介绍见Linux进程关系

 

put*() 设置 (*由以下代替)

    euid, egid： 用于更改euid，egid。

    uid, gid  ： 改变进程的uid, gid。只有super user才有权改变进程uid和gid (意味着要以$sudo python的方式运行Python)。

    pgid, sid ： 改变进程所在的进程组(process group)和会话(session)。

 

getenviron()：获得进程的环境变量

setenviron()：更改进程的环境变量

 

例1，进程的real UID和real GID
```python
import os
print(os.getuid())
print(os.getgid())
```
将上面的程序保存为py_id.py文件，分别用$python py_id.py和$sudo python py_id.py看一下运行结果

 

##saved UID和saved GID

我们希望saved UID和saved GID如我们在Linux用户与“最小权限”原则中描述的那样工作，但这很难。原因在于，当我们写一个Python脚本后，我们实际运行的是python这个解释器，而不是Python脚本文件。对比C，C语言直接运行由C语言编译成的执行文件。我们必须更改python解释器本身的权限来运用saved UID机制，然而这么做又是异常危险的。

比如说，我们的python执行文件为/usr/bin/python (你可以通过$which python获知)

我们先看一下

$ls -l /usr/bin/python

的结果:

-rwxr-xr-x root root

 

我们修改权限以设置set UID和set GID位 (参考Linux用户与“最小权限”原则)

$sudo chmod 6755 /usr/bin/python

/usr/bin/python的权限成为:

-rwsr-sr-x root root

 

随后，我们运行文件下面test.py文件，这个文件可以是由普通用户vamei所有:
```python
import os
print(os.getresuid())
```
我们得到结果:

(1000, 0, 0)

上面分别是UID，EUID，saved UID。我们只用执行一个由普通用户拥有的python脚本，就可以得到super user的权限！所以，这样做是极度危险的，我们相当于交出了系统的保护系统。想像一下Python强大的功能，别人现在可以用这些强大的功能作为攻击你的武器了！使用下面命令来恢复到从前:

$sudo chmod 0755 /usr/bin/python

 

关于脚本文件的saved UID/GID，更加详细的讨论见

http://www.faqs.org/faqs/unix-faq/faq/part4/section-7.html

 

##总结

get*, set*

umask(), uname()
