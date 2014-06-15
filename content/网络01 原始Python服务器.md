#Python网络01 原始Python服务器


 

之前我的Python教程中有人留言，表示只学Python没有用，必须学会一个框架(比如Django和web.py)才能找到工作。而我的想法是，掌握一个类似于框架的高级工具是有用的，但是基础的东西可以让你永远不被淘汰。不要被工具限制了自己的发展。今天，我在这里想要展示的，就是不使用框架，甚至不使用Python标准库中的高级包，只使用标准库中的socket接口(我不是很明白套接字这个翻译，所以使用socket的英文名字)，写一个Python服务器。

 

在当今Python服务器框架 (framework, 比如Django, Twisted, web.py等等) 横行的时代，从底层的socket开始写服务器似乎是一个出力不讨好的笨方法。框架的意义在于掩盖底层的细节，提供一套对于开发人员更加友好的API，并处理诸如MVC的布局问题。框架允许我们快速的构建一个成型而且成熟的Python服务器。然而，框架本身也是依赖于底层(比如socket)。对于底层socket的了解，不仅可以帮助我们更好的使用框架，更可以让我们明白框架是如何设计的。更进一步，如果拥有良好的底层socket编程知识和其他系统编程知识，你完全可以设计并开发一款自己的框架。如果你可以从底层socket开始，实现一个完整的Python服务器，支持用户层的协议，并处理好诸如MVC(Model-View-Control)、多线程(threading)等问题，并整理出一套清晰的函数或者类，作为接口(API)呈现给用户，你就相当于设计了一个框架。

 

socket接口是实际上是操作系统提供的系统调用。socket的使用并不局限于Python语言，你可以用C或者JAVA来写出同样的socket服务器，而所有语言使用socket的方式都类似(Apache就是使用C实现的服务器)。而你不能跨语言的使用框架。框架的好处在于帮你处理了一些细节，从而实现快速开发，但同时受到Python本身性能的限制。我们已经看到，许多成功的网站都是利用动态语言(比如Python, Ruby或者PHP，比如twitter和facebook)快速开发，在网站成功之后，将代码转换成诸如C和JAVA这样一些效率比较高的语言，从而让服务器能更有效率的面对每天亿万次的请求。在这样一些时间，底层的重要性，就远远超过了框架。

 

下面的一篇文章虽然是在谈JAVA，但我觉得也适用于Python的框架之争。

http://yakovfain.com/2012/10/11/the-degradation-of-java-developers/

 

##TCP/IP和socket

我们需要对网络传输，特别是TCP/IP协议和socket有一定的了解。socket是进程间通信的一种方法 (参考Linux进程间通信)，它是基于网络传输协议的上层接口。socket有许多种类型，比如基于TCP协议或者UDP协议(两种网络传输协议)。其中又以TCP socket最为常用。TCP socket与双向管道(duplex PIPE)有些类似，一个进程向socket的一端写入或读取文本流，而另一个进程可以从socket的另一端读取或写入，比较特别是，这两个建立socket通信的进程可以分别属于两台不同的计算机。所谓的TCP协议，就是规定了一些通信的守则，以便在网络环境下能够有效实现上述进程间通信过程。双向管道(duplex PIPE)存活于同一台电脑中，所以不必区分两个进程的所在计算机的地址，而socket必须包含有地址信息，以便实现网络通信。一个socket包含四个地址信息: 两台计算机的IP地址和两个进程所使用的端口(port)。IP地址用于定位计算机，而port用于定位进程 (一台计算机上可以有多个进程分别使用不同的端口)。

 



一个TCP socket连接的网络

 

##TCP socket

在互联网上，我们可以让某台计算机作为服务器。服务器开放自己的端口，被动等待其他计算机连接。当其他计算机作为客户，主动使用socket连接到服务器的时候，服务器就开始为客户提供服务。

 

在Python中，我们使用标准库中的socket包来进行底层的socket编程。

首先是服务器端，我们使用bind()方法来赋予socket以固定的地址和端口，并使用listen()方法来被动的监听该端口。当有客户尝试用connect()方法连接的时候，服务器使用accept()接受连接，从而建立一个连接的socket：

