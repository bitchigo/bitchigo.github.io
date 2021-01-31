---
title: "记一次基于python+adb的游戏脚本"
date: 2021-01-31T13:03:43+08:00
draft: false
tags: ["游戏","python","adb","安卓","脚本"]
categories: ["python",“安卓”]
---

# 简述

由于日本游戏会检测root权限,甚者会检测虚拟机,于是便想到了由python编写脚本传递给adb执行,这样的话检测作弊也是比较困难的,

# 代码

```python
import aircv as ac #图像识别库
import subprocess
import time
import random
import os

tili=True  #标记本轮是否体力用空
def getScreen():
    subprocess.run(f"adb.exe shell screencap -p /sdcard/screen.png", capture_output=True)  #使手机截屏并存储到/sdcard/screen.png
    subprocess.run(f"adb.exe pull /sdcard/screen.png", capture_output=True)  #将图片拉取到电脑

def sendKey(width,height):
    subprocess.run(f'adb.exe shell input tap {width} {height}', capture_output=True)  #模拟点击屏幕

def find(btn):
    global tili
    imsrc = ac.imread('screen.png')   #屏幕图像
    imobj = ac.imread('btn/'+btn+'.png') #按钮图像
    pos=ac.find_template(imsrc, imobj)  #获取比对信息
    if pos and pos.get('confidence') > 0.70:  #相似度大于0.7时执行 
        if(btn=="end"):  #如果体力为空 标记本轮结束
            tili=False
        print(btn)  #debug
        w = [pos['rectangle'][0][0], pos['rectangle'][2][0]] #获取按钮宽度范围
        h = [pos['rectangle'][0][1], pos['rectangle'][1][1]] #获取按钮高度范围
        pos = [random.randint(w[0], w[1]), random.randint(h[0], h[1])]  #将范围取随机值
        sendKey(pos[0],pos[1])

subprocess.run(f"adb shell input keyevent 26", capture_output=True)  #电源键 亮屏
time.sleep(0.2)
subprocess.run(f'adb.exe shell input swipe 550 1400  540 580', capture_output=True) 
time.sleep(0.2)
subprocess.run(f'adb.exe shell input swipe 550 1400  540 580', capture_output=True) 
time.sleep(0.2)
subprocess.run(f"adb shell am force-stop com.klab.lovelive.allstars", capture_output=True)  #关闭游戏进程
time.sleep(0.2)
subprocess.run(f"adb shell am start com.klab.lovelive.allstars/.MainActivity", capture_output=True)  #启动游戏
time.sleep(32)
sendKey(1580,320)

while(True):
    while(tili):
        getScreen()
        find("jueding")
        find("pengyou")
        find("start")
        find("jiesuan")
        find("tongchang")
        find("live")
        find("tuijian")
        find("end")
        
    subprocess.run(f"adb shell am force-stop com.klab.lovelive.allstars", capture_output=True)
    subprocess.run(f"adb shell input keyevent 3", capture_output=True)   #home键
    subprocess.run(f"adb shell input keyevent 26", capture_output=True)  #锁屏
    tili=True
    time.sleep(3600)
```

# github地址

> https://github.com/bitchigo/lovelive_allstarts