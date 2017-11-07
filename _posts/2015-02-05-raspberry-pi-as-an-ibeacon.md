---
layout: post
title: "树莓派变身为iBeacon基站"
category:  Raspberry Pi
---

准备一个蓝牙适配器，注意不是所有的蓝牙适配器都适用于树莓派的，购买之前请先查清楚。我用的是奥睿科 BTA-406-RD USB蓝牙4.0适配器。

- 安装依赖库

`sudo apt-get install bluetooth bluez-utils blueman`

- 查看蓝牙设备

`hcitool dev`

- 查看蓝牙状态

`sudo service bluetooth status`
应该可以看到`[ ok ] bluetooth is running.`，如果不是，重启一下。

- 扫描设备

`hcitool scan`
出现你的设备`68:69:7C:31:9A:75 iPhone`

- 测试连接

`sudo l2ping -c 1 68:69:7C:31:9A:75`
如果能Ping通的话说明可以连接。

- 下载[linux-ibeacon](https://github.com/dburr/linux-ibeacon)

感谢此库的作者为我们提供了现成的脚本。

- 运行

`./ibeacon -h`查看帮助文档，按需运行。
