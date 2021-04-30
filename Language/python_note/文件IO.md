# IO编程

## 文件读写

### 读文件

```python
# 打开文件，返回一个文件对象
fileobject = open('1.txt', 'r')

# 读文件对象内容
# 1. 全文读取
s = fileobject.read()
# 2. 按字节读取
a = fileobject.read(5)
# 3. 按行读取文件，通过循环可以读取所有行
b = fileobject.readline()
# 4. 一次读取所有内容并按行返回一个list，循环读取list即可
list = fileobject.readlines()

# 全部读取完毕后，记得关闭文件
fileobject.close()
```

* 有的时候文件不存在，因此open的时候会出错，那么为了让文件在出错的时候也能够正常关闭，以免占用系统资源，可以利用`try...finally...`方式。

```python
try:
    f = open('/path/to/file', 'r')
    print(f.read())
finally:
    if f:
        f.close()
```

* 每次都这么写实在太麻烦了，引入with语句自动帮我们调用close()方法`with open('1.txt', 'r')`

#### 读文本文件

一般文本文件使用utf-8编码的时候，我们可以利用如下语句打开

```python
f = open('1.txt', 'r', encoding="UTF-8")
```

如果读取非UTF-8编码的文本文件，比如读取GBK编码的文件，利用如下语句

```python
f = open('1.txt', 'r', encoding="gbk")
```

#### 读二进制文件

二进制文件如图片等，可以利用如下模式打开：

```python
python>>> f = open('test.jpg', 'rb')
print(f.read())
b'\xff\xd8\xff\xe1\x00\x18Exif\x00\x00...' # 十六进制表示的字节
```

* 遇到有些编码不规范的文件，你可能会遇到`UnicodeDecodeError`，因为在文本文件中可能夹杂了一些非法编码的字符。遇到这种情况，`open()`函数还接收一个`errors`参数，表示如果遇到编码错误后如何处理。最简单的方式是直接忽略：

```python
f = open('gbk.txt', 'r', encoding='gbk', errors='ignore')
```

### 写文件

```python
# 以UTF-8编码格式写入内容，并覆盖原文件
with open('1.txt', 'w', encoding='utf-8') as f:
    f.write('123Hl算法12343534534534535')
# 以gbk编码格式，并追加写入内容 
with open('1.txt', 'a', encoding='gbk') as f:
    f.write('123Hl算法12343534534534535')
```

## StringIO和BytesIO

### StringIO

StringIO顾名思义就是在内存中读写str。

```python
from io import StringIO
# 首先创建一个StingIO对象
f = StringIO()

# 分别向内存中写入如下内容
f.write('hello')
f.write(',\n')
f.write('world!\ndd\n')

# 通过getvalue获取到写入的值
print(f.getvalue())
```

``` python
# 要读取StringIO，可以用一个str初始化StringIO，然后，像读文件一样读取
from io import StringIO
f = StringIO('Hello!\nHi!\nGoodbye!')
while True:
    s = f.readline()
    if s == '':
        break
    print(s.strip())
```

### ByteIO

StringIO操作的只能是str，如果要操作二进制数据，就需要使用BytesIO。

BytesIO实现了在内存中读写bytes，我们创建一个BytesIO，然后写入一些bytes：

```python
from io import BytesIO

f = BytesIO()
# encode()是str的一个方法，‘中文’.encode()，是将‘中文’这个字符串，变成了bytes；
f.write('中文'.encode('utf-8'))
print(f.getvalue())
```

请注意，写入的不是str，而是经过UTF-8编码的bytes。

和StringIO类似，可以用一个bytes初始化BytesIO，然后，像读文件一样读取：

```python
ff = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
print(ff.read())
```

### stream position

>Tip：seek()函数

概述：seek()** 方法用于移动文件读取指针到指定位置。

seek() 方法语法如下：

```python
fileObject.seek(offset[, whence])
```

参数

- **offset** -- 开始的偏移量，也就是代表需要移动偏移的字节数
- **whence：**可选，默认值为 0。给offset参数一个定义，表示要从哪个位置开始偏移；0代表从文件开头开始算起，1代表从当前位置开始算起，2代表从文件末尾算起。

返回值：如果操作成功，则返回新的文件位置，如果操作失败，则函数返回 -1。

