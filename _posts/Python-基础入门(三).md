---
layout: post
title: Python 基础入门(三)
description: ""
categories: [Python]
keywords: [python]

---

## 输入/输出

Python两种输出值的方式，表达式语句和print()函数。第三种方式是使用文件对象的write()方法，标准输出文件可以用sys.stdout的引用。

在很多时候，你会想要让你的程序与用户（可能是你自己）交互。你会从用户那里得到输入，然后打印一些结果。我们可以分别使用`raw_input`和`print`语句来完成这些功能。对于输出，你也可以使用多种多样的`str`（字符串）类。例如，你能够使用`rjust`方法来得到一个按一定宽度右对齐的字符串。利用`help(str)`获得更多详情。

### 读取键盘输入

python提供了input()内置函数从标准输入读入一行文本，默认的标准输入是键盘。

input可以接受一个Python表达式作为输入，然后从运算结果返回。

```python
str = input('please input:')
print("the context is :", str)
输出：
please input:hu
the context is : hu
```

### 读和写文件

open()将会返回一个file文件，基本语法格式如下：

open(filename,mode)

filename:包含了你要访问的文件名的字符串值。

mode:决定了打开文件的模式：只读，写入，追加等。

```python
f = open("demo.txt", "w")
f.write("test str")
f.close()
输出：
 ✘ kristennn@aNdyzz  ~/PycharmProjects/teachdemo  cat demo.txt 
test str%                 
```

### 文件对象的方法

- **f.read()**

  为了读取一个文件的内容，调用f.read(size)，这将读取一定数目的数据，然后作为字符串或者字节对象返回。

  size是一个可选的数字类型的参数。当size被忽略了或为负时，那么该文件的所有内容都将被读取并且返回。

  ```python
  f = open("demo.txt", "r")
  str = f.read()
  print(str)
  f.close()
  输出：
  test str
  ```

- **f.readline()**

  该函数会从一个文件中读取单独的一行换行符为’\n’。f.readline()如果返回一个空字符串，说明已经读取到最后一行。

  ```python
  f = open("demo.txt", "r")
  
  while True:
      str=f.readline()
      if len(str)==0:
          break
      print(str)
  
  f.close()
  输出：
  test str
  
  test str2
  
  test str3
  ```

- **f.readlines()**

  该函数会返回文件所包含的所有行。

  如果参数可选参数sizehint，则读取指定长度的字节，并且将这些字节按行分隔。

  ```python
  f = open("demo.txt", "r")
  str = f.readlines()
  print(str)
  f.close()
  输出：
  ['test str\n', 'test str2\n', 'test str3']
  ```

- **f.write()**

  f.write(string)将string写入到文件中，然后返回写入的字符数。

  ```python
  f = open("demo.txt", "w")
  value = f.write("i am GOD")
  print(value)
  f.close()
  输出
  8
  ```

- **f.close()**

  在文本文件中（那些打开文件的模式下没有b的），只会想相对于文件起始位置进行定位。

  当你处理完一个文件后，调用f.close()来关闭文件并释放系统资源，如果尝试再调用该文件，则会抛出异常。

  当处理一个文件对象时，使用with关键字也是非常好的方法。在结束时他会帮助你正确的关闭文件，并且写起来也要比try-finally语句块要简短

## 异常

当你的程序中出现某些 异常的 状况的时候，异常就发生了。例如，当你想要读某个文件的时候，而那个文件不存在。或者在程序运行的时候，你不小心把它删除了。上述这些情况可以使用**异常**来处理。

### 处理异常

我们可以使用`try..except`语句来处理异常。我们把通常的语句放在try-块中，而把我们的错误处理语句放在except-块中。

```python
try:
    s = input("enter string")
except EOFError:
    print("why you bully me?")
    sys.exit()
except:
    print("some unknow error")
print("done")
输出：
enter string1111^D
why you bully me?
done
```

我们把所有可能引发错误的语句放在`try`块中，然后在`except`从句/块中处理所有的错误和异常。`except`从句可以专门处理单一的错误或异常，或者一组包括在圆括号内的错误/异常。如果没有给出错误或异常的名称，它会处理 所有的 错误和异常。对于每个`try`从句，至少都有一个相关联的`except`从句。

如果某个错误或异常没有被处理，默认的Python处理器就会被调用。它会终止程序的运行，并且打印一个消息，我们已经看到了这样的处理。

你还可以让`try..catch`块关联上一个`else`从句。当没有异常发生的时候，`else`从句将被执行。

### 引发异常

你可以使用`raise`语句 引发 异常。你还得指明错误/异常的名称和伴随异常 触发的 异常对象。你可以引发的错误或异常应该分别是一个`Error`或`Exception`类的直接或间接导出类。

```python
class ShortInputException(Exception):
    def __init__(self, length, atleast):
        Exception.__init__(self)
        self.length = length
        self.atleast = atleast

try:
    s = input('Enter something --> ')
    if len(s) < 3:
        raise ShortInputException(len(s), 3)
except EOFError:
    print("why u bully me")
except ShortInputException as x:
    print("ShortInputException: The input was of length", x.length, " was expecting at least", x.atleast)
else:
    print("No exception was raised.")
输出：
Enter something --> 2
ShortInputException: The input was of length 1  was expecting at least 3
```

这里，我们创建了我们自己的异常类型，其实我们可以使用任何预定义的异常/错误。这个新的异常类型是`ShortInputException`类。它有两个域——`length`是给定输入的长度，`atleast`则是程序期望的最小长度。

## Python标准库

Python标准库是随Python附带安装的，它包含大量极其有用的模块。熟悉Python标准库是十分重要的，因为如果你熟悉这些库中的模块，那么你的大多数问题都可以简单快捷地使用它们来解决。

### 操作系统接口

```python
import os

str = os.getcwd()
print(str)
输出：
/Users/kristennn/PycharmProjects/teachdemo

```

