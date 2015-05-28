---
layout: post
title: "python入门--内置数据类型"
description: "python built-in data"
category: Tech
tags: [python]
---

##内置数据类型

###字典
像java中的hashtable

```
>>> d = {"server":"mpilgrim", "database":"master"}
>>> d
{'server': 'mpilgrim', 'database': 'master'}
>>> d["server"]
'mpilgrim'
>>> d["database"]
'master'
>>> d["mpilgrim"]
Traceback (innermost last):
File "<interactive input>", line 1, in ?
KeyError: mpilgrim
```

```
>>> d
{'server': 'mpilgrim', 'database': 'master'}
>>> d["database"] = "pubs"
>>> d
{'server': 'mpilgrim', 'database': 'pubs'}
>>> d["uid"] = "sa"
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'pubs'}
```

####字典Key大小写敏感

```
>>> d = {}
>>> d["key"] = "value"
>>> d["key"] = "other value"
>>> d
{'key': 'other value'}
>>> d["Key"] = "third value"
>>> d
{'Key': 'third value', 'key': 'other value'}
```

####字典中混合类型

```
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'pubs'}
>>> d["retrycount"] = 3
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'master', 'retrycount': 3}
>>> d[42] = "douglas"
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'master',
42: 'douglas', 'retrycount': 3}
```

####字典删除

```
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'master',
42: 'douglas', 'retrycount': 3}
>>> del d[42]
>>> d
{'server': 'mpilgrim', 'uid': 'sa', 'database': 'master', 'retrycount': 3}
>>> d.clear()
>>> d
{}
```

###列表

####定义

```
>>> li = ["a", "b", "mpilgrim", "z", "example"]
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li[0]
'a'
>>> li[4]
'example'
```

####使用负的索引：

```
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li[−1]
'example'
>>> li[−3]
'mpilgrim'
```

####截取list

```
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li[1:3]
['b', 'mpilgrim']
>>> li[1:−1]
['b', 'mpilgrim', 'z']
>>> li[0:3]
['a', 'b', 'mpilgrim']
```

```
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li[:3]
['a', 'b', 'mpilgrim']
>>> li[3:]
['z', 'example']
>>> li[:]
['a', 'b', 'mpilgrim', 'z', 'example']
```

####list添加数据

```
>>> li
['a', 'b', 'mpilgrim', 'z', 'example']
>>> li.append("new")
>>> li
['a', 'b', 'mpilgrim', 'z', 'example', 'new']
>>> li.insert(2, "new")
>>> li
['a', 'b', 'new', 'mpilgrim', 'z', 'example', 'new']
>>> li.extend(["two", "elements"])
>>> li
['a', 'b', 'new', 'mpilgrim', 'z', 'example', 'new', 'two', 'elements']
```

####extend 和 append的区别

```
>>> li = ['a', 'b', 'c']
>>> li.extend(['d', 'e', 'f'])
>>> li
['a', 'b', 'c', 'd', 'e', 'f']
>>> len(li)
6
>>> li[−1]
'f'
>>> li = ['a', 'b', 'c']
>>> li.append(['d', 'e', 'f'])
>>> li
['a', 'b', 'c', ['d', 'e', 'f']]
>>> len(li)
4
>>> li[−1]
['d', 'e', 'f']
```

####查询list

```
>>> li
['a', 'b', 'new', 'mpilgrim', 'z', 'example', 'new', 'two', 'elements']
>>> li.index("example")
5
>>> li.index("new")
2
>>> li.index("c")
Traceback (innermost last):
File "<interactive input>", line 1, in ?
ValueError: list.index(x): x not in list
>>> "c" in li
False
```

####删除list

```
>>> li
['a', 'b', 'new', 'mpilgrim', 'z', 'example', 'new', 'two', 'elements']
>>> li.remove("z")
>>> li
['a', 'b', 'new', 'mpilgrim', 'example', 'new', 'two', 'elements']
>>> li.remove("new")
Dive Into Python 20
>>> li
['a', 'b', 'mpilgrim', 'example', 'new', 'two', 'elements']
>>> li.remove("c")
Traceback (innermost last):
File "<interactive input>", line 1, in ?
ValueError: list.remove(x): x not in list
>>> li.pop()
'elements'
>>> li
['a', 'b', 'mpilgrim', 'example', 'new', 'two']
```

####使用list操作

```
>>> li = ['a', 'b', 'mpilgrim']
>>> li = li + ['example', 'new']
>>> li
['a', 'b', 'mpilgrim', 'example', 'new']
>>> li += ['two']
>>> li
['a', 'b', 'mpilgrim', 'example', 'new', 'two']
>>> li = [1, 2] * 3
>>> li
[1, 2, 1, 2, 1, 2]
```

###元组
元组是不可变list，当它创建后就不可变

####定义

```
>>> t = ("a", "b", "mpilgrim", "z", "example")
>>> t
('a', 'b', 'mpilgrim', 'z', 'example')
>>> t[0]
'a'
>>> t[−1]
'example'
>>> t[1:3]
('b', 'mpilgrim')
```

