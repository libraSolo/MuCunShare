+++
date = '2021-8-8T16:21:57+08:00'
draft = false
title = 'Python_游戏脚本思路及API'
author = '木村凉太'
categories = 'Python'
hiddenFromHomePage = true 
+++


# Python_游戏脚本思路及API

# EVE游戏自动检测喊话

通过图片识别来获取文本，并自动喊话

## 获取窗口

代码主要用来获取电脑启动的程序，以程序名字来获取，也可以用来匹配其他窗口应用程序。

```python
def foo(hwnd, mouse):
    # 获取窗口
    global titles
    if IsWindow(hwnd) and IsWindowEnabled(hwnd) and IsWindowVisible(hwnd):
        titles.add(GetWindowText(hwnd))

EnumWindows(foo, 0)
lt = [t for t in titles if t]
lt.sort()
for t in lt:
    if (t.find('MuMu模拟器')) >= 0:
        hwnd = FindWindow(None, t)
        print('MuMu模拟器窗口:',hwnd)
def play_game():
    # 置顶窗口，调整尺寸
    global titles
    EnumWindows(foo, 0)
    lt = [t for t in titles if t]
    lt.sort()
    for t in lt:
        if(t.find('MuMu模拟器')) >= 0:
            hwnd = FindWindow(None, t)
            # print(hwnd)
            SetWindowPos(hwnd, win32con.HWND_TOP,0,0,1280, 720,win32con.SWP_SHOWWINDOW)	# 第二个参数可以调整置顶方式
            hwnd=FindWindow(None, t)
            # print(hwnd)
            size = GetWindowRect(hwnd)
            # print(size)
            return size
```

## 部分接口

模拟键盘或者鼠标按键操作

```python
# 模拟鼠标点击
def mouse_click(x, y):
    win32api.SetCursorPos([x, y])   # 移动鼠标位置
    win32api.mouse_event(win32con.MOUSEEVENTF_LEFTDOWN, 0, 0, 0, 0)
    win32api.mouse_event(win32con.MOUSEEVENTF_LEFTUP, 0, 0, 0, 0)

# 返回粘贴板的内容
def getText():
    win32clipboard.OpenClipboard()
    d = win32clipboard.GetClipboardData(win32con.CF_TEXT)
    win32clipboard.CloseClipboard()
    return d

# 模拟 Ctrl + C
def setText(aString):
    win32clipboard.OpenClipboard()
    win32clipboard.EmptyClipboard()
    win32clipboard.SetClipboardData(win32con.CF_TEXT, aString.encode('gbk'))
    win32clipboard.CloseClipboard()

# 模拟 Ctrl + V
def ctrlV():
  win32api.keybd_event(17,0,0,0) #ctrl
  win32api.keybd_event(86,0,0,0) #V
  win32api.keybd_event(86,0,win32con.KEYEVENTF_KEYUP,0)#释放按键
  win32api.keybd_event(17,0,win32con.KEYEVENTF_KEYUP,0)
```

## OCR识别

此处使用 pytesseract、PIL ，需要安装 pytesseract-OCR，可以在网上找资源，安装后更改 pytesseract.py 文件的最开始的一行代码，需要指定 pytesseract-OCR 的根路径，即可使用。

```python
save_path = 'num.png'
x ,x2 = 100+72.5*i, 135 +75*i
img = ImageGrab.grab(bbox=(x,310,x2,335))	# 更改尺寸大小，方便识别
img = img.resize((140, 100))		# 图片反转颜色，方便识别
ImageOps.invert(img).save(save_path)	# 保存图片

# 识别数字
num = pytesseract.image_to_string(Image.open(save_path),config='--psm 10 --oem 3 -c tessedit_char_whitelist=0123456789')

```

`pytesseract.image_to_string(img)`，后续的 config 是识别英文、数字、中文之类的参数，此处我也不是很理解，可以识别不清楚时寻找相应的参数即可。

## 完整代码

