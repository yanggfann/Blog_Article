---
#标题
title: 字符串模糊匹配---Python fuzzywuzzy库
date: 2018-7-6  20:34:23
categories: 
- GE实习
- fuzzywuzzy模糊查找
tags:
- GE 
- Python
---
**[fuzzywuzzy](https://github.com/seatgeek/fuzzywuzzy)**模糊字符串匹配使用Levenshtein Distance来计算序列之间的差异。 <!--more-->

## fuzzywuzzy安装

方法一：

```python
pip install fuzzywuzzy
```

方法二，在Anaconda上安装：
1.启动anaconda命令窗口：开始->所有程序->anaconda->anaconda prompt
2.在anaconda prompt中输入pip install 路径 + whl文件名

## Levenshtein距离

fuzzywuzzy进行模糊匹配时所用到的求相似度的距离是[Levenshtein diatance](https://en.wikipedia.org/wiki/Levenshtein_distance)

* **[Levenshtein距离](https://www.jianshu.com/p/de3064109bb2)简介**
Levenshtein 距离是一种编辑距离，用来表示两个字符串的差异。编辑距离是指从字符串 A 开始，修改成字符串 B 的最小步骤数，每个以步骤中，你可以删除一个字符、修改一个字符或者新增一个字符。

比如我们把 acat 变成 gate 的时候，需要做如下的修改：
>删除 a
>把 c 改成 g
>新增 e
所以 acat 和 gate 的 Levenshtein 距离是 3。

## fuzzywuzzy用法

* **Usage**

```python
>>> from fuzzywuzzy import fuzz
>>> from fuzzywuzzy import process
```

* **Simple Ratio**

```python
>>> fuzz.ratio("this is a test", "this is a test!")
    97
```

* **Partial Ratio**

```python
>>> fuzz.partial_ratio("this is a test", "this is a test!")
    100
```

* **Token Sort Ratio**

```python
>>> fuzz.ratio("fuzzy wuzzy was a bear", "wuzzy fuzzy was a bear")
    91
>>> fuzz.token_sort_ratio("fuzzy wuzzy was a bear", "wuzzy fuzzy was a bear")
    100
```

* **Token Set Ratio**

```python
>>> fuzz.token_sort_ratio("fuzzy was a bear", "fuzzy fuzzy was a bear")
    84
>>> fuzz.token_set_ratio("fuzzy was a bear", "fuzzy fuzzy was a bear")
    100
```

* **Process**

```python
>>> choices = ["Atlanta Falcons", "New York Jets", "New York Giants", "Dallas Cowboys"]
>>> process.extract("new york jets", choices, limit=2)
    [('New York Jets', 100), ('New York Giants', 78)]
>>> process.extractOne("cowboys", choices)
    ("Dallas Cowboys", 90)
```