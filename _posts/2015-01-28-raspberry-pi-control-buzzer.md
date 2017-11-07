---
layout: post
title: "树莓派控制有源蜂鸣器"
category:  Raspberry Pi
---

- 连线  
有源蜂鸣器的正极接3.3V电源，负极接GPIO 23。
- 新建buzzer.py

```Python
import RPi.GPIO as GPIO
import time

def buzz(buzzTime):
        GPIO_PIN = 23;
        GPIO.setmode(GPIO.BCM)
        GPIO.setup(GPIO_PIN, GPIO.OUT);

        GPIO.output(GPIO_PIN, GPIO.LOW);
        time.sleep(buzzTime);
        GPIO.output(GPIO_PIN, GPIO.HIGH);
        GPIO.cleanup();

if __name__ == '__main__':
        buzz(3);
```
- 运行
`sudo python buzzer.py`
