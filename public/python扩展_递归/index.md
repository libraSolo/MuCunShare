# Python扩展_递归


# Python扩展_递归

# ⭐递归

**形式：** 自己调用自己的函数

**递：** 递进

**归：** 返回

## 🚩递归的写法

1. 编写函数（功能）
2. 确定**结束条件**和**返回值**
3. 调用函数自身，修改参数

```python
def printNum(n):
    print(n)
    if n == 1:
        return
    printNum(n - 1)     # 调用自身的函数

printNum(100)

'''
执行结果：
100
99
...
2
1
'''
```

## ❤经典例子

### 🧡求指定位数的阶乘

阶乘：n！ = n * (n - 1 )!

分析：

f(1) = 1    			n = 1

f(n) = f(n-1) * n    		n>1

```python
def function(a):
    if a == 1:
        return 1
    return function(a - 1) * a

print(function(3))

'''
执行结果：
6
'''
```

### 💛斐波那契数列

斐波那契数列：0，1，1，2，3，5，8，13，21，34，55，89，144...

分析：

f(n) = n-1   				n<=2

f(n) = f(n-1) + f(n-2) 		n>2

```python
def function2(n):
    if n <= 2:
        return n - 1
    return function2(n-1) + function2(n-2)

print(function2(10))

'''
执行结果：
34
'''
