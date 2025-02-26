# Python_字典

**dictionary 无序的对象集合 使用 (key-value) 储存，极快的查找速度**

**key 必须使用 *不可变类型* 且 *唯一性***

# 1. 字典的访问

```python
info = {}   # 定义一个空字典
info = {"name": "student", "age": 22}
print(info, type(info))
print(info["age"])

# print(info["sex"])  # 访问不存在的key，会报错：keyError
print(info.get("sex"))  # get方法返回不存在的值，默认返回：None

print(info.get("sex", "男"))     # get方法可以设定默认值，只在找不到时，生效

'''
执行结果：
{'name': 'student', 'age': 22} <class 'dict'>
22
None
男
'''
```

# 2. 使用列表创建字典 fromkeys

```python
namelist = ["black", "red", "white"]
dic1 = {}.fromkeys(namelist)    # 以列表中的元素为key, 创建字典 默认value为None
dic1 = {}.fromkeys(namelist, "默认值")     # 默认值可修改
print(dic1)

'''
执行结果：
{'black': '默认值', 'red': '默认值', 'white': '默认值'}
'''
```

# 3. 常用的数据操作

## 3.1 增

```python
info = {"name": "student", "age": 22}
newSex = input("请输入性别：")
info["sex"] = newSex
print(info)

'''
执行结果：
请输入性别：男
{'name': 'student', 'age': 22, 'sex': '男'}
'''
```

## 3.2 删

```python
info = {"name": "student", "age": 22}
del info["age"]     # 删除元素
print(info)

# info = {"name": "student", "age": 22}
# del info    # 删除字典
# print(info) # 不存在

info = {"name": "student", "age": 22}
info.clear()    # 清空
print(info)

# pop()
info = {"name": "student", "age": 22}
name1 = info.pop("name")    # 要指定key。 返回删除key对应的value
# info.pop("sex")   # 删除一个不存在的key, 会报错：keyError
# info.pop("sex", "男")  # 可以给定默认值
print(name1)
print(info)

# popitem()     # 删除最后一个键值对，返回的是：元组类型
info = {"name": "student", "age": 22}
res = info.popitem()
print(res)
res = info.popitem()
print(res)

'''
执行结果：
{'name': 'student'}
{}
student
{'age': 22}
('age', 22)
('name', 'student')
'''
```

## 3.3 改

```python
info = {"name": "student", "age": 22}
info["name"] = "black"
print(info)

info = {"name": "student", "age": 22}
info.update({"sex": "男", "age": 18})    # 指定的键存在，更新。指定的键不存在，新增
# info.update(sex="男",age=18)   # 另一种写法， 赋值
print(info)

'''
执行结果：
{'name': 'black', 'age': 22}
{'name': 'student', 'age': 18, 'sex': '男'}
'''
```

## 3.4 查

```python
info = {"name": "student", "age": 22}
# for i in info:  # 拿到的是键
#     print(i)

print(info.keys(), type(info.keys()))   # 得到所有的键(可迭代对象) 理解为：列表
print(info.values(), type(info.values()))   # 得到所有的值(可迭代对象) 理解为：列表
print(info.items(), type(info.items()))     # 得到所有的键和值(可迭代对象)   理解为：元组

# 遍历所有的键
for key in info.keys():
    print(key)
# 遍历所有的值
for value in info.values():
    print(value)
# 遍历所有的键和值
for key, value in info.items():
    print(f"key={key}, value={value}")

'''
执行结果：
dict_keys(['name', 'age']) <class 'dict_keys'>
dict_values(['student', 22]) <class 'dict_values'>
dict_items([('name', 'student'), ('age', 22)]) <class 'dict_items'>
name
age
student
22
key=name, value=student
key=age, value=22
'''
```

# 4. 扩展 使用函数，同时得到列表的下标和元素内容

```python
list1 = ["a", "b", "c", "d"]
# print(type(enumerate(mylist)))
for i, x in enumerate(list1):
    print(i+1, x)

'''
执行结果：
1 a
2 b
3 c
4 d
'''