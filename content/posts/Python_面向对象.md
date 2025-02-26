---
date : 2021-08-12T16:21:57+08:00
draft : false
title : 'Python_面向对象'
author : '木村凉太'
categories : ['Python']
hiddenFromHomePage : true 
---

# Python_面向对象

# 👰面向对象编程

**面向过程：** 是以过程为中心的编程思想。
**面向对象：** 是对象及其之间的交互为中心的编程思想

![面向对象和面向过程](./images/2022/01/3131308690.png)

## 👩‍🦰类和对象

```python
class Car:  # 定义类: class 类名:

    energy = "电动"  # 属性,表示"特征"

    def move(self):  # 方法,表示"行为"
        print("在移动...")


c = Car()  # 对象实例化 : 对象名称 = 类名()
print("能源类型:", c.energy)  # 访问属性: 对象名称.属性
c.move()  # 调用方法: 对象名称.方法
```

**类：** 具有相同属性和方法的一类事物。(模具 / 图纸…)

**对象：** 也被称为 “实例”。将类中定义的特征具体化（赋值），就是一个对象或实例。

## 👧对象的属性

**对象的属性**

* 在类的外部，添加对象属性
* 在类的内部，添加对象属性

```python
class Dog:

    def bark(self):
        print("汪汪！")


c = Dog()   # 初始化一个对象，就分配一个新的内存空间
c.name = "cc"
d = Dog()
d.name = "dd"
print(c.name, end="")
c.bark()
print(d.name, end="")
d.bark()
```

## 🧒魔术方法

**定义：** 前后用两个下划线包裹，具有特殊功能的方法

魔术方法：__ init __ 方法

```python
class Dog:
    def __init__(self, name):     # 初始化对象时，默认被调用
        self.name = name    # 成员属性， 每个对象特有的属性(实例属性)
        print("小狗出生了")

c =Dog("cc")

# 类属性和成员属性
class Dog:

    def __init__(self, name):
        self.name = name
        self.legs = 4
        print(f"{self.name}出生了")

c = Dog("cc")
d = Dog("dd")
print("legs:", c.legs)
print("legs:", d.legs)