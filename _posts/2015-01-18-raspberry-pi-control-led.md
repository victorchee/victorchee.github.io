---
layout: post
title: "树莓派控制LED"
category:  Raspberry Pi
---

## Bash控制LED
- LED的正极连接GPIO 25，负极接地  
- 接下来通过Bash命令来点亮或熄灭LED  
  - `sudo su`切换到root模式  
  - 在操作GPIO 25前，先要把这个引脚的操作接口从内核空间暴露到用户空间中：  
`echo 25 > /sys/class/gpio/export`  
  - 切换到刚新建的控制文件目录中去  
`cd /sys/class/gpio/gpio25`  
`ls`发现有好几个文件，我们只需要操作direction和value就可以了。  
  - direction文件用于指定这个GPIO接口会被用于输入或是输出，由于LED是输出设备，所以应该设置为输出模式：  
`echo out > direction`  
  - 使用echo命令向value文件中输出1，就可以点亮LED了  
`echo 1 > value`  
输出0，就可以熄灭LED  
`echo 0 > value`

## Python控制LED
- Raspbian系统中已经预装了GPIO模块，以root身份启动Python的交互式解释器，因为操作GPIO接口需要root权限。  
`sudo python`
- 导入GPIO模块  
`import RPi.GPIO as GPIO`
- 指定GPIO引脚编号  
`GPIO.setmode(GPIO.BCM)`
- 把GPIO 25设置为输出状态  
`GPIO.setup(25, GPIO.OUT)`
- 点亮LED  
`GPIO.output(25, GPIO.HIGH)`
- 熄灭LED  
`GPIO.output(25, GPIO.LOW)`
- 退出Python交互式解释器  
`exit()`

## Python控制LED闪烁

- 新建blick.py  
`vi blick.py`  
- 输入代码

```Python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setup(25, GPIO.OUT)

while True:
    GPIO.output(25, GPIO.HIGH)
    time.sleep(1)
    GPIO.output(25, GPIO.LOW)
    time.sleep(1)
```
- 运行
`sudo python blick.py`
- 关闭
`Ctrl+C`关闭，注意等LED熄灭的时候再关闭，如果LED点亮的状态下关闭的话，LED会保持常亮，不要直接去拔掉LED，因为25口还是会保持高电平状态，应该再次运行，待LED熄灭的时候再关闭。
