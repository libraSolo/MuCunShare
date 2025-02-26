+++
date = '2021-8-5T16:21:57+08:00'
draft = false
title = 'Python_字符串'
author = '木村凉太'
categories = 'Python'
hiddenFromHomePage = true 
+++

# Python_字符串

# 1. 字符串String

```python
# python中没有字符类型，一个字符也是字符串类型
# UTF-8编码 所有字符串都是unicode字符串
# 单引号 双引号 三引号

word = 'String'
sentence = "this is sentence"
paragraph = """ more
this is paragraph """
```

# 2. 转义字符

```python
my_str = "i'm a student"
my_str1 = 'i\'m a student'
my_str2 = r'i\'m a student'  # 转义不生效

print(my_str)
print(my_str1)
print(my_str2)

'''
执行结果：
i'm a student
i'm a student
i\'m a student
'''
```

# 3. 字符串切片（字符串的截取）

```python
str1 = "student"
# 正索引 0123456
# 负索引-7....-1
print(len(str1))    # 内置函数len() 获取长度
print(str1[2:6])    # 截取
print(str1[2:6:2])  # 左闭，右开，间隔步长
print(str1[::-1])   # 倒着打印

'''
执行结果：
14
ncun
nu
iatgnailnucnum
'''
```

# 4. 字符串的拼接

```python
str1 = "student"
str1 += "study"
print(str1)
print('*' * 30)

str2 = "tooooooooooooooooooo"\
       "long"\
    "write"

print(str2)

'''
执行结果：
studentstudy
******************************
tooooooooooooooooooolongwrite
'''
```

# 5. String 常用函数

## 5.1 字母转换

```python
# capitalize 首字母大写
str1 = "student"
res = str1.capitalize()
print(res)

# title 每个单词的首字母大写
str1 = "student and teacher "
res = str1.title()
print(res)
res = str1.istitle()  #判断是否每个单词首字母大写
print(res)

# upper 全大写
# str1.upper()
# lower 全小写
# str1.lower()
# swapcase 大小写互换
# str1.swapcase()

'''
执行结果：
Student
Student And Teacher 
False
'''
```

## 5.2 统计和查找

```python
# count 统计字符串中某个元素的数量
str1 = "Students"
res = str1.count("S") #区分大小写，可为字符串
print(res)

# find 查找某个字符第一次出现的位置
# find("字符串"，开始位置，结束位置) 结束位置取不到，取到之前的一个值。取不到为-1

# index 与 find 功能相同。find找不到返回-1，而index取不到会报错。

'''
执行结果：
1
'''
```

## 5.3 判断开头、结尾

```python
# startswith 判断是否以某个字符或字符串为开头
# startswith('字符串'，开始位置，结束位置) 返回为True或False

# endswith 判断是否以某个字符或字符串为结尾
# endswith('字符串'，开始位置，结束位置) 返回为True或False
```

## 5.4 分割与拼接

```python
# split 按某字符将字符串分割成 列表(默认从左到右按空格分隔)
# split("字符串", 切割个数)
str1 = "我_是_憨_憨"
res = str1.split("_", 1)
print(res)

# rsplit 从右往左排列  注意前面有r
# join 按某字符将列表拼接成字符串(容器类型都可)
list1 = ["st", "ud", "ent"]
res = "-".join(list1)
print(res)

'''
执行结果：
['我', '是_憨_憨']
st-ud-ent
'''
```

## 5.5去掉与替换

```python
# replace 替换字符串(第三个参数选择替换的次数)
str1 = "student"
res = str1.replace("stu","aaa")
print(res)
res = str1.replace("t", "a", 1)
print(res)

# strip 默认去掉首位两边的空白符
str1 = "\r student \t \n"
print(str1)
res = str1.strip()
print(res)

# lstrip() 只去掉左侧空白符
# rstrip() 只去掉右侧空白符
'''
执行结果：
aaadent
saudent
 student 	 

student
'''
```

## 5.6 判断类型

```python
# isspace() 字符串中只包含空白 返回True 否则返回False
res = "     "
print(res.isspace())

# isalpha() 字符串是否为纯字母
res = "abc1"
print(res.isalpha())
res = "abc"
print(res.isalpha())
res = "#a"
print(res.isalpha())

# isalnum() 字符串至少一个字符，所有字符都有字母或者数字
res = "a"
print(res.isalnum())
res = "1.1"
print(res.isalnum())
res = "1"
print(res.isalnum())

# isdecimal 检测字符串是否以数字组成，必须是纯数字
# isdigit() 字符串只包含数字则返回True，否则返回False
# isnumeric() 字符串只包含数字字符，返回True，否则返回False
# 差别在 汉字数字 罗马数字 byte数字

'''
执行结果：
True
False
True
False
True
False
True
'''
```

## 5.7 长度、填充

```python
str1 = "student"

# len()计算长度
print(len(str1))

# center 填充字符串，原字符居中(默认空格填充)
res = str1.center(10, "*")
print(res)

'''
执行结果：
7
*student**
'''
```

## 5.8 编码和解码

```python
# max 返回字符串编码最大的字母
# min 返回字符串编码最小的字母

# ord 内置函数，返回汉字或字母的Unicode编码
# chr 内置函数，给定Unicode编码 返回字符串

str1 = "学生"
str_utf8 = str1.encode("UTF-8")
str_gbk = str1.encode("GBK")
print(f"URF-8编码：{str_utf8}")
print(f"GBK编码：{str_gbk}")

print(f"URF-8解码：{str_utf8.decode('UTF-8')}")
print(f"GBK解码：{str_gbk.decode('GBK')}")

'''
执行结果：
URF-8编码：b'\xe5\xad\xa6\xe7\x94\x9f'
GBK编码：b'\xd1\xa7\xc9\xfa'
URF-8解码：学生
GBK解码：学生
'''