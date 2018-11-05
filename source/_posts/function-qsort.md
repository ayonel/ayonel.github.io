---
title: 两句话的快排
url: 270.html
id: 270
tags:
  杂
date: 2017-03-10 17:58:52
---

&#160; &#160; &#160; &#160;现在博客正式从aws上迁移到[malash](http://malash.me)的服务器上。嘻嘻，以后再也不用花钱啦~~ 
&#160; &#160; &#160; &#160;然后malash跟我强行装了“用两句Haskell实现快排”的逼。确实感觉到函数式编程无限的想象力。 Haskell如何写，具体忘了，下面是Python的实现，原理类似。
```
def sort(arr):
    if len(arr) == 0:
        return []
    x = arr.pop(0)
    return sort(filter(lambda a : a < x ,arr)) + [x] + sort(filter( lambda a : a > x, arr))
```
&#160; &#160; &#160; &#160;**整个世界清净了，这确实是快排~~** 
下面是一个不怎么"函数式"的实现，代码还是很简单，并且很直观！
```
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) / 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quicksort(left) + middle + quicksort(right)
```