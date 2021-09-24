---
layout: post
title: "python入门--条件表达式 方法"
description: "python control flow"
category: Tech
tags: [python]
---

###if表达式
```
>>> x = int(input("Please enter an integer: "))
Please enter an integer: 42
>>> if x < 0:
...     x = 0
...     print('Negative changed to zero')
... elif x == 0:
...     print('Zero')
... elif x == 1:
...     print('Single')
... else:
...     print('More')
...
More
```

###for表达式
```
>>> # Measure some strings:
... words = ['cat', 'window', 'defenestrate']
>>> for w in words:
...     print(w, len(w))
...
cat 3
window 6
defenestrate 12
```

```
>>> for w in words[:]:  # Loop over a slice copy of the entire list.
...     if len(w) > 6:
...         words.insert(0, w)
...
>>> words
['defenestrate', 'cat', 'window', 'defenestrate']
```

###range() 方法
```
>>> for i in range(5):
...     print(i)
...
0
1
2
3
4
```

```
>>> a = ['Mary', 'had', 'a', 'little', 'lamb']
>>> for i in range(len(a)):
...     print(i, a[i])
...
0 Mary
1 had
2 a
3 little
4 lamb
```

将range转化为list

```
>>> list(range(5))
[0, 1, 2, 3, 4]
```

###break continue 和 loop else
Python的for…else和while…else语法，这是Python中最不常用、最为误解的语法特性之一

Python中的for、while循环都有一个可选的else分支（类似if语句和try语句那样），在循环迭代正常完成之后执行。换句话说，如果我们不是以除正常方式以外的其他任意方式退出循环(break return)，那么else分支将被执行。也就是在循环体内没有break语句、没有return语句，或者没有异常出现。考虑一个简单的（无用的）例子：

```
>>> for i in range(5):
...     print(i)
... else:
...     print('Iterated over everything')
...
0
1
2
3
4
Iterated over everything
```

continue:

```
>>> for num in range(2, 10):
...     if num % 2 == 0:
...         print("Found an even number", num)
...         continue
...     print("Found a number", num)
Found an even number 2
Found a number 3
Found an even number 4
Found a number 5
Found an even number 6
Found a number 7
Found an even number 8
Found a number 9
```

###pass语句
他什么也不做，只是为了填充语法

```
>>> while True:
...     pass  # Busy-wait for keyboard interrupt (Ctrl+C)
...
```

```
>>> def initlog(*args):
...     pass   # Remember to implement this!
...
```

###定义方法
```
>>> def fib(n):    # write Fibonacci series up to n
...     """Print a Fibonacci series up to n."""
...     a, b = 0, 1
...     while a < n:
...         print(a, end=' ')
...         a, b = b, a+b
...     print()
...
>>> # Now call the function we just defined:
... fib(2000)
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597
```

```
>>> fib
<function fib at 10042ed0>
>>> f = fib
>>> f(100)
0 1 1 2 3 5 8 13 21 34 55 89

```

每个方法都有返回值，无论是否有return，默认返回值是`None`

```
>>> fib(0)
>>> print(fib(0))
None
```

带返回值的函数

```
>>> def fib2(n): # return Fibonacci series up to n
...     """Return a list containing the Fibonacci series up to n."""
...     result = []
...     a, b = 0, 1
...     while a < n:
...         result.append(a)    # see below
...         a, b = b, a+b
...     return result
...
>>> f100 = fib2(100)    # call it
>>> f100                # write the result
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
```

###定义方法
使用默认参数值

```
def ask_ok(prompt, retries=4, complaint='Yes or no, please!'):
    while True:
        ok = input(prompt)
        if ok in ('y', 'ye', 'yes'):
            return True
        if ok in ('n', 'no', 'nop', 'nope'):
            return False
        retries = retries - 1
        if retries < 0:
            raise OSError('uncooperative user')
        print(complaint)
```
This function can be called in several ways:

* giving only the mandatory argument: ask_ok('Do you really want to quit?')
* giving one of the optional arguments: ask_ok('OK to overwrite the file?', 2)
* or even giving all arguments: ask_ok('OK to overwrite the file?', 2, 'Come on, only yes or no!')


默认参数的有效值是函数声明的时候：

```
i = 5

def f(arg=i):
    print(arg)

i = 6
f()
```
will print `5`.


**Important warning:** 参数默认值只被分配一次。使用list, dictionary, instances of most classes. 都会改变参数默认值

```
def f(a, L=[]):
    L.append(a)
    return L
```

```
print(f(1))
print(f(2))
print(f(3))
```

输出：

```
[1]
[1, 2]
[1, 2, 3]
```

如果不想让参数默认值改变：

```
def f(a, L=None):
    if L is None:
        L = []
    L.append(a)
    return L
```

```
# 作用等同于 def funcvar(x): return x + 1
funcvar = lambda x: x + 1
>>> print funcvar(1)
2
 
# an_int 和 a_string 是可选参数，它们有默认值
# 如果调用 passing_example 时只指定一个参数，那么 an_int 缺省为 2 ，a_string 缺省为 A default string。如果调用 passing_example 时指定了前面两个参数，a_string 仍缺省为 A default string。
# a_list 是必备参数，因为它没有指定缺省值。
def passing_example(a_list, an_int=2, a_string="A default string"):
    a_list.append("A new item")
    an_int = 4
    return a_list, an_int, a_string
 
>>> my_list = [1, 2, 3]
>>> my_int = 10
>>> print passing_example(my_list, my_int)
([1, 2, 3, 'A new item'], 4, "A default string")
>>> my_list
[1, 2, 3, 'A new item']
>>> my_int
10
```