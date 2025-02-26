# Python_函数进阶



# Python_函数进阶

# 1. 函数的定义

```python
def func():  # 见名知意
    pass  # 占位

# 函数的调用
func()

print(id(func()), type(func))

'''
执行结果：
1879014704 <class 'function'>
'''
```

## 1.1 函数名也可以作为变量进行赋值\传递\储存

```python
def func1():
    print("func1")


def func2():
    print("func2")


res = func1

print(id(res), id(func1))

res()

myfuncs = [func1, func2]
for i in myfuncs:
    i()

'''
执行结果：
2087062875808 2087062875808
func1
func1
func2
'''
```

# 2. 参数

## 2.1 【位置参数】普通参数

```python
def printInfo(a, b):
    print(a, b)


printInfo(10, 20)

'''
执行结果：
10 20
'''
```

`printInfo(10, 20, 30) # 使用位置参数时，数量要一致，否则报错`

## 2.2 【默认参数】有默认值

**形式参数** 在 **默认参数** 前

```python
def printInfo(name, age=18):
    print(f"name:{name}, age:{age}")


printInfo("张三", 20)
printInfo("李四")  # 没有给定实际参数时，会给定默认值

'''
执行结果：
name:张三, age:20
name:李四, age:18
'''
```

## 2.2 【指定参数名复制】 函数调用时，根据参数名，进行针对性赋值

```python
def printInfo(name="张三", age=18):
    print(f"name:{name}, age:{age}")


printInfo(age=20, name="李四")  # 指定形参变量名， 可以不按照位置顺序传递参数
printInfo("李四", age=20)

'''
执行结果：
name:李四, age:20
name:李四, age:20
'''

printInfo(age = 20, "李四") # 如果混用， 位置实参 在 关键字实参 前面

```

## 2.3 可变参数

***args** : arguments 可变参数

****kwargs*👎 : keyword arguments 关键字参数

`* 和 ** 将多个变量进行"打包"，变为元组或字典`

```python
def printInfo(name, age, *args):
    print(f"name:{name}, age:{age}")
    print(args, type(args))


printInfo("张三", 18, 22, "憨憨")


def printInfo(name, age, **kwargs):
    print(f"name:{name}, age:{age}")
    print(kwargs, type(kwargs))


printInfo("张三", 18, city="Beijing", sex="man")

'''
执行结果：
name:张三, age:18
(22, '憨憨') <class 'tuple'>
name:张三, age:18
{'city': 'Beijing', 'sex': 'man'} <class 'dict'>
'''
```

`在'函数调用'时，*和**的作用是：'解包'`

`符号 * 将传进来的字符串、元组、列表、集合转化为多个标准参数`

`符号 ** 将传进来的字典，转化为多个关键字参数`

```python
def printInfo(name, age, *args):
    print(f"name:{name}, age:{age}")
    print(args, type(args))


rate = [8, 9, 10]
rate = (8, 9, 10)
rate = {8, 9, 10}  # 无序
printInfo("张三", 18, *rate)


def printInfo(name, age, **kwargs):
    print(f"name:{name}, age:{age}")
    print(kwargs, type(kwargs))


info = {"city": "Beijing", "sex": "man"}
printInfo("张三", 18, **info)

'''
执行结果：
name:张三, age:18
(8, 9, 10) <class 'tuple'>
name:张三, age:18
{'city': 'Beijing', 'sex': 'man'} <class 'dict'>
'''
```

## 2.4 函数的调用顺序

**函数调用时，参数传递的顺序:**

**调用时---> 实际参数，位置参数，关键字参数**

`顺序： args , *args, **kwargs`

```python
def testArgs(a, b, c=10, d=20, *args, **kwargs):
    print(f"a={a},b={b},c={c},d={d},args={args},kwargs={kwargs}")


testArgs(1, 2, 3, 4, 5, 6, 7, x=100, y=200)


def testArgs(*args, a, b, c=10, d=20, **kwargs):  # 可变参数在最前面时，匹配到关键字赋值的其他实际参数
    print(f"a={a},b={b},c={c},d={d},args={args},kwargs={kwargs}")


testArgs(1, 2, 3, 4, 5, a=6, c=100, b=7, x=100, y=200)

'''
执行结果：
a=1,b=2,c=3,d=4,args=(5, 6, 7),kwargs={'x': 100, 'y': 200}
a=6,b=7,c=100,d=20,args=(1, 2, 3, 4, 5),kwargs={'x': 100, 'y': 200}
'''
```

## 2.5 命名关键字参数

***后面的参数，被称为‘命名关键字参数’**

```python
def testStar(a, b, c, *, name, age):
    print(f"a={a},b={b},c={c},name={name},age={age}")


testStar(1, 2, 3, name="张三", age=30)  # *的作用，表示*后面的参数必须使用命名的方式来进行参数赋值

'''
执行结果：
a=1,b=2,c=3,name=张三,age=30
'''
```

# 3 返回值

## 3.1 单返回值

```python
def test():
    pass


# return 可返回所有类型
res = test()
print(res, type(res))

'''
执行结果:
None <class 'NoneType'>
'''
```

## 3.2 多个返回值

**将多个返回值，封装在列表，字典，元组容器中。(set会改变顺序)**

```python
def divid(a, b):
    return [a, b]


def divid2(a, b, c):
    return a, b, b  # 返回元组


res = divid2(1, 2, 3)
print(res)

'''
执行结果：
(1, 2, 2)
'''
