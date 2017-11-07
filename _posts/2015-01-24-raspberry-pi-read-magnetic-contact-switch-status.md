---
layout: post
title: "树莓派读取门磁开关状态"
category:  Raspberry Pi
---

## 接线
当从GPIO接口上读取数据时，实际上是检查这个接口是被接在3.3V电源上还是被接地了，输入接口必须连接一个确定的信号（3.3V或接地），如果尝试读取一个既未接电源又未接地的引脚，会得到一个不稳定的结果。
注意：需要输入高电平时，只能输入3.3V电源，不能接5V电源，向GPIO接口输入大于3.3V的电压会烧坏树莓派。

- 门磁开关正极接3.3V电源
- 门磁开关负极接GPIO 24
这样门磁开关闭合时，3.3V电压会输入到GPIO 24接口。
- 取一个10kΩ的电阻，一端接地，一端接GPIO 24

因为，门磁开关悬空的情况下，会带来不稳定的数据读取结果，所以加个电阻在GPIO 24接口上，如果门磁开关没有闭合的时候，GPIO 24是通过电阻接地的，闭合的时候，由于门磁开关的电阻小于10kΩ，3.3V电压会接入到GPIO 24。

## Bash读取GPIO状态
- sudo su
获取root权限
- 把输入接口暴露到用户态  
`echo 24 > /sys/class/gpio/export`
- 切换到刚生成的目录中去  
`cd /sys/class/gpio/gpio24`
- 设置接口的工作模式为输入模式  
`echo in > direction`
- 读取门磁开关状态  
`cat value`  
输出0表示接地，即门磁开关未闭合  
输出1表示门磁开关未闭合

## Python读取GPIO状态
- 新建switch.py

```Python
import RPi.GPIO as GPIO

GPIO.setmode(GPIO.BCM)
GPIO.setup(24, GPIO.IN, pull_up_down=GPIO.PUD_UP)

while True:
        GPIO.wait_for_edge(24, GPIO.RISING)
        print("switch on")
        GPIO.wait_for_edge(24, GPIO.FALLING)
        print("switch off")
```
- 运行：`sudo python switch.py`
门磁开关改变的时候，会输出相应的状态信息。
