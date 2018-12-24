---
#标题
title: input()与sys.stdin.readline()的区别
date: 2018-6-23  20:34:23
---
在Python标准库的sys库中，有三个文件描述符，分别是stdin、stdout和stderr，这三个文件描述符分别代表标准输入、标准输出和错误输出。sys.stdin.readline()会将标准的输入全部获取，包括末尾的'\n',但是input()获取的输入不包括换行符'\n'。通过在Python Shell 3验证两者的区别。<!--more-->

## input

input()获取的输入不包括换行符'\n'。

```python
>>> s = input()
abc
>>> print(len(s))
3
>>>
```


## sys.stdin.readline()

sys.stdin.readline()会将标准的输入全部获取，包括末尾的'\n'。

```python
>>> import sys
>>> s2 = sys.stdin.readline()
abc
>>> print(len(s2))
4
>>> print(s2)
abc

>>>
```

去掉末尾'\n'的两种方法如下：
第一种：

```python
>>> import sys
>>> s2 = sys.stdin.readline()
abc
>>> print(len(s2))
4
>>> s3 = sys.stdin.readline().replace('\n','')
abc
>>> print(len(s3))
3
```

第二种：

```python
>>> import sys
>>> s2 = sys.stdin.readline()
abc
>>> print(len(s2))
4
>>> s3 = sys.stdin.readline().strip('\n')
abc
>>> print(len(s3))
3
```
