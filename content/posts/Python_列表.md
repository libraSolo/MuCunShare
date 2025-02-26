---
date : 2021-08-03T16:21:57+08:00
draft : false
title : 'Python_列表'
author : '木村凉太'
categories : ['Python']
hiddenFromHomePage : true 
---


# Python_列表

**列表、元组、字典、集合，都是常用的容器类型**
**容器类型，就是用来在内存中保存数据的。**

# 1. 列表的创建与元素的访问

```python
# 方法一：直接赋值
namelist = []  # 定义一个空的列表
print(type(namelist))

namelist = ["11", "11", 11, True, "aa"]  # 存放多种数据类型，可重复
print(namelist)
print(namelist[1]) # 通过索引访问列表的每个元素

# 字符串和列表都属于序列的概念，所以很多方法都是类似的
# 列表的切片，可以参考字符串的相关操作
namelist = ["11", "11", 11, True, "aa"]
print(namelist[1:-1:2]) # [1,-1) 步长为2 可参考字符串

'''
执行结果：
<class 'list'>
['11', '11', 11, True, 'aa']
11
['11', True]
'''
```

```python
# 方法二：通过list()方法创建或强制类型转化为列表 str()  int()
a = list()  # 创建空的列表对象
print(a)

a = list("student") # 创建可迭代的列表对象
print(a)

a = list(range(10))
print(a)

'''
执行结果：
[]
['s', 't', 'u', 'd', 'e', 'n', 't']
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
'''
```

```python
# 方法三：通过列表推导式创建列表

a = [x*2 for x in range(5)]
print(a)

'''
执行结果：
[0, 2, 4, 6, 8]
'''
```

# 2. 常用操作 len()  max()   min()   sum()

## 2.1 列表的循环遍历

```python
namelist = ["11", 11, True, "aa"]

for name in namelist:
    print(name)

i = 0
while i <len(namelist):
    print(namelist[i])
    i +=1

'''
执行结果：
11
11
True
aa
11
11
True
aa
'''
```

## 2.2 max()和 min() 获取列表中最大的或者最小的元素

```python
list1 = [False, 2.0, 30, "a"]   # 字符串和数值无法比较
list1 = [1,2,3,4,5]
print(max(list1))
print(min(list1))   #相等比较取前者

'''
执行结果：
5
1
'''
```

## 2.3 sum() 对全部为数值的元素的列表求和

```python
list1 = [1,2,3,4,5]
print(sum(list1))

'''
执行结果：
15
'''
```

# 3.常用的数据操作

基本的数据操作：增删改查

## 3.1 增

```python
# append 追加
namelist = ["11", "11", 11, True, "aa"]
print("增加前的数据".center(30, "-"))
for name in namelist:
    print(name,end=",")
temp = input("\n追加数据为：")
namelist.append(temp)   # 在末尾追加一个元素

print("增加后的数据".center(30, "-"))
for name in namelist:
    print(name, end=",")

'''
执行结果：
------------增加前的数据------------
11,11,11,True,aa,
追加数据为：33
------------增加后的数据------------
11,11,11,True,aa,33,
'''
```

```python
# extend 扩展
a = [1, 2]
b = [3, 4]
a.append(b) # 将b列表当作一个元素增加到a列表中
print(a)

a.extend(b) # 将b列表中的每个元素拿出来增加到a列表中
print(a)

'''
执行结果：
[1, 2, [3, 4]]
[1, 2, [3, 4], 3, 4]
'''
```

```python
# + 操作
a = [1, 2]
b = [3, 4]
a = a + b   # 生成新的列表
print(a)    # 占用空间要大于extend

# * 操作
a = [1, 2]
b = a * 3
print(b)

# insert 插入
a = [1, 2]
a.insert(3, 1)
print(a)

'''
执行结果：
[1, 2, 3, 4]
[1, 2, 1, 2, 1, 2]
[1, 2, 1]
'''
```

## 3.2 删除 del pop remove

```python
namelist = ["11", "11", 11, True, "aa"]

del namelist[0]     # 在指定位置删除一个元素，没有返回值

name = namelist.pop()   # 弹出最后一个元素，有返回值（弹出元素）
print(name)

name = namelist.pop(-2)   # 弹出最后指定位置元素，有返回值（弹出元素）
print(name)

namelist.remove("11")   # 删除指定元素，如果匹配到多个，则删除第一个。 无匹配则报错
print(namelist)

'''
执行结果：
aa
11
[True]
'''
```

## 3.3 改

```python
namelist = ["11", "11", 11, True, "aa"]
print("原内容".center(30, "-"))
print(namelist)
print("修改内容".center(30, "-"))
namelist[0] = 22
print(namelist)

'''
执行结果：
-------------原内容--------------
['11', '11', 11, True, 'aa']
-------------修改内容-------------
[22, '11', 11, True, 'aa']
'''
```

## 3.4 查

```python
# in     not in
find = input("请输出你想找的内容:")
namelist = ["11", "11", 11, True, "aa"]
if find in namelist:
    print("在")
else:
    print("不在")

'''
执行结果：
请输出你想找的内容:aa
在
'''
```

```python
# index
namelist = ["11", "11", 11, True, "aa"]
print(namelist.index(11, 0, 3))  # 查[0,3)范围内指定元素, 返回对应下标.找不到会报错

# count
namelist = ["11", "11", 11, True, "aa"]
print(namelist.count(11))   # 统计某个元素出现的次数

'''
执行结果：
2
1
'''
```

```python
# 排序和反转
a = [1, 4, 2, 3]
print(a, id(a))

a.reverse() # 所有元素反转
print(a, id(a)) # a的内存地址没有改变

a.sort()    # 所有元素升序排序
print(a, id(a)) # a的内存地址没有改变

a.sort(reverse=True)    # 所有元素降序排序

'''
执行结果：
[1, 4, 2, 3] 1299297293384
[3, 2, 4, 1] 1299297293384
[1, 2, 3, 4] 1299297293384
'''
```

# 4. 列表的嵌套(二维,三位列表....)

```python
namelist = []
print(len(namelist))    # 0

namelist = [[], [], []]
print(len(namelist))    # 3
print(namelist[0])  # 取列表第一个元素

'''
执行结果：
0
3
[]
'''