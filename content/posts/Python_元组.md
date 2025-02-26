+++
date = '2021-8-1T16:21:57+08:00'
draft = false
title = 'Python_元组'
author = '木村凉太'
categories = 'Python'
hiddenFromHomePage = true 
+++

# Python_元组

**Tuple(元组) 内容不可变，但是包含可变对象**
**相像列表list**

# 1. 创建元组

```python
tup1 = ()  # 创建空元组
print(tup1, type(tup1))

'''
执行结果：
() <class 'tuple'>
'''

tup1 = (50)
tup2 = (50,)  # 元组的必备条件为 , 单元素最后,不可省。 多元素可省略
print(tup1, type(tup1))
print(tup2, type(tup2))

'''
执行结果：
50 <class 'int'>
(50,) <class 'tuple'>
'''

tup1 = 50, 60, 70
tup2 = 50,
print(tup1, type(tup1))
print(tup2, type(tup2))  # 创建元组可以不加() 不推荐

'''
执行结果：
(50, 60, 70) <class 'tuple'>
(50,) <class 'tuple'>
'''
```

# 2. 常用的数据操作

## 2.1 增(连接)

```python
tup1 = (1, 2, 3)
tup2 = (4, 5, 6)
tup = tup1 + tup2
print(tup, type(tup))

'''
执行结果：
(1, 2, 3, 4, 5, 6) <class 'tuple'>
'''
```

## 2.2 删(删除元组对象，不可删内部元素)

```python
tup1 = (1, 2, 3)
print(tup1)
del tup1
print(tup1) # 报错，因为已经删除
```

## 2.3 改 (只能更改内部可变变量)

```python
list1 = [2, 3]
tup1 = (1, list1)
print(tup1)
list1.append(4)
print(tup1)

'''
执行结果：
(1, [2, 3])
(1, [2, 3, 4])
'''
```

## 2.4 查

```python
tup1 = (1, 2, 3)
print(tup1[0])  # 根据下标访问
print(tup1[::-1])   # 切片

'''
执行结果：
1
(3, 2, 1)
'''
```

# 3. 常用操作

## 3.1 循环

```python
tup1 = (1, 2, 3)
for i in tup1:
    print(i)

'''
执行结果：
1
2
3
'''
```

## 3.2 in    not in

```python
tup1 = (1, 2, 3)
if 3 in tup1:
    print("true")
else:
    print("false")

'''
执行结果:
true
'''
```

## 3.3 计数

**计数 count 最大最小max min 求和 sum 长度 len**

**类比List**

## 3.4 强制类型转换 tuple()

```python
# 字符串
s = "student"
tup = tuple(s)
print(tup, type(tup))

# 列表
list1 = [1, 2, 3]
tup = tuple(list1)
print(tup, type(tup))

# 生成器对象 (后期详细补充)
s = (i*2 for i in range(5))
print(tuple(s))

'''
执行结果：
('s', 't', 'u', 'd', 'e', 'n', 't') <class 'tuple'>
(1, 2, 3) <class 'tuple'>
(0, 2, 4, 6, 8)
'''
```

# 4. 补充

```python
list1 = [2, 3]
tup1 = (1, list1)
print(tup1)
list1 = [2, 3, 4]
print(tup1)

'''
执行结果：
(1, [2, 3])
(1, [2, 3])
'''
```

**列表更改，但是元组还是[1, [2, 3]]。
tup1保存了list1的地址，第二次访问仍然可以通过地址访问到之前的列表内容。list1第二次赋值，仅仅改变了list1自身指向的内存空间，没有改变tup1已经保存的list1之前的地址**