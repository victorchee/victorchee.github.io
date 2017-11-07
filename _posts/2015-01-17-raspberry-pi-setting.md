---
layout: post
title: "树莓派的基本配置"
category:  Raspberry Pi
---

## 购买[Raspberry Pi](http://www.raspberrypi.org/)和配件
- 在ickey.cn买了一个套装，花了308  
包含一块B+板，亚克力外壳，EDUP免驱USB无线网卡和铝制散热片3个。
- 在京东上买了一个TF卡，SanDisk Ultra 16G class10，花了42.8  
一个5V2.1A的杂牌充电器，花了29.9。
- 由于自己只有Lighting转DVI的线，所以拿了公司的HDMI转DVI线先配置一下，之后都用SSH，用不着显示器了。如果用HDMI转VGA的线，最好用有源的，因为HDMI转DVI只是数字信号转换，并不需要电源，VGA就不行了。
- 去新街口买了个门磁开关，本来想买个常闭的，结果只有个常开的，所以又买了个继电器做转换。还买了若干电阻、发光二极管、按钮开关、蜂鸣器和杜邦线。一共花了43块。

## Mac下烧录系统映像到TF卡
- 去 http://www.raspberrypi.org/downloads/ 下载Raspbian  
得到一个img文件和一个zip文件。
- `df -h`查看设备列表
![1](/images/raspberry-pi-setting/1.png)
- 插入TF卡后再`df -h`
![2](/images/raspberry-pi-setting/2.png)
多出来一个/dev/disk2s1就是TF卡，各人的可能稍微有点区别，请相应修改。
- 卸载掉TF卡，以便`dd`命令将映像写入到TF卡中  
输入：`sudo diskutil unmount /dev/disk2s1`，然后输入密码，  
输出：Volume NO NAME on disk2s1 unmounted
- 写入映像到TF卡中  
将设备名，这里是/dev/disk2s1，把最后的s1入掉然后在前面加上r，修改为/dev/rdisk2，这个就是TF卡的原始设备名称。  
输入：`sudo dd bs=1m if=2014-12-24-wheezy-raspbian.img of=/dev/rdisk2`  
输出：`3125+0 records in  
3125+0 records out  
3276800000 bytes transferred in 229.204813 secs (14296384 bytes/sec)`
- 移除TF卡  
写入完成后发现TF卡的名称变为‘boot’了，右键‘Eject’安全移除即可。

## 配置Raspberry Pi
- Expand FileSystem  
把SD卡中的镜像文件扩展到整张卡中，充分利用SD卡的空间。
- Change User Password  
最好设置一个自己的常用密码，默认密码不安全。
- Internationalisation Options  
  - Change Locale  
空格反选en_GB.UTF-8,移到en_US.UTF-8空格选中，回车确定，选中en_US.UTF-8，再回车确定。
  - Change Timezone  
选择Asia->Shanghai
  - Change Keyboard Layout  
直接选择Generic 105-key(Intl) PC，然后选择other->English(US)，并选择No compose key，打开Ctrl+Alt+Backspace组合键功能。
- Advanced Options  
  - Overscan  
如果出现屏幕图像没有完全铺满显示器，则禁用overscan，如果没有问题的话，跳过此步。
- Finish  
reboot出现raspberrypi login:输入pi和刚设置的密码

## 配置无线网卡
- 插入网卡，运行`lsusb`  
看到有一条是WLAN Adapter，表明已经网卡已经识别。
- 运行`ifconfig`  
发现wlan0还未分配IP地址。
- 运行`sudo vi /etc/wpa_supplicant/wpa_supplicant.conf`打开网络配置文件，在最后添加：

```
network={
    ssid="Wi-Fi账号"
    psk="Wi-Fi密码"
}
```
:x保存退出    
- 过几秒应该已经连上了网络，如果没有则重启网络服务
`sudo /etc/init.d/networking restart`
- 再次调用`ifconfig`，发现wlan0已经有IP地址了，Ping一下百度，也能Ping通了。以后每次启动都会自动连接上WIFI了。

## Raspbian的基本操作
- 更换源    
在http://www.raspbian.org/RaspbianMirrors这里找到最快的源    
`sudo vim /etc/apt/sources.list`按照原来的源把找到的源更新上去，把原来的注释掉    
- 更新系统  
`sudo apt-get update`更新软件信息数据库  
`sudo apt-get dist-upgrade`更新已安装软件  
`sudo apt-get upgrade`系统升级  
- `sudo init 5`或`sudo startx`启动图形界面  
Ctrl+Alt+F1再切换到tty1工作组。
- SSH  
Raspbian自带了SSH－Server，无需再做配置，在Mac的终端直接运行`ssh pi@IP-Address`，输入密码即可登录，exit退出。  
Tip：`ssh -o ServerAliveInterval=60 pi@IP-Address`，此参数会每隔60S给树莓派发送一个心跳包，从而保持长连接。
- 关机
`sudo shutdown -h now`其实就是调用的`sudo halt`。
- 重启
`sudo shutdown -r now`或者`sudo reboot`。
- 用Tmpfs延长TF卡的使用寿命
TF卡作为树莓派的硬盘，避免不了频繁读写操作。
通过Tmpfs可以将一些太频繁读写的目录挂载到内存中，把这些目录写入到`/etc/fstab`(使用root权限打开)中去：

```
tmpfs /tmp tmpfs defaults,noatime,nosuid,size=100m 0 0
tmpfs /var/tmp tmpfs defaults,noatime,nosuid,size=30m 0 0
tmpfs /var/log tmpfs defaults,noatime,nosuid,mode=0755,size=100m 0 0
tmpfs /var/spool/mqueue tmpfs defaults,noatime,nosuid,mode=0700,gid=12,size=30m 0 0
```
`:x!`保存，重启树莓派即可生效。
注意：所有挂载到tmpfs的文件因为在内存中，重启时会丢失，所以不要把需要重启后依然有用的文件挂载上去。
