---
#标题
title: 根据多条件更高效的在DataFrame中加入新列
date: 2018-7-16  20:34:23
categories: 
- 问题优化解决
- GE
tags: 
- DataFrame
- Python
---
在实际应用中，常常需要根据多个条件，为DataFrame加入新列。因此，本篇博文给出两种在DataFrame中加入新列的方法。其中，使用for循环的方法较为低效，在实际开发过程中应尽量避免使用。使用apply的方法更为高效，推荐使用该方法。 <!--more-->

## 问题陈述

有如下一个DataFrame df:

```python
    A    B
a   2    2
b   3    1
c   1    3
```

想要基于下面的条件添加一个新列：
	1. if row A == B : 0
	2. if row A >  B : 1
	3. if row A <  B : -1
因此，给定上面的表，得到的结果为：

```python
    A    B    C
a   2    2    0
b   3    1    1
c   1    3   -1
```


## 解决方案1
通过循环遍历将得到新列的值存入数组，再插入DataFrame df。

```python
def add_column(df):
	c = []
	for i in range(row.shape[0]):
		if df['A'][i] == df['B'][i]:
			c.append(0)
		elif df['A'][i] > df['B'][i]:
			c.append(1)
		else:
			c.append(-1)
	df.insert(df.shape[1], 'C', c)
	return df

df = add_column(df)
```

## 解决方案2
形式化上面列出的一些方法：创建一个在DataFrame的行上运行的函数，就像这样，[参考链接](https://stackoverflow.com/questions/21702342/creating-a-new-column-based-on-if-elif-else-condition):

```python
def f(row):
    if row['A'] == row['B']:
        val = 0
    elif row['A'] > row['B']:
        val = 1
    else:
        val = -1
    return val
```

然后将其应用于DataFrame，并设置axis = 1

```python
In [1]: df['C'] = df.apply(f, axis=1)

In [2]: df
Out[2]:
   A  B  C
a  2  2  0
b  3  1  1
c  1  3 -1
```