> Tip：tell()函数

**tell()** 方法返回文件的当前位置，即文件指针当前位置。

tell() 方法语法如下：

```
fileObject.tell()
```

返回值: 返回文件的当前位置。

**注意**

```python
from io import StringIO
# >>>>>>>>>>>>>> 第一种方式
f = StringIO()
f.write('Hello\nworld\n!')
# 这里引入了 stream position的概念，当我们对StringIO流通过write写入内容时，指针指向了字符串的末尾
# 此时通过readline读取的内容从指针指向的位置开始，因此为空。我们可以通过tell方法看一下文件的当前位置
print(f.tell())   # 结果是13，即字符串的尾部
s = f.readline()  
print(s)          # 输出为空

# 若想输出成功，可以通过seed方法将指针指向字符串开头
f.seek(0)
s = f.readline()
print(s)

# >>>>>>>>>>>>>> 第二种方式
# 用一个str初始化StringIO
f = StringIO('Hello\nworld\n!')
# 这种方式，指针指向了开头，可以用tell方法测试一下
print(f.tell(),'\n')  # 输出0
# 此时可以通过readline直接输出
while True:
    s = f.readline()
    if s == '':
        break
    print(s.strip())
```

## 操作文件和目录

如果我们要操作文件、目录，可以在命令行下面输入操作系统提供的各种命令来完成。比如`dir`、`cp`等命令。

如果要在Python程序中执行这些目录和文件的操作怎么办？其实操作系统提供的命令只是简单地调用了操作系统提供的接口函数，Python内置的`os`模块也可以直接调用操作系统提供的接口函数

打开Python交互式命令行，我们来看看如何使用`os`模块的基本功能：

```python
import os
# 输出当前操作系统的名字，如果是`posix`，说明系统是`Linux`、`Unix`或`Mac OS X`，如果是`nt`，就是`Windows`系统。
print(os.name)
# 要获取详细的系统信息，可以调用uname()函数：windows不可用该函数
os.uname()
```

### 环境变量

```python
# 在操作系统中定义的环境变量，全部保存在os.environ这个变量中，可以直接查看：
print(os.environ)
# 要获取某个环境变量的值，可以调用os.environ.get('key')：
print(os.environ.get('PATH'))
```

小练习：读系统环境变量发现输出内容非常多，且没有换行，如何分行显示