####元组没有方法

```
>>> t
('a', 'b', 'mpilgrim', 'z', 'example')
>>> t.append("new")
Traceback (innermost last):
File "<interactive input>", line 1, in ?
AttributeError: 'tuple' object has no attribute 'append'
>>> t.remove("z")
Traceback (innermost last):
File "<interactive input>", line 1, in ?
AttributeError: 'tuple' object has no attribute 'remove'
>>> t.index("example")
Traceback (innermost last):
File "<interactive input>", line 1, in ?
AttributeError: 'tuple' object has no attribute 'index'
>>> "z" in t
True
```

###变量声明
变量不需要明确定义，直接赋值使用，当出了作用域，会自动销毁

####定义一个myParams变量

```
if __name__ == "__main__":
    myParams = {"server":"mpilgrim", \
                "database":"master", \
                "uid":"sa", \
                "pwd":"secret" \
                }
```

定义多行变量时，需要使用`\`作为后续行标志

####引用不存在的变量

```
>>> x
Traceback (innermost last):
File "<interactive input>", line 1, in ?
NameError: There is no variable named 'x'
>>> x = 1
>>> x
1
```

You will thank Python for this one day.

####一次声明多个变量

```
>>> v = ('a', 'b', 'e')
>>> (x, y, z) = v
>>> x
'a'
>>> y
'b'
>>> z
'e'
```

####声明连续变量

```
>>> range(7)
[0, 1, 2, 3, 4, 5, 6]
>>> (MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY) = range(7)
>>> MONDAY
0
>>> TUESDAY
1
>>> SUNDAY
6
```

###格式化字符串

```
>>> k = "uid"
>>> v = "sa"
>>> "%s=%s" % (k, v)
'uid=sa'
```

####格式化字符串 vs 连接字符串

```
>>> uid = "sa"
>>> pwd = "secret"
>>> print pwd + " is not a good password for " + uid
secret is not a good password for sa
>>> print "%s is not a good password for %s" % (pwd, uid)
Dive Into Python 25
secret is not a good password for sa
>>> userCount = 6
>>> print "Users connected: %d" % (userCount, )
Users connected: 6
>>> print "Users connected: " + userCount
Traceback (innermost last):
File "<interactive input>", line 1, in ?
TypeError: cannot concatenate 'str' and 'int' objects
```

####格式化数值

```
>>> print "Today's stock price: %f" % 50.4625
50.462500
>>> print "Today's stock price: %.2f" % 50.4625
50.46
>>> print "Change since yesterday: %+.2f" % 1.5
+1.50
```

###映射list

####介绍list解析

```
>>> li = [1, 9, 8, 4]
>>> [elem*2 for elem in li]
[2, 18, 16, 8]
>>> li
[1, 9, 8, 4]
>>> li = [elem*2 for elem in li]
>>> li
[2, 18, 16, 8]
```

####keys,values,和items函数

```
>>> params = {"server":"mpilgrim", "database":"master", "uid":"sa", "pwd":"secret"}
>>> params.keys()
['server', 'uid', 'database', 'pwd']
>>> params.values()
['mpilgrim', 'sa', 'master', 'secret']
>>> params.items()
[('server', 'mpilgrim'), ('uid', 'sa'), ('database', 'master'), ('pwd', 'secret')]
```

####list解析
```
>>> params = {"server":"mpilgrim", "database":"master", "uid":"sa", "pwd":"secret"}
>>> params.items()
[('server', 'mpilgrim'), ('uid', 'sa'), ('database', 'master'), ('pwd', 'secret')]
>>> [k for k, v in params.items()]
['server', 'uid', 'database', 'pwd']
>>> [v for k, v in params.items()]
['mpilgrim', 'sa', 'master', 'secret']
>>> ["%s=%s" % (k, v) for k, v in params.items()]
['server=mpilgrim', 'uid=sa', 'database=master', 'pwd=secret']
```

###连接list与分割字符串

####join,不能join非字符串

```
>>> params = {"server":"mpilgrim", "database":"master", "uid":"sa", "pwd":"secret"}
>>> ["%s=%s" % (k, v) for k, v in params.items()]
['server=mpilgrim', 'uid=sa', 'database=master', 'pwd=secret']
>>> ";".join(["%s=%s" % (k, v) for k, v in params.items()])
'server=mpilgrim;uid=sa;database=master;pwd=secret'
```

####分割字符串

```
>>> li = ['server=mpilgrim', 'uid=sa', 'database=master', 'pwd=secret']
>>> s = ";".join(li)
>>> s
'server=mpilgrim;uid=sa;database=master;pwd=secret'
>>> s.split(";")
['server=mpilgrim', 'uid=sa', 'database=master', 'pwd=secret']
>>> s.split(";", 1)
['server=mpilgrim', 'uid=sa;database=master;pwd=secret']
```