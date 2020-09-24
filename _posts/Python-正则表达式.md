---
layout: post
title: Python 正则表达式
description: ""
categories: [python]
keywords: [Linux，大数据]
---

正则表达式是一个很强大的字符串处理工具，几乎任何关于字符串的操作都可以使用正则表达式来完成，作为一个爬虫工作者，每天和字符串打交道，正则表达式更是不可或缺的技能，正则表达式的在不同的语言中使用方式可能不一样，不过只要学会了任意一门语言的正则表达式用法，其他语言中大部分也只是换了个函数的名称而已，本质都是一样的。

## 正则表达式基础

正则表达式并不是Python的一部分。正则表达式是用于处理字符串的强大工具，拥有自己独特的语法以及一个独立的处理引擎，效率上可能不如str自带的方法，但功能十分强大。得益于这一点，在提供了正则表达式的语言里，正则表达式的语法都是一样的，区别只在于不同的编程语言实现支持的语法数量不同；但不用担心，不被支持的语法通常是不常用的部分。如果已经在其他语言里使用过正则表达式，只需要简单看一看就可以上手了。

下图展示了使用正则表达式进行匹配的流程： 

[![0SEoMn.png](https://s1.ax1x.com/2020/09/24/0SEoMn.png)](https://imgchr.com/i/0SEoMn)

正则表达式的大致匹配过程是：依次拿出表达式和文本中的字符比较，如果每一个字符都能匹配，则匹配成功；一旦有匹配不成功的字符则匹配失败。如果表达式中有量词或边界，这个过程会稍微有一些不同，但也是很好理解的，看下图中的示例以及自己多使用几次就能明白。

[![0SVPZ6.png](https://s1.ax1x.com/2020/09/24/0SVPZ6.png)](https://imgchr.com/i/0SVPZ6)

### 数量词的贪婪模式与非贪婪模式

正则表达式通常用于在文本中查找匹配的字符串。Python里数量词默认是贪婪的（在少数语言里也可能是默认非贪婪），总是尝试匹配尽可能多的字符；非贪婪的则相反，总是尝试匹配尽可能少的字符。例如：正则表达式"ab*"如果用于查找"abbbc"，将找到"abbb"。而如果使用非贪婪的数量词"ab*?"，将找到"a"。

### 反斜杠的困扰

与大多数编程语言相同，正则表达式里使用"\"作为转义字符，这就可能造成反斜杠困扰。假如你需要匹配文本中的字符"\"，那么使用编程语言表示的正则表达式里将需要4个反斜杠"\\\\"：前两个和后两个分别用于在编程语言里转义成反斜杠，转换成两个反斜杠后再在正则表达式里转义成一个反斜杠。Python里的原生字符串很好地解决了这个问题，这个例子中的正则表达式可以使用r"\\"表示。同样，匹配一个数字的"\\d"可以写成r"\d"。有了原生字符串，你再也不用担心是不是漏写了反斜杠，写出来的表达式也更直观。

### 匹配模式

正则表达式提供了一些可用的匹配模式，比如忽略大小写、多行匹配等，这部分内容将在Pattern类的工厂方法re.compile(pattern[, flags])中一起介绍。

## re模块

Python通过re模块提供对正则表达式的支持。使用re的一般步骤是先将正则表达式的字符串形式编译为Pattern实例，然后使用Pattern实例处理文本并获得匹配结果（一个Match实例），最后使用Match实例获得信息，进行其他的操作。

```python
import re
pattern=re.compile(r'hello')
match=pattern.match('hello world')

if match:
    print(match.group())
输出：
hello
```

## python中的正则表达式

1. 元字符
2. 模式
3. 函数
4. re 内置对象用法
5. 分组用法
6. 环视用法

### 元字符

- .

  匹配任意字符，除了换行符，当re.DOTALL标记被指定时，则可以匹配包括换行符的任意字符

  ```python
  import re
  
  a='xz123'
  b=re.findall('x...',a)
  print(b)
  输出：['xz12']
  ```

- *

  匹配0个或多个的表达式

  ```python
  import re
  a = 'xyxy123'
  b = re.findall('x*',a)
  print(b)
  运行结果：
  ['x', '', 'x', '', '', '', '', '']
  ```

- ?

  匹配0个或1个由前面的正则表达式定义的片段，非贪婪方式

  ```python
  import re
  a = 'xy123'
  b = re.findall('x?',b)
  print(b)
  运行结果：
  ['x', '', '', '', '', '']
  ```

- .*的使用举例

  ```python
  import re
  secrect_code = 'hadkfalifexxIxxfasdjifja134xxlovexx23345sdfxxyouxx8dfse'
  b = re.findall('xx.*xx',secrect_code)
  print(b)
  运行结果：
  ['xxIxxfasdjifja134xxlovexx23345sdfxxyouxx']
  ```

- .*?的使用举例

  ```python
  secrect_code = 'hadkfalifexxIxxfasdjifja134xxlovexx23345sdfxxyouxx8dfse'
  b = re.findall('xx.*?xx',secrect_code)
  print(b)
  运行结果：
  ['xxIxx', 'xxlovexx', 'xxyouxx']
  ```

- (.*？)使用括号与不使用括号的差别

  ( )匹配括号内的表达式，也表示一个组

  ```python
  import re
  
  secrect_code = 'hadkfalifexxIxxfasdjifja134xxlovexx23345sdfxxyouxx8dfse'
  d = re.findall('xx(.*?)xx',secrect_code)
  print(d)
  for each in d:
      print(each)
  运行结果：
  ['I', 'love', 'you']
  I
  love
  you
  ```

- re.S的作用使.的作用包括了\n

  ```python
  import re
  s = '''sdfxxhelloxxfsdf
  xxworldxxasdf'''
  d = re.findall('xx(.*?)xx',s,re.S)
  print(d)
  运行结果：
  ['hello', 'world']
  ```

- $

  匹配字符串的末尾

  ```python
  import re
  s= 'detail.html?key=TISSUE INJURY'
  f = re.findall('key=(.*?)',s)
  print(f)
  运行结果：
  ['']
  
  import re
  s= 'detail.html?key=TISSUE INJURY'
  f = re.findall('key=(.*?)$',s)
  print(f)
  运行结果：
  ['TISSUE INJURY']
  ```

- \d

  匹配任意数字，等价于 [0-9]

  ```python
  import re
  s = 'asdfasf1234567fasd555fas'
  b = re.findall('(\d+)',s)
  print(b)
  运行结果：
  ['1234567', '555']
  
  import re
  s = 'asdfasf1234.567fasd55.5fas'
  b = re.findall('(\d+\.\d+)',s)
  print(b)
  运行结果：
  ['1234.567', '55.5']
  ```

- \ 转义字符

  ```python
  import re
  s = 'sda123sdaslda.s/'
  b = re.search('\.',s)
  print(b)
  print(b.span())
  print(b.group())
  运行结果：
  <_sre.SRE_Match object; span=(13, 14), match='.'>
  (13, 14)
  .
  ```

- a|b

  匹配a或b

  ```python
  import re
  s = 'fishdafishc'
  b= re.search(r'fish(c|d)',s)
  print(b)
  运行结果：
  <_sre.SRE_Match object; span=(0, 5), match='fishd'>
  ```

- {n}

  精确匹配n个前面表达式**

  ```python
  import re
  s ='i fishhhfish.a'
  b = re.search(r'fish{3}',s)
  print(b.group())
  运行结果：
  fishhh
  
  
  s  = 'i fishhhfishfishfish.a'
  b = re.search(r'(fish){3}',s )
  print(b.group())
  运行结果：
  fishfishfish
  ```

- {n,m}

  匹配 n 到 m 次由前面的正则表达式定义的片段，贪婪方式**

  ```python
  import re
  s = 'i fishhhfishfishfish.a'
  b = re.search(r'(fish){2,5}',s)
  print(b.group())
  运行结果：
  fishfishfish
  ```

- sub 的使用举例

  ```python
  import re
  s = '123rrrrr123'
  output = re.sub('123(.*?)123','123%d123'%789,s)
  print(output)
  运行结果：
  123789123
  
  #把一串文本中的所有数字都去掉
  content = '54aK54yr5oiR54ix5L2g'
  content = re.sub('\d+', '', content)
  print(content)
  运行结果：
  aKyroiRixLg
  ```

### findall，search和match的区别

#### match

从首字母开始开始匹配，string如果包含pattern子串，则匹配成功，返回Match对象

#### search

若string中包含pattern子串，则返回Match对象，否则返回None，注意，如果string中存在多个pattern子串，只返回第一个

#### findall

返回string中所有与pattern相匹配的全部字串，返回形式为数组

```python
>>> import re
>>> s = '123.gifgdfkgj.jpg1673.gifgdfk66.jpg'
>>> m = re.match(r'(\d*)(\.gif|\.jpg)',s)
>>> m.group()
'123.gif'
>>> m.group(0)
'123.gif'
>>> m.group(1)
'123'
>>> m.group(2)
'.gif'
>>> m.groups()
('123', '.gif')
------------------------------------------------------------
>>> m = re.findall(r'(\d*)(\.gif|\.jpg)',s)
>>> m
[('123', '.gif'), ('', '.jpg'), ('1673', '.gif'), ('66', '.jpg')]
------------------------------------------------------------
>>> s = 'gdfkg1.jpg1673.gifgdfk66.jpg'
>>> m = re.search(r'.(\d*)(\.gif|\.jpg)',s)
>>> m
<_sre.SRE_Match object; span=(4, 10), match='g1.jpg'>
>>> m.group()
'g1.jpg'
>>> m.group(0)
'g1.jpg'
>>> m.group(1)
'1'
>>> m.group(2)
'.jpg'
>>> m.group(3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: no such group
>>> m.groups()
('1', '.jpg')

```

### 模式

正则表达式可以包含一些可选标志修饰符来控制匹配的模式。修饰符被指定为一个可选的标志。多个标志可以通过按位 OR(|) 它们来指定。如 re.I | re.M 被设置成 I 和 M 标志：

| 修饰符 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| re.I   | 使匹配对大小写不敏感                                         |
| re.L   | 做本地化识别（locale-aware）匹配                             |
| re.M   | 多行匹配，影响 ^ 和 $                                        |
| re.S   | 使 . 匹配包括换行在内的所有字符                              |
| re.U   | 根据Unicode字符集解析字符。这个标志影响 \w, \W, \b, \B.      |
| re.X   | 该标志通过给予你更灵活的格式以便你将正则表达式写得更易于理解。 |

- re.I

```
import re
s = 'hello world!'
regex=re.compile("hello world",re.I)
print(regex.match(s).group())
```

- re.M

```python
import re
s = '''first line
second line
third line'''

regex_start = re.compile("^\w+")
print (regex_start.findall(s))
输出：['first']

regex_start_m = re.compile("^\w+", re.M)
print (regex_start_m.findall(s))
输出：['first', 'second', 'third']

regex_end_m=re.compile("\w+$")
print(regex_end_m.findall(s))
输出：['line']

regex_end_m=re.compile("\w+$",re.M)
print(regex_end_m.findall(s))
输出：['line', 'line', 'line']
```

- re.S

```python
import re
s = '''first line
second line
third line'''

regex = re.compile(".+")
print (regex.findall(s))
输出：['first line', 'second line', 'third line']

regex_dotall = re.compile(".+",re.S)
print(regex_dotall.findall(s))
输出：print(regex_dotall.findall(s))
```

- re.X

```python
email_regex = re.compile("[\w+\.]+@[a-zA-Z\d]+\.(com|cn)")

email_regex = re.compile("""[\w+\.]+  # 匹配@符前的部分
                            @  # @符
                            [a-zA-Z\d]+  # 邮箱类别
                            \.(com|cn)   # 邮箱后缀  """, re.X)
```

正则表达式的模式是可以同时使用多个的，在 python 里面使用按位或运算符 | 同时添加多个模式

如 re.compile('', re.I|re.M|re.S)

每个模式在 re 模块中其实就是不同的数字

```
print re.I
# output> 2
print re.L
# output> 4
print re.M
# output> 8
print re.S
# output> 16
print re.X
# output> 64
print re.U
# output> 32
```

## 描述模式

### 次数/个数

```tex
*匹配0个或多个的表达式。
+匹配1个或多个的表达式。
?匹配0个或1个由前面的正则表达式定义的片段，非贪婪方式
{n}精确匹配n个前面表达式
{n，} 至少有n次
{n, m}匹配 n 到 m 次由前面的正则表达式定义的片段，贪婪方式
```

### 范围

```
[amk]-----字符集合
 匹配 'a'，'m'或'k'。三者中的任何一个
[^abc]------负值字符集合
匹配除了a,b,c之外的字符
[0-9]------数字范围
[a-z]------字符范围
[^a-z]------负值字符范围
```

```
\w匹配字母数字及下划线
\W匹配非字母数字及下划线
\s匹配任意空白字符，等价于 [\t\n\r\f].
\S匹配任意非空字符
\d匹配任意数字，等价于 [0-9]
\D匹配任意非数字
\A匹配字符串开始
\Z匹配字符串结束，如果是存在换行，只匹配到换行前的结束字符串
\z匹配字符串结束
\G匹配最后匹配完成的位置
\n匹配一个换行符
\t匹配一个制表符
^匹配字符串的开头
$匹配字符串的末尾。
.匹配任意字符，除了换行符，当re.DOTALL标记被指定时，则可以匹配包括换行符的任意字符。
a|b匹配a或b
( )匹配括号内的表达式，也表示一个组
```