![image-20210429160021435](https://i.loli.net/2021/04/29/UBFpdfhV8YgwQqA.png)

```python
import os
from io import StringIO

# 将系统path环境变量输出，并将分号;替换成换行符号\n
str1 = os.environ.get('PATH').replace(';', '\n')

# 将该字符串读入到系统内存中
s = StringIO(str1)
# 分行读取内容并输出
while True:
    m = s.readline()
    if m == '':
        break
    print(m.strip())
```

### 操作文件和目

操作文件和目录的函数一部分放在`os`模块中，一部分放在`os.path`模块中，这一点要注意一下。查看、创建和删除目录可以这么调用：

```python
import os

# 查看当前目录的绝对路径:
print(os.path.abspath('.'))

# 在当前路径下，想要新建一个testdir目录
# 拼接想要增加的目录名称，显示新目录的完整路径
print(os.path.join('d:\Coding\python_code\python_study_demo\IO编程', 'testdir'))
# 输出  d:\Coding\python_code\python_study_demo\IO编程\testdir

# 创建该目录
# 方式一：利用路径拼接返回一个想要创建的目录的完整路径，然后利用mkdir创建
os.mkdir(os.path.join('d:\Coding\python_code\python_study_demo\IO编程', 'testdir1'))
# 方式二：直接写上完整路径
os.mkdir(r'd:\Coding\python_code\python_study_demo\IO编程\testdir2')

# 删除该目录，也是两种方式类似于创建，只不过方法替换成了rmdir
os.rmdir(os.path.join('d:\Coding\python_code\python_study_demo\IO编程', 'testdir1'))
os.rmdir(r'd:\Coding\python_code\python_study_demo\IO编程\testdir2')
```

#### 拼接路径

把两个路径合成一个时，不要直接拼字符串，而要通过`os.path.join()`函数，这样可以正确处理不同操作系统的路径分隔符。

在Linux/Unix/Mac下，`os.path.join()`返回这样的字符串：

```
part-1/part-2
```

而Windows下会返回这样的字符串：

```
part-1\part-2
```

#### 拆分路径

要拆分路径时，也不要直接去拆字符串，而要通过`os.path.split()`函数，这样可以把一个路径拆分为两部分，后一部分总是最后级别的目录或文件名

```
>>> os.path.split('/Users/michael/testdir/file.txt')
('/Users/michael/testdir', 'file.txt')
```

`os.path.splitext()`可以直接让你得到文件扩展名，很多时候非常方便：

```
>>> os.path.splitext('/path/to/file.txt')
('/path/to/file', '.txt')
```

这些合并、拆分路径的函数并不要求目录和文件要真实存在，它们只对字符串进行操作。

### 重命名、删除、复制文件

文件操作使用下面的函数。假定当前目录下有一个`test.txt`文件：

```python
# 对文件重命名:
>>> os.rename('test.txt', 'test.py')
# 删掉文件:
>>> os.remove('test.py')
import shutil
shutil.copyfile('1.txt', '2.txt')
```

但是复制文件的函数居然在`os`模块中不存在！原因是复制文件并非由操作系统提供的系统调用。理论上讲，我们通过上一节的读写文件可以完成文件复制，只不过要多写很多代码。

幸运的是`shutil`模块提供了`copyfile()`的函数，你还可以在`shutil`模块中找到很多实用函数，它们可以看做是`os`模块的补充。

最后看看如何利用Python的特性来过滤文件。比如我们要列出当前目录下的所有目录，只需要一行代码：

```python
>>> [x for x in os.listdir('.') if os.path.isdir(x)]
['.lein', '.local', '.m2', '.npm', '.ssh', '.Trash', '.vim', 'Applications', 'Desktop', ...]
```

要列出所有的`.py`文件，也只需一行代码：

```python
>>> [x for x in os.listdir('.') if os.path.isfile(x) and os.path.splitext(x)[1]=='.py']
['apis.py', 'config.py', 'models.py', 'pymonitor.py', 'test_db.py', 'urls.py', 'wsgiapp.py']
```

## 序列化

python提供pickle模块实现序列化

序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

反过来，把变量内容从序列化的对象重新读到内存里称之为反序列化，即unpickling。

分别用到的方法有:

`pickle.dumps()、pickle.dump()、pickle.loads()、pickle.load()`

```python
import pickle

d = dict(name='Bob', age=20, score=88)
# pickle.dumps()方法把任意对象序列化成一个bytes，然后就可以把这个bytes写入文件
print(pickle.dumps(d))

with open('test.txt', 'wb') as f:
    # pickle.dump()直接把对象序列化后写入一个file-like Object
    pickle.dump(d, f)

# pickle.loads方法
with open('test.txt','rb') as f:
    # 反序列化
    print(pickle.loads(f.read()))

# pickle.load方法
with open('test.txt','rb') as t:
    # pickle.load方法接受一个文件，从一个file-like Object中直接反序列化出对象
    s = pickle.load(t)
    print(s)
```

Pickle的问题和所有其他编程语言特有的序列化问题一样，就是它只能用于Python，并且可能不同版本的Python彼此都不兼容，因此，只能用Pickle保存那些不重要的数据，不能成功地反序列化也没关系。

### josn序列化

Python内置的`json`模块提供了非常完善的Python对象到JSON格式的转换。我们先看看如何把Python对象变成一个JSON：

```
>>> import json
>>> d = dict(name='Bob', age=20, score=88)
>>> json.dumps(d)
'{"age": 20, "score": 88, "name": "Bob"}'
```

`dumps()`方法返回一个`str`，内容就是标准的JSON。类似的，`dump()`方法可以直接把JSON写入一个`file-like Object`。

要把JSON反序列化为Python对象，用`loads()`或者对应的`load()`方法，前者把JSON的字符串反序列化，后者从`file-like Object`中读取字符串并反序列化：

```
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> json.loads(json_str)
{'age': 20, 'score': 88, 'name': 'Bob'}
```

由于JSON标准规定JSON编码是UTF-8，所以我们总是能正确地在Python的`str`与JSON的字符串之间转换。