```python
# Written by Vamei
# Server side
import socket

# Address
HOST = ''
PORT = 8000

reply = 'Yes'

# Configure socket
s      = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((HOST, PORT))

# passively wait, 3: maximum number of connections in the queue
s.listen(3)
# accept and establish connection
conn, addr = s.accept()
# receive message
request    = conn.recv(1024)

print 'request is: ',request
print 'Connected by', addr
# send message
conn.sendall(reply)
# close connection
conn.close()
```
socket.socket()创建一个socket对象，并说明socket使用的是IPv4(AF_INET，IP version 4)和TCP协议(SOCK_STREAM)。

 

然后用另一台电脑作为客户，我们主动使用connect()方法来搜索服务器端的IP地址(在Linux中，你可以用$ifconfig来查询自己的IP地址)和端口，以便客户可以找到服务器，并建立连接:

```python
# Written by Vamei
# Client side
import socket

# Address
HOST = '172.20.202.155'
PORT = 8000

request = 'can you hear me?'

# configure socket
s       = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))

# send message
s.sendall(request)
# receive message
reply   = s.recv(1024)
print 'reply is: ',reply
# close connection
s.close()
```
在上面的例子中，我们对socket的两端都可以调用recv()方法来接收信息，调用sendall()方法来发送信息。这样，我们就可以在分处于两台计算机的两个进程间进行通信了。当通信结束的时候，我们使用close()方法来关闭socket连接。

(如果没有两台计算机做实验，也可以将客户端IP想要connect的IP改为"127.0.0.1"，这是个特殊的IP地址，用来连接当地主机。)

 

##基于TCP socket的HTTP服务器

上面的例子中，我们已经可以使用TCP socket来为两台远程计算机建立连接。然而，socket传输自由度太高，从而带来很多安全和兼容的问题。我们往往利用一些应用层的协议(比如HTTP协议)来规定socket使用规则，以及所传输信息的格式。

 

HTTP协议利用请求-回应(request-response)的方式来使用TCP socket。客户端向服务器发一段文本作为request，服务器端在接收到request之后，向客户端发送一段文本作为response。在完成了这样一次request-response交易之后，TCP socket被废弃。下次的request将建立新的socket。request和response本质上说是两个文本，只是HTTP协议对这两个文本都有一定的格式要求。

 



 

request-response cycle

 

现在，我们写出一个HTTP服务器端：
```python
# Written by Vamei

import socket

# Address
HOST = ''
PORT = 8000

# Prepare HTTP response
text_content = '''HTTP/1.x 200 OK  
Content-Type: text/html

<head>
<title>WOW</title>
</head>
<html>
<p>Wow, Python Server</p>
<IMG src="test.jpg"/>
</html>
'''

# Read picture, put into HTTP format
f = open('test.jpg','rb')
pic_content = '''HTTP/1.x 200 OK  
Content-Type: image/jpg

'''
pic_content = pic_content + f.read()
f.close()

# Configure socket
s    = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((HOST, PORT))

# infinite loop, server forever
while True:
    # 3: maximum number of requests waiting
    s.listen(3)
    conn, addr = s.accept()
    request    = conn.recv(1024)
    method    = request.split(' ')[0]
    src            = request.split(' ')[1]

    # deal with GET method
    if method == 'GET':
        # ULR    
        if src == '/test.jpg':
            content = pic_content
        else: content = text_content

        print 'Connected by', addr
        print 'Request is:', request
        conn.sendall(content)
    # close connection
    conn.close()
```

##深入HTTP服务器程序

如我们上面所看到的，服务器会根据request向客户传输的两条信息text_content和pic_content中的一条，作为response文本。整个response分为起始行(start line), 头信息(head)和主体(body)三部分。起始行就是第一行:

HTTP/1.x 200 OK
它实际上又由空格分为三个片段，HTTP/1.x表示所使用的HTTP版本，200表示状态(status code)，200是HTTP协议规定的，表示服务器正常接收并处理请求，OK是供人来阅读的status code。

 

头信息跟随起始行，它和主体之间有一个空行。这里的text_content或者pic_content都只有一行的头信息，text_content用来表示主体信息的类型为html文本：

Content-Type: text/html
而pic_content的头信息(Content-Type: image/jpg)说明主体的类型为jpg图片(image/jpg)。

 

主体信息为html或者jpg文件的内容。

(注意，对于jpg文件，我们使用'rb'模式打开，是为了与windows兼容。因为在windows下，jpg被认为是二进制(binary)文件，在UNIX系统下，则不需要区分文本文件和二进制文件。)

 

