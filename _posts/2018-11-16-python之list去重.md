---
layout:     post
title:      python列表去重
subtitle:   列表去重
date:       2018-11-16
author:     永泉狂客
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - python
    - list
---

list1 = [1,2,5,2,5,7,3,4,7,34,9]

## 1.while遍历

```
def delRepeat(list1):
   for x in list1:
       while list1.count(x)>1:
            del list1[list1.index(x)]
   return list1
```
## 2.for遍历

```
news_list1 = []
for id in list1:
    if id not in news_list1:
        news_list1.append(id)
print (news_list1)
```
## 3.利用set（）特性

```
new_list1 = set(list1)
```
## 4.lambda+reduce

```
func = lambda x,y:x if y in x else x + [y]
a = reduce(func, [[], ] + list1)
```
