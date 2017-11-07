---
layout: post
title: "在树莓派上共享配置"
category: Raspberry Pi
---

`sudo apt-get install samba samba-common-bin`
`sudo vim /etc/samba/smb.conf`
`
[Raspberry Pi]
    path = /mnt/disk
    valid users = root pi
    browseable = yes
    public = yes
    writable = yes
    guest ok = yes
`
`sudo touch /etc/samba/smbpasswd`
`sudo smbpasswd -a pi`
输入密码‘chee’
`sudo /etc/init.d/samba restart`



http://www.jianshu.com/p/57956e7050ad
