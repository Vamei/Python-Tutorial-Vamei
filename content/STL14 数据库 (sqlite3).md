# Python标准库14 数据库 (sqlite3)

Python自带一个轻量级的关系型数据库SQLite。这一数据库使用SQL语言。SQLite作为后端数据库，可以搭配Python建网站，或者制作有数据存储需求的工具。SQLite还在其它领域有广泛的应用，比如HTML5和移动端。Python标准库中的sqlite3提供该数据库的接口。

我将创建一个简单的关系型数据库，为一个书店存储书的分类和价格。数据库中包含两个表：category用于记录分类，book用于记录某个书的信息。一本书归属于某一个分类，因此book有一个外键(foreign key)，指向catogory表的主键id。

![](191548418481935.png)

### 创建数据库

我首先来创建数据库，以及数据库中的表。在使用connect()连接数据库后，我就可以通过定位指针cursor，来执行SQL命令：
```py
import sqlite3 
# test.db is a file in the working directory.
conn = sqlite3.connect("test.db")

c = conn.cursor() 
# create tables
c.execute('''CREATE TABLE category
      (id int primary key, sort int, name text)''')
c.execute('''CREATE TABLE book
      (id int primary key, 
       sort int, 
       name text, 
       price real, 
       category int,
       FOREIGN KEY (category) REFERENCES category(id))''') 
# save the changes
conn.commit() 
# close the connection with the database
conn.close()
```
SQLite的数据库是一个磁盘上的文件，如上面的test.db，因此整个数据库可以方便的移动或复制。test.db一开始不存在，所以SQLite将自动创建一个新文件。

利用execute()命令，我执行了两个SQL命令，创建数据库中的两个表。创建完成后，保存并断开数据库连接。

## 插入数据

上面创建了数据库和表，确立了数据库的抽象结构。下面将在同一数据库中插入数据：
```py
import sqlite3

conn = sqlite3.connect("test.db")
c = conn.cursor()  books = [(1, 1, 'Cook Recipe', 3.12, 1),
            (2, 3, 'Python Intro', 17.5, 2),
            (3, 2, 'OS Intro', 13.6, 2),
           ] 
# execute "INSERT" 
c.execute("INSERT INTO category VALUES (1, 1, 'kitchen')") 
# using the placeholder
c.execute("INSERT INTO category VALUES (?, ?, ?)", [(2, 2, 'computer')]) 
# execute multiple commands
c.executemany('INSERT INTO book VALUES (?, ?, ?, ?, ?)', books)

conn.commit()
conn.close()
```

插入数据同样可以使用execute()来执行完整的SQL语句。SQL语句中的参数，使用"?"作为替代符号，并在后面的参数中给出具体值。这里不能用Python的格式化字符串，如"%s"，因为这一用法容易受到SQL注入攻击。

我也可以用executemany()的方法来执行多次插入，增加多个记录。每个记录是表中的一个元素，如上面的books表中的元素。

## 查询

在执行查询语句后，Python将返回一个循环器，包含有查询获得的多个记录。你循环读取，也可以使用sqlite3提供的fetchone()和fetchall()方法读取记录：
```py
import sqlite3

conn = sqlite3.connect('test.db')
c = conn.cursor() 
# retrieve one record
c.execute('SELECT name FROM category ORDER BY sort') print(c.fetchone()) print(c.fetchone()) 
# retrieve all records as a list
c.execute('SELECT * FROM book WHERE book.category=1') print(c.fetchall()) 
# iterate through the records
for row in c.execute('SELECT name, price FROM book ORDER BY sort'): print(row)
```

## 更新与删除

你可以更新某个记录，或者删除记录：

```py
 conn = sqlite3.connect("test.db")
c = conn.cursor()

c.execute('UPDATE book SET price=? WHERE id=?',(1000, 1))
c.execute('DELETE FROM book WHERE id=2')

conn.commit()
conn.close()
```

你也可以直接删除整张表：
```py
c.execute('DROP TABLE book')
```
如果删除test.db，那么整个数据库会被删除。

## 总结

sqlite3只是一个SQLite的接口。想要熟练的使用SQLite数据库，还需学习更多的**关系型数据库**的知识。
