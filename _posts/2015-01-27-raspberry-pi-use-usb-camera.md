---
layout: post
title: "树莓派上使用USB摄像头"
category:  Raspberry Pi
---

原来有个老的USB摄像头，现在可以派上用场了，装在树莓派上充当眼睛吧。

## Motion

- 更新系统和软件

`sudo apt-get update`
`sudo apt-get upgrade`

- 更新固件（firmware）

`sudo rpi-update`

- 插上usb摄像头，输入`ls /dev`, 查看dev中是否存在一个video0这个设备，如果有了说明系统识别了，如果没有，重复上一步确保firmware最新，还不行就是驱动问题。

- 安装网络监控软件motion

`sudo apt-get install motion`

- 配置motion，编辑`/etc/motion/motion.conf`文件

`control_localhost off`
允许外网访问该端口查看和修改motion.conf里的配置
`webcam_localhost off`
不限制只在局域网内访问motion的监控结果

- 运行motion软件，输入

`sudo motion`

- 用浏览器打开

配置网页：http://IP:8080
监控网页：http://IP:8081

## Python

- 新建camera.py文件

```python
import pygame
import pygame.camera
from pygame.locals import *

# initialize
pygame.init()
pygame.camera.init()

# capture a image
camera = pygame.camera.Camera("/dev/video0", (640, 480))
camera.start()
image = camera.get_image()
pygame.image.save(image, "image.jpg")
camera.stop()
```
- 运行`sudo python camera.py`，会有一张图片保存到当前目录。
