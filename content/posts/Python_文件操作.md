# Python_文件操作

# 📖文件操作

## 📕文件操作的基本流程

### 🎇绝对路径和相对路径

**绝对路径：** 从盘符开始			（`"F:\study_project\文件操作\test.txt"`）
**相对路径：** 型对于当前源码所在位置	（`"test.txt"`）

### 🎆编码格式

**指定编码格式，无乱码产生**

windows 是 GBK ，可以在设置更改

默认打开 pycharm 是 UTF-8 编码，所以可能会有乱码

```python
# 指定编码
f = open("test.txt", "w", encoding="UTF-8")
f.write("你好，世界")
f.close()
```

### 📕写(write)

#### 📕单行写入

```python
f = open("test.txt", "w")   # 不存在文件则新建
f.write("hello world")
f.close()   # 关闭文件  打开要关闭 避免占内存
```

#### 📕多行写入

```python
# writelines 一次性写入多行
f = open("test.txt", "w", encoding="UTF-8")
# f.write("你好，世界\n人生苦短\n我用Python")
content = ["你好，世界", "人生苦短", "我用Python"]
f.write("\n".join(content))
f.close()
```

### 📘读(read)

创建一个有 10 行 python 的文件

```python
f = open("test2.txt", "w", encoding="UTF-8")
f.write("Python\n" * 10)
f.close()
```

读取文件：

```python
f = open("test2.txt", "r", encoding="UTF-8")
data = f.read()
print(data)

'''
执行结果：
Python
Python
Python
Python
Python
Python
Python
Python
Python
Python
'''
```

重新读取文件按照**字符**读取：

```python
f = open("test2.txt", "r", encoding="UTF-8")
data = f.read(1)    # 读取1个字符
print(data)
data2 = f.read(2)   # 向后继续读2个字符  注：汉字也是一个字符
print(data2)
f.close()

'''
执行结果：
P
yt
'''
```

发现第二次读取并不是重新开始读取，而是接着重新打开后读取 1 个字符后的结尾 (**指针所在位置**) 读取。

#### 📘readline 读一行

**读取一行，返回一个字符串**

```python
f = open("test2.txt", "r", encoding="utf-8")
while True:
    content = f.readline()
    if content:
        print(f"{content}", end="")
    else:
        break

f.close()

'''
执行结果：
Python
Python
Python
Python
Python
Python
Python
Python
Python
Python
'''
```

#### 📘readlines 读多行

```python
f = open("test2.txt", "r", encoding="utf-8")
content = f.readlines()
print(content)
i = 1
for data in content:
    print(f"{i}:{data}", end="")
    i +=1
# 上述代码可用下面的函数enumerate()
# for i, data in enumerate(content):
#     print(f"{i}:{data}", end="")
f.close()

'''
执行结果：
['Python\n', 'Python\n', 'Python\n', 'Python\n', 'Python\n', 'Python\n', 'Python\n', 'Python\n', 'Python\n', 'Python\n']
1:Python
2:Python
3:Python
4:Python
5:Python
6:Python
7:Python
8:Python
9:Python
10:Python
'''
```

#### 📘 tell 返回指针当前所在位置(字节)

```python
f = open("test3.txt", "w", encoding="utf-8")
f.write("你好，世界")
f.close()
f = open("test3.txt", "r", encoding="utf-8")
content = f.read(2)                   # read读 字符
print(content)                        # gbk 中文占2个字节
print("当前指针所在位置：", f.tell())     # utf-8中文占3个字节
f.close()

'''
执行结果：
你好
当前指针所在位置： 6
'''
```

#### 📘seek 定位文件读取的指针所在位置(字节)

`f.seek(x, y)`
x : 开始的偏移量，也就是代表需要移动偏移的字节数
y : 可选，默认值为 0。给x参数一个定义，表示要从哪个位置开始偏移；0代表从文件开头开始算起，1代表从当前位置开始算起，2代表从文件末尾算起。

```python
f = open("test3.txt", "r", encoding="utf-8")
content = f.read(2)
print(content)
f.seek(4)
print("当前指针所在位置：", f.tell())

'''
执行结果：
你好
当前指针所在位置： 4
'''
```

## 📙文件访问模式

![文件访问模式](http://mucunliangtai.com/usr/uploads/2022/01/2139373727.png)

### 🏷访问不存在的文件

当文件不存在，w 和 a 可以新建文件
r 会报错

### ✏r+ 可读可写

```python
f = open("test4.txt", "r+", encoding="utf-8")
data = f.read()
f.write("我用Python")

f.seek(0)
f.write(" 我用Python")
f.close()
```

## ✒二进制文件操作

二进制文件的读写 （图片、音乐、视频等）

将文件的内容单纯的用 0 1 来进行读取

```python
fin = open("文件操作.png", mode="rb")
fout = open("文件操作_copy.png", mode="wb")

# 方式一
# while True:
#     data = fin.read(100)
#     if data:
#         fout.write(data)
#     else:
#         break

# 方式二
while True:
    data = fin.read(100)
    if data != b"":
        fout.write(data)
    else:
        break
fin.close()
fout.close()
```

## 🔑缓冲区

**写入过程：** 数据 --> 缓冲区（内存) --> 文件中（硬盘)

flush 刷新缓冲区

### 🔑 truncate 截断

```python
# 新建文件test5
f = open("test5.txt", "w", encoding="utf-8")
f.write("Python"*5)
f.close()

f = open("test5.txt", "r+", encoding="utf-8")
f.seek(11)
f.truncate()  # 从指针位置到结束位置全部删除
# f.truncate(8)     从开始位置到指定位置 保存
f.close()
```

## 📝文件对象 可迭代

```python
f = open("test2.txt", "r", encoding="utf-8")
for line in f:
    print(line, end="")
f.close()

f = open("test5.txt", "r")
print("文件名：", f.name)
print("文件打开的模式", f.mode)
print("文件可写：", f.writable())
print("文件可读：", f.readable())

'''
执行结果：
Python
Python
Python
Python
Python
Python
Python
Python
Python
Python
文件名： test5.txt
文件打开的模式 r
文件可写： False
文件可读： True
'''