我们并没有写客户端程序，后面我们会用浏览器作为客户端。request由客户端程序发给服务器。尽管request也可以像response那样分为三部分，request的格式与response的格式并不相同。request由客户发送给服务器，比如下面是一个request：

GET /test.jpg HTTP/1.x
Accept: text/*
 

起始行可以分为三部分，第一部分为请求方法(request method)，第二部分是URL，第三部分为HTTP版本。request method可以有GET， PUT， POST， DELETE， HEAD。最常用的为GET和POST。GET是请求服务器发送资源给客户，POST是请求服务器接收客户送来的数据。当我们打开一个网页时，我们通常是使用GET方法；当我们填写表格并提交时，我们通常使用POST方法。第二部分为URL，它通常指向一个资源(服务器上的资源或者其它地方的资源)。像现在这样，就是指向当前服务器的当前目录的test.jpg。

按照HTTP协议的规定，服务器需要根据请求执行一定的操作。正如我们在服务器程序中看到的，我们的Python程序先检查了request的方法，随后根据URL的不同，来生成不同的response(text_content或者pic_content)。随后，这个response被发送回给客户端。

 

##使用浏览器实验

为了配合上面的服务器程序，我已经在放置Python程序的文件夹里，保存了一个test.jpg图片文件。我们在终端运行上面的Python程序，作为服务器端，再打开一个浏览器作为客户端。(如果有时间，你也完全可以用Python写一个客户端。原理与上面的TCP socket的客户端程序相类似。)

在浏览器的地址栏输入:

127.0.0.1:8000
 

(当然，你也可以用令一台电脑，并输入服务器的IP地址。) 我得到下面的结果:

 



OK，我已经有了一个用Python实现的，并从socket写起的服务器了。

从终端，我们可以看到，浏览器实际上发出了两个请求。第一个请求为 (关键信息在起始行，这一个请求的主体为空):

```bash
GET / HTTP/1.1
Host: 127.0.0.1:8000
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:14.0) Gecko/20100101 Firefox/14.0.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
```

我们的Python程序根据这个请求，发送给服务器text_content的内容。

 

浏览器接收到text_content之后，发现正文的html文本中有<IMG src="text.jpg" />，知道需要获得text.jpg文件来补充为图片，立即发出了第二个请求:

```bash
GET /test.jpg HTTP/1.1
Host: 127.0.0.1:8000
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:14.0) Gecko/20100101 Firefox/14.0.1
Accept: image/png,image/*;q=0.8,*/*;q=0.5
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://127.0.0.1:8000/
```

我们的Python程序分析过起始行之后，发现/test.jpg符合if条件，所以将pic_content发送给客户。

最后，浏览器根据html语言的语法，将html文本和图画以适当的方式显示出来。(html可参考http://www.w3schools.com/html/default.asp)

 

##探索的方向

1) 在我们上面的服务器程序中，我们用while循环来让服务器一直工作下去。实际上，我们还可以根据我之前介绍的多线程的知识，将while循环中的内容改为多进程或者多线程工作。(参考Python多线程与同步，Python多进程初步，Python多进程探索)

2) 我们的服务器程序还不完善，我们还可以让我们的Python程序调用Python的其他功能，以实现更复杂的功能。比如说制作一个时间服务器，让服务器向客户返回日期和时间。你还可以使用Python自带的数据库，来实现一个完整的LAMP服务器。

3) socket包是比较底层的包。Python标准库中还有高层的包，比如SocketServer，SimpleHTTPServer，CGIHTTPServer，cgi。这些都包都是在帮助我们更容易的使用socket。如果你已经了解了socket，那么这些包就很容易明白了。利用这些高层的包，你可以写一个相当成熟的服务器。

4) 在经历了所有的辛苦和麻烦之后，你可能发现，框架是那么的方便，所以决定去使用框架。或者，你已经有了参与到框架开发的热情。

 

更多内容

TCP/IP和port参考: TCP/IP illustrated http://book.douban.com/subject/1741925/

socket参考: UNIX Network Programming http://book.douban.com/subject/1756533/

           Python socket 官方文档 http://docs.python.org/2/library/socket.html

HTTP参考: HTTP, the definitive guide http://book.douban.com/subject/1440226/

