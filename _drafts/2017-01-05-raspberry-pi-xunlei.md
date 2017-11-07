---
layout: post
title: "在树莓派上搭建迅雷下载"
category: Raspberry Pi
---

下载Xware1.0.31_armel_v5te_glibc.zip
`unzip Xware1.0.31_armel_v5te_glibc.zip`
`./portal`
得到控制端口激活码，去http://yuancheng.xunlei.com添加激活码
在浏览器中输入http://192.168.1.1:9000/getsysinfo（IP替换为你路由器的IP，端口换为你的控制端口）
[ 0, 1, 1, 0, “7DHS94”, 1, “201_2.1.3.121”, “shdixang”, 1 ]
其中有用的几项为：

第一项：0表示返回结果成功
第二项：1表示检测网络正常，0表示检测网络异常
第四项：1表示已绑定成功，0表示未绑定
第五项：未绑定的情况下，为绑定的需要的激活码
第六项：1表示磁盘挂载检测成功，0表示磁盘挂载检测失败

查看硬盘
`sudo fdisk -l`
格式化成ext4
`sudo mkfs.ext4 /dev/sda1`
支持exFat
`sudo apt-get install exfat-fuse`
挂载硬盘sda1到/mnt/disk
`sudo mount /dev/sda1 /mnt/disk`
`sudo chown pi:pi /mnt/disk/`
取消挂载
`sudo umount /mnt/disk`
查看挂载
`df -h`
查看硬盘UUID
`sudo blkid`
自动挂载
`sudo vim /etc/fstab`
加入`UUID=d5a3d30a-d2e7-4b12-bb31-b4439c5db200 /home/pi/nas-data ext4 defaults 0 2`

在迅雷目录新建./etc/thunder_mounts.cfg
`
avaliable_mount_path_pattern
{
/mnt/disk
}
`
关闭迅雷
`./portal -s`

http://mkitby.com/2016/05/15/raspberry-pi-nas-manage-hdd-power/
硬盘自动休眠
安装 hdparm，
sudo apt-get install hdparm -y
确保你的硬盘支持hdparm,
sudo hdparm -y /dev/sdb
如果输出如下结果，就表明支持，
/dev/sdb:
 issuing standby command
如果看到类似如下输出，就表明不支持，可以跳到 hd-idle 或者 sdparm，
/dev/sda:
 issuing standby command
SG_IO: bad/missing sense data, sb[]:  70 00 05 00 00 00 00 0a 00 00 00 00 20 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
检查硬盘是否支持写缓存功能，
sudo hdparm -I /dev/sdb | grep 'Write cache'
如果看到 * (星号)，表示支持，
*    Write cache
如果没有看到 * (星号)，表示不支持，
Write cache
编辑配置文件，
sudo vim /etc/hdparm.conf
spindown_time值乘以 5 得到总的时间（单位秒）. 例如想配置成空闲10分钟就休眠，spindown_time = 10 * 60 / 5 = 120
在文件结尾加入，
/dev/sdb {
write_cache = on
spindown_time = 120
}
如果不支持写缓存，去掉 write_cache = on。

重启 hdparm 服务，

sudo service hdparm restart
更多hdparm配置可以参考这(http://www.linux-magazine.com/Online/Features/Tune-Your-Hard-Disk-with-hdparm)


参考：
http://luyou.xunlei.com/forum.php?mod=viewthread&tid=15167&extra=page%3D1%26filter%3Dtypeid%26typeid%3D1
http://blog.flag.moe/to-build-nas
http://www.itdadao.com/articles/c15a925162p0.html
http://www.jianshu.com/p/17cee17159f4
http://blog.flag.moe/to-build-nas

http://www.dreamxu.com/raspberrypi-nas/
