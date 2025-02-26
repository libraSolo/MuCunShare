---
date : 2021-08-13T16:21:57+08:00
draft : false
title : 'Python扩展_变量'
author : '木村凉太'
categories : ['Python']
hiddenFromHomePage : true 
---

# Python扩展_变量

# 🥪变量

## 🧀局部变量

**定义：** 函数内部定义的变量

```python
def test1():
    a = 100
    print(a)  # 局部变量，仅能在 test1 中使用


def test2():
    a = 200
    print(a)  # a 互不影响

test1()
test2()
print(a)  # 报错，只能在所定义的函数内部使用

'''
执行结果：
100
200
NameError: name 'a' is not defined
'''
```

## 🌮全局变量

**定义：** 在文件内有效，能在多个函数内部调用

```python
a = 100

def test3():
    print(a)

def test4():
    print(a)  # test3 和 test4都可以调用a

test3()
test4()

'''
执行结果：
100
100
'''
```

## 🍖全局变量和局部变量

**🍖全局变量和局部变量名字相同时，优先使用最近定义的变量。**
**有局部变量时，优先使用局部变量；没有局部变量时，看是否有全局变量。**

```python
a = 100

def test5():
    a = 50
    print(a)

test5()

'''
执行结果：
50
'''
```

🍖**在函数中修改全局变量前，需要在函数中声明变量为 global**
**被修改的全局变量，在其他函数中访问时数据也会跟着改变。**

```python
a = 100

def test1():
    global a  # 在函数中声明全局变量的关键字：global
    a = 200
    print("test1.....", a)

def test2():
    print("test2中的全局变量：", a)

print("全局的：", a)
test1()
print("test1执行后全局的：", a)
test2()

'''
执行结果：
全局的： 100
test1..... 200
test1执行后全局的： 200
test2中的全局变量： 200
'''
```

**🍖可以通过 local() 和 global() 打印局部和全局变量**

```python
a = 100

def test1():
    a = 200
    global b
    b = 500
    print(locals())  # 显示模块（文件）内的局部变量，即为“全局变量”
    print(globals())  # 不论写在哪里，始终显示的都是全局变量

test1()
print(b)

'''
执行结果：
{'a': 200}
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x0000021F204FB550>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, '__file__': 'F:/py/test.py', '__cached__': None, 'a': 100, 'test1': <function test1 at 0x0000021F204A3EA0>, 'b': 500}
500
'''
```

## 🥩形式参数和局部变量

```python
def test(a, b):
    print(f"局部变量a={a}, b={b}")
    a = 10
    b.append("bbb")


x = 20
y = ["111"]
print(f"调用前{x},{y}")
test(x, y)
print(f"调用后{x},{y}")

'''
执行结果：
调用前20,['111']
局部变量a=20, b=['111']
调用后20,['111', 'bbb']
'''
```

对于**不可变参数**，在函数内，每次都是让局部变量指向新的地址值，所以 a = 10，是局部变量 a 指向了新数值的新地址，不影响外部的 x 。

对于**可变参数**，在函数内，是传递进来的（原有的）地址值上修改数据内容，所以 b.append("bbb"),是修改了（和 y 相同的）地址空间的列表内容，所以外部参数内容会受影响。