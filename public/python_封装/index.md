# Python_封装


# Python_封装

# 🚗封装

**定义：** 对外部隐藏信息，不能随意访问 / 修改对象的数据、方法。通过限制类的属性和方法的访问方式，实现 “封装”。

## 🚓封装的层次

**封装分为三个层次**

* **类的封装：** 外部可以任意访问/修改类中的属性和方法。
* **私有属性：** 外部不可以访问/修改类的属性或方法。
* **公有方法+私有属性：** 外部有条件限制的访问/修改属性，调用方法。

## 🚕封装的表现

**类的定义：** 将某些特定属性和方法进行“隔离”。

（每个学生有自己的年龄，外部可以通过对象任意读取或修改）

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age


s1 = Student("李白", 18)
s2 = Student("杜甫", 20)

print(s1.age)
s1.age = 200    # 可随意更改
print(s1.age)

'''
执行结果:
18
200
'''
```

**属性私有：** 两个下划线开头的属性就是私有属性。私有属性只能在类的内部使用，外部不能使用。

（不让外部读取/修改学生的年龄）

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.__age = age    # 使用2个下划线开头的属性，定义为私有属性

s1 = Student("李白", 18)
# print(s1.__age)     # 外界无法直接通过属性名来访问

# __dict__
print(s1.__dict__)      #  查看s1的所有属性
print(s1._Student__age) # 可通过_Student__age 访问私有属性

'''
执行结果：
{'name': '李白', '_Student__age': 18}
18
'''
```

**私有属性+共有方法：** 可以实现“有限制条件的开放给外部”。

（可以读取年龄，但是不能随意修改年龄）

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.__age = age

    def getAge(self):
        return self.__age

    def setAge(self, age):
        if 0 < age <=130:
            self.__age = age
        else:
            print("不能随意修改")

s1 = Student("李白", 18)
print(s1.getAge())
s1.setAge(200)
print(s1.getAge())

'''
执行结果：
18
不能随意修改
18
'''
```

## 🛺封装的简化(常用)写法

**装饰器的概念：** 以 @ 开头，调用另一个函数（或方法），扩展对“所修饰方法”的功能。

@property 装饰器修饰的属性用来返回属性值；还可以继续进行 setter 的设置，用来设置属性值。

```python
class Dog:

    @property
    def bark(self):
        print("汪汪")
        return "return值"

d = Dog()
s = d.bark
print(s)

'''
执行结果：
汪汪
return值
'''
```

简化 Student

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.__age = age

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, age):
        if 0 < age <=130:
            self.__age = age
        else:
            print("不能随意修改")

s1 = Student("李白", 18)
print(s1.age)
s1.age = 20
print(s1.age)

'''
执行结果：
18
20
'''
```

# 🚎总结

* 使用 @property 装饰器时，方法名不必与属性名相同。
* 可以更好地防止外部通过猜测里有属性名访问。
* 凡是赋值语句，就会触发 set 方法。获取属性值，则会触发 get 方法。

**扩展：**

```python
# 私有方法，可以在类内部使用
# 可以有条件的开放给外部

class Student:

    def talk(self, identity):
        if identity == "铁子":
            self.__tellSecrect()
        else:
            print("speak...")

    def __tellSecrect(self):
        print("...")

s = Student()
s.talk("铁子")

'''
执行结果：
...
'''
