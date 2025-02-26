+++
date = '2024-8-24T16:21:57+08:00'
draft = false
title = 'Python_算法相关笔记'
author = '木村凉太'
categories = 'Python'
hiddenFromHomePage = true 
+++

# Python_语言特性笔记

来源于 stackoverflow 或 github 中

# 1. Python 函数参数传递

```py
num = 1
def func1(num):
    num = 2

func1(num)
print(num)  # num = 1
```

```python
list = []
def func2(list):
    list.append(1)

func2(list)
print(list)  # list = [1]
```

在Python中，变量是内存中一个对象的 '引用' ，进入到函数中的引用和外部引用无关。由内存值可以看出，内部id发生了改变，已经不是原来的数据了。

而在 List 当中，此对象是可变的对象，可以在其中修改它的值。
可变对象：list，dict，set
不可变：string，tuple，number

```python
num = 1   
print(id(num))  # id = 1776386160
def func1(num):
    print(id(num))  # id = 1776386160
    num = 2
    print(id(num))  # id = 1776386192

func1(num)
print(id(num),num)  # id = 1776386160
```

# 2. @staticmethod 和 @classmethod

Python 的三种方法：实例方法、静态方法、类方法

```python
class A():
    def func(self, num):
        print(f"实例方法:{self}", num)

    @staticmethod
    def staticmethod(num):
        print('静态方法', num)

    @classmethod
    def classmethod(cls, num):
        print(f'类方法:{cls}', num)

a = A()
a.staticmethod(1)   # 静态方法 1
A.staticmethod(1)   # 静态方法 1

a.classmethod(1)    # 类方法:<class '__main__.A'> 1
A.classmethod(1)    # 类方法:<class '__main__.A'> 1

a.func(1)   # 实例方法:<__main__.A object at 0x00000272151BA048> 1
A.func(1)   # 报错
```

静态方法都可以调用
类方法所传递的第一个参数都会改变为 此类 A
实例方法所传递的第一个参数是 实例 a

# 3. 类变量和实例变量

类变量在所有实例中间共享值
实例变量各自独立

# 4.下划线 _ 和 __

```python
__func__ : Python 内部名字
_func : 私有方法或属性，但是不是解释器所理解的，也可以和普通方法一样使用，单下划线是告诉程序，这个东西是私有的。
__func: 真正的私有，解释器所理解的，会被理解为 _class__func，若直接调用 __func 则会报错
```

# 5. 生成器和列表

```python
L = [x*x for x in range(10)]
print(L)    # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
t = (x*x for x in range(10))
print(t)    # <generator object <genexpr> at 0x00000225ADAD3308>
```

一个是列表，一个是生成器，所占用内部空间不同

# 6. *args 和 **kwargs

*args 和 **kwargs 称呼无所谓，可变。
*args：未知数量的变量
**kwargs：未知数量的字典形式变量

# 7. 鸭子类型

# 8.新式类和旧式类

```py
class A():
    def foo1(self):
        print "A"
class B(A):
    def foo2(self):
        pass
class C(A):
    def foo1(self):
        print "C"
class D(B, C):
    pass

d = D()
d.foo1()

# A
```

依次且深度优先
D --> B --> A
C 被 覆盖了。

# 9. 深浅拷贝

[我的csdn 深浅拷贝的文章](https://blog.csdn.net/LibraSolo/article/details/111189635)

```python
import copy

a = [1, 2, 3, [4, 5]]
b = copy.copy(a)
c = copy.deepcopy(a)
d = a

a.append(6)
a[3].append(7)
print(a)  # [1, 2, 3, [4, 5, 7], 6]
print(b)  # [1, 2, 3, [4, 5, 7]]
print(c)  # [1, 2, 3, [4, 5]]
print(d)  # [1, 2, 3, [4, 5, 7], 6]