+++
date = '2021-8-11T16:21:57+08:00'
draft = false
title = 'Python_集合'
author = '木村凉太'
categories = 'Python'
hiddenFromHomePage = true 
+++

# Python_集合

set和dict类似，也是一组key的集合，但没有value。**key唯一且无序**

# 1. 集合的创建

```python
s1 = set()  # 空集合 只能用set()
print(s1, type(s1))

s1 = {1, 2, 3}
print(s1, type(s1))

'''
执行结果：
set() <class 'set'>
{1, 2, 3} <class 'set'>
'''
```

# 2. 集合的性质

## 2.1 唯一、无序

```python
# 集合：无序
s1 = {1, 2, 3}
s2 = {3, 2, 1}
print(s1, type(s1))  # {1, 2, 3}
print(s2, type(s2))  # {1, 2, 3}

# 集合：唯一
s1 = {1, 1, 2, 3, 3}
print(s1)

'''
执行结果：
{1, 2, 3} <class 'set'>
{1, 2, 3} <class 'set'>
{1, 2, 3}
'''
```

## 2.2 集合存储各种数据类型

```python
# 集合：存放不可变(key)
s1 = {1, 2, 3, False, "4"}   # 字典存放多种数据类型
print(s1)
# list1 = [1, 2, 3]     # 列表可变 存放报错
# s2 = {1, list1}
# print(s2)

# dic1 = {1: 2}     # 字典可变 存放报错
# s3 = {1, dic1}
# print(s3)

tuple1 = (1, 2, 3)
s3 = {1, tuple1}
print(s3)

'''
执行结果：
{False, 1, 2, 3, '4'}
{1, (1, 2, 3)}
'''
```

# 3. 常用的数据操作

## 3.1 增

```python
s1 = {1, 2, 3}
print(s1, id(s1))
s1.add(4)
print(s1, id(s1))

# 指定序列，依次加入到集合
s1 = {1, 2, 3}
s1.update("abc")
print(s1, id(s1))

s1 = {1, 2, 3}
list1 = ["a", "b", "c"]
s1.update(list1)
print(s1, id(s1))

'''
执行结果：
{1, 2, 3} 1833057707624
{1, 2, 3, 4} 1833057707624
{1, 2, 3, 'c', 'a', 'b'} 1833060551816
{1, 2, 3, 'c', 'a', 'b'} 1833057707624
'''
```

## 3.2 删

**删 clear    remove  discard pop**

```python
s1 = {1, 2, 3}
s1.clear()  # 清空
print(s1)

s1 = {1, 2, 3}
s1.remove(1)    # 移除元素。移除不存在的元素，报错：keyError。不可给定默认值
# s1.discard(4) # 如果没有找到，不做任何操作
print(s1)

s1 = {1, 2, 3}
a = s1.pop()     # 随机弹出一个元素，每次都不一定相同
print(a)
print(s1)

'''
执行结果：
set()
{2, 3}
1
{2, 3}
'''
```

## 3.3 改

**改：先删除后增加**

```python
s1 = {1, 2, 3}
s1.remove(1)
s1.add(4)
print(s1)

'''
执行结果：
{2, 3, 4}
'''
```

## 3.4 查

```python
# 遍历
s1 = {1, 2, 3}
for i in s1:
    print(i)

# 包含 in     not in
s1 = {1, 2, 3}
res = 1 not in s1
print(res)

'''
执行结果：
1
2
3
False
'''
```

# 4. 集合的运算

## 4.1 交、并、差、补

```python
s1 = {1, 2, 3, 4}
s2 = {3, 4, 5, 6}

# 集合的运算交 & 或者 intersection
result = s1 & s2
# result = s1.intersection(s2)
print(result)

# 集合的运算并 | 或者 union
result = s1 | s2
# result = s1.union(s2)
print(result)

# 集合的运算差 - 或者 difference
result = s1 - s2
# result = s1.difference(s2)
print(result)

# 集合的运算异或集(对称差集)    ^ 或者 symmetric_difference
result = s1 ^ s2
# result = s1.symmetric_difference(s2)
print(result)

'''
执行结果：
{3, 4}
{1, 2, 3, 4, 5, 6}
{1, 2}
{1, 2, 5, 6}
'''
```

## 4.2 子集

```python
a = {1, 2}
b = {1, 2, 3, 4}
result = a <= b
# result= a.issubset(b)     同a<=b
# result= b.issuperset(a)   同a<=b
print(result)

# 不相交 isdisjoint
s1 = {1, 2, 3, 4}
s2 = {3, 4, 5, 6}
result = s1.isdisjoint(s2)
print(result)

'''
执行结果：
True
False
'''
```

# 5. 应用

## 5.1 列表应用

```python
num = [1, 1, 2, 3, 3]
s = set(num)    # 注意：顺序可能打乱
res = list(s)
print(res)

'''
执行结果：
[1, 2, 3]
'''
```

## 5.2 冰冻集合 frozenset

**强制转化为冰冻集合**
**一旦创建，不能进行任何修改，只能用于集合运算操作**

```python
s = frozenset()
print(s, type(s))

s1 = [1, 2, 3]
s1 = "abc"
s2 = frozenset(s1)
print(s2, type(s2))

# s2.add(4)     # 报错

s3 = frozenset("4")
print(s2 & s3)

'''
执行结果：
frozenset() <class 'frozenset'>
frozenset({'b', 'c', 'a'}) <class 'frozenset'>
frozenset()
'''