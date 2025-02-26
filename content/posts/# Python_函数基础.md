# Python_函数基础
# 1. 定义一个函数

```python
def printInfo():
    print("-" * 20)
    print("人生苦短，我学Python")
    print("-" * 20)
```

## 1.1 函数的调用

```python
printInfo()

'''
执行结果：
--------------------
人生苦短，我学Python
--------------------
'''
```

## 1.2 带参数的函数

**a, b 为‘形参’**

```python
def add2Num(a, b):
    c = a + b
    print(c)
```

**调用时，给定的数值为实参**

```python
add2Num(10, 20)

'''
执行结果：
30
'''
```

## 1.3 带返回值的函数

```python
def add2Num2(a, b):
    return a + b

result = add2Num2(100, 30)
print(result)

'''
执行结果：
130
'''
```

**return "返回"了结果, return结束了函数的执行**

```python
def subtraction(a, b):
    if a < b:
        return b - a
        print("return 没有结束")
    else:
        return a - b
        print("return 没有结束")
    print("return 没有结束")

print(subtraction(3, 5))

'''
执行结果：
2
'''
```

# 2. 函数(调用)的嵌套

```python
def add2Num3(a, b):
    return a + b


def add3num(a, b, c):
    return add2Num3(add2Num3(a, b), c)


print(add3num(1, 2, 3))

'''
执行结果：
6
'''