```python
import time
import re
from win32gui import *
import win32con
import win32api
import win32clipboard
from PIL import Image, ImageGrab,ImageOps
import os
import pytesseract
from pynput.keyboard import Key,Controller


titles = set()
findnum = re.compile(r'<.*>(.*)</.*>')
eveCnf = []
global OCRnum
OCRnum = 0

def foo(hwnd, mouse):
    # 获取窗口
    global titles
    if IsWindow(hwnd) and IsWindowEnabled(hwnd) and IsWindowVisible(hwnd):
        titles.add(GetWindowText(hwnd))

EnumWindows(foo, 0)
lt = [t for t in titles if t]
lt.sort()
for t in lt:
    if (t.find('MuMu模拟器')) >= 0:
        hwnd = FindWindow(None, t)
        print('MuMu模拟器窗口:',hwnd)

def play_game():
    # 置顶窗口，调整尺寸
    global titles
    EnumWindows(foo, 0)
    lt = [t for t in titles if t]
    lt.sort()
    for t in lt:
        if(t.find('MuMu模拟器')) >= 0:
            hwnd = FindWindow(None, t)
            # print(hwnd)
            SetWindowPos(hwnd, win32con.HWND_TOP,0,0,1280, 720,win32con.SWP_SHOWWINDOW)
            hwnd=FindWindow(None, t)
            # print(hwnd)
            size = GetWindowRect(hwnd)
            # print(size)
            return size




# 模拟鼠标点击
def mouse_click(x, y):
    win32api.SetCursorPos([x, y])   # 移动鼠标位置
    win32api.mouse_event(win32con.MOUSEEVENTF_LEFTDOWN, 0, 0, 0, 0)
    win32api.mouse_event(win32con.MOUSEEVENTF_LEFTUP, 0, 0, 0, 0)

# 返回粘贴板的内容
def getText():
    win32clipboard.OpenClipboard()
    d = win32clipboard.GetClipboardData(win32con.CF_TEXT)
    win32clipboard.CloseClipboard()
    return d

# 模拟 Ctrl + C
def setText(aString):
    win32clipboard.OpenClipboard()
    win32clipboard.EmptyClipboard()
    win32clipboard.SetClipboardData(win32con.CF_TEXT, aString.encode('gbk'))
    win32clipboard.CloseClipboard()

# 模拟 Ctrl + V
def ctrlV():
  win32api.keybd_event(17,0,0,0) #ctrl
  win32api.keybd_event(86,0,0,0) #V
  win32api.keybd_event(86,0,win32con.KEYEVENTF_KEYUP,0)#释放按键
  win32api.keybd_event(17,0,win32con.KEYEVENTF_KEYUP,0)

# 主要文本识别以及具体游戏实现
def game():
    size = play_game()
    numList = []
    for i in range(3):
        save_path = 'num.png'
        x ,x2 = 100+72.5*i, 135 +75*i
        img = ImageGrab.grab(bbox=(x,310,x2,335))
        img = img.resize((140, 100))
        ImageOps.invert(img).save(save_path)

        num = pytesseract.image_to_string(Image.open(save_path),config='--psm 10 --oem 3 -c tessedit_char_whitelist=0123456789')
        num = num.split('\n')[0]
        numList.append(num)
    for i in range(len(numList)):
        for j in numList[i]:
            if j == 'S':
               numList[i] = numList[i].replace("S","5")
            elif j == 'q':
                numList[i] = numList[i].replace("q", "9")
            elif j == 'l':
                numList[i] = numList[i].replace("l", "1")
            elif j == 'e':
                numList[i] = numList[i].replace("e", "2")
            elif j == 'o' or 'O':
                numList[i] = numList[i].replace("o", "0")
                numList[i] = numList[i].replace("O", "0")

    str4 = eveCnf[2]
    str3 = str4.replace('x',str(numList[0]))
    str2 = str3.replace('y',str(numList[1]))
    str1 = str2.replace('z',str(numList[2]))
    print(f'第 {OCRnum} 次OCR识别'+str1)
    try:
        if int(numList[0]) < eveCnf[1] and int(numList[1]) <eveCnf[1] and int(numList[2]) < eveCnf[1]:
            print('数值小于'+str(eveCnf[1])+','+str(eveCnf[0])+'s后再次执行')
            time.sleep(eveCnf[0])
        else:
            setText(str1)
            mouse_click(265, 550)
            time.sleep(2)
            mouse_click(230, 630)
            time.sleep(2)
            ctrlV()
            mouse_click(490, 470)
            time.sleep(2)
            mouse_click(480, 630)
            time.sleep(2)
            mouse_click(510, 60)
            time.sleep(eveCnf[0])
    except:
        print('读取数字可能失败，等待1s后再次读取')
        time.sleep(1)

def readCnf():
    f = open('./eve.cnf', 'r+')
    for i in range(3):
        tmp = f.readline()
        tmp = re.findall(findnum, tmp)[0]
        if i == 0 or i == 1:
            eveCnf.append(int(tmp))
        else:
            eveCnf.append(tmp)
    f.close()

def EVEsize():
    # 置顶窗口，调整尺寸
    global titles
    EnumWindows(foo, 0)
    lt = [t for t in titles if t]
    lt.sort()
    for t in lt:
        if(t.find('\EVE')) >=0:
            hwnd = FindWindow(None, t)
            SetWindowPos(hwnd, win32con.HWND_TOP,1000,300,500, 200,win32con.SWP_SHOWWINDOW)

if __name__ == '__main__':
    readCnf()
    EVEsize()
    print('模拟器设置画面亮度为100/普通，本地统计拉至左上角，此脚本不阻挡本地统计')
    input("输入数字 1 并回车则开始预警：")

    while True:
        OCRnum +=1
        game()