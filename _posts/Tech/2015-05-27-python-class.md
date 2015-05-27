---
layout: post
title: "python入门--类 异常"
description: "python class exception"
category: Tech
tags: [jekyll]
---

###类
Python支持有限的多继承形式。私有变量和方法可以通过添加至少两个前导下划线和最多尾随一个下划线的形式进行声明（如“__spam”，这只是惯例，而不是Python的强制要求）。当然，我们也可以给类的实例取任意名称。例如：

```
class MyClass(object):
    common = 10
    def __init__(self):
        self.myvariable = 3
    def myfunction(self, arg1, arg2):
        return self.myvariable
 
    # This is the class instantiation
>>> classinstance = MyClass()
>>> classinstance.myfunction(1, 2)
3
# This variable is shared by all classes.
>>> classinstance2 = MyClass()
>>> classinstance.common
10
>>> classinstance2.common
10
# Note how we use the class name
# instead of the instance.
>>> MyClass.common = 30
>>> classinstance.common
30
>>> classinstance2.common
30
# This will not update the variable on the class,
# instead it will bind a new object to the old
# variable name.
>>> classinstance.common = 10
>>> classinstance.common
10
>>> classinstance2.common
30
>>> MyClass.common = 50
# This has not changed, because "common" is
# now an instance variable.
>>> classinstance.common
10
>>> classinstance2.common
30
 
# This class inherits from MyClass. The example
# class above inherits from "object", which makes
# it what's called a "new-style class".
# Multiple inheritance is declared as:
# class OtherClass(MyClass1, MyClass2, MyClassN)
class OtherClass(MyClass):
    # The "self" argument is passed automatically
    # and refers to the class instance, so you can set
    # instance variables as above, but from inside the class.
    def __init__(self, arg1):
        self.myvariable = 3
        print arg1
 
>>> classinstance = OtherClass("hello")
hello
>>> classinstance.myfunction(1, 2)
3
# This class doesn't have a .test member, but
# we can add one to the instance anyway. Note
# that this will only be a member of classinstance.
>>> classinstance.test = 10
>>> classinstance.test
10
```

###异常
Python中的异常由 try-except [exceptionname] 块处理，例如：

```
def some_function():
    try:
        # Division by zero raises an exception
        10 / 0
    except ZeroDivisionError:
        print "Oops, invalid."
    else:
        # Exception didn't occur, we're good.
        pass
    finally:
        # This is executed after the code block is run
        # and all exceptions have been handled, even
        # if a new exception is raised while handling.
        print "We're done with that."
 
>>> some_function()
Oops, invalid.
We're done with that.
```

###导入
外部库可以使用 import [libname] 关键字来导入。同时，你还可以用 from [libname] import [funcname] 来导入所需要的函数。例如：

```
import random
from time import clock
 
randomint = random.randint(1, 100)
>>> print randomint
64
```

###文件I / O
Python针对文件的处理有很多内建的函数库可以调用。例如，这里演示了如何序列化文件(使用pickle库将数据结构转换为字符串)：

```
spath="F:/baa.txt"
# Opens file for writing.
#Creates this file doesn't exist.
f=open(spath,"w") 
f.write("First line 1.\n")
f.writelines("First line 2.")
f.close()
f=open(spath,"r") # Opens file for reading
for line in f:
    print (line)
f.close()
```

###其它杂项

* 数值判断可以链接使用，例如 1<a<3 能够判断变量 a 是否在1和3之间。
* 可以使用 del 删除变量或删除数组中的元素。
* 列表推导式(List Comprehension)提供了一个创建和操作列表的有力工具。列表推导式由一个表达式以及紧跟着这个表达式的for语句构成，for语句还可以跟0个或多个if或for语句，来看下面的例子：

```
>>> lst1 = [1, 2, 3]
>>> lst2 = [3, 4, 5]
>>> print [x * y for x in lst1 for y in lst2]
[3, 4, 5, 6, 8, 10, 9, 12, 15]
>>> print [x for x in lst1 if 4 > x > 1]
[2, 3]
# Check if an item has a specific property.
# "any" returns true if any item in the list is true.
>>> any([i % 3 for i in [3, 3, 4, 4, 3]])
True
# This is because 4 % 3 = 1, and 1 is true, so any()
# returns True.
 
# Check how many items have this property.
>>> sum(1 for i in [3, 3, 4, 4, 3] if i == 4)
2
>>> del lst1[0]
>>> print lst1
[2, 3]
>>> del lst1
```


全局变量在函数之外声明，并且可以不需要任何特殊的声明即能读取，但如果你想要修改全局变量的值，就必须在函数开始之处用global关键字进行声明，否则Python会将此变量按照新的局部变量处理（请注意，如果不注意很容易被坑）。例如：

```
number = 5
 
def myfunc():
    # This will print 5.
    print number
 
def anotherfunc():
    # This raises an exception because the variable has not
    # been bound before printing. Python knows that it an
    # object will be bound to it later and creates a new, local
    # object instead of accessing the global one.
    # 在python3中调用此方法会报错
    print number
    number = 3
 
def yetanotherfunc():
    global number
    # This will correctly change the global.
    number = 3
```