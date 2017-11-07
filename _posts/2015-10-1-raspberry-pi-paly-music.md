---
layout: post
title: "树莓派播放音乐"
category:  Raspberry Pi
---

硬件：带电源音响。

软件：MPlayer。

- **安装MPlayer**

`sudo apt-get install mplayer`

- **上传音乐到树莓派**

由于Mac自带Python环境，所以用Python创建一个微型Web服务器。先切换到音乐目录，然后执行

`python -m SimpleHTTPServer 6666`

找到本机的IP地址，我的是192.168.0.100，然后`Http://192.168.0.100:6666`即可访问到文件。

然后在树莓派上创建个目录用来放音乐`mkdir music`，切换到此目录之后下载音乐

`wget Http://192.168.0.100:6666/name.mp3`

- **播放**

`mplayer name.mp3`可播放单首歌曲；

在音乐目录下执行`ls > music.lst`来生成播放列表；

执行`mplayer -playlist music.lst`即可播放改列表。

`mplayer -shuffle -playlist music.lst`可随机播放。

- **关闭**

`killall mplayer`

- **调音量**

Shell中播放音乐时，音量都不大，所以我用alsamixer把音量跳到最大，然后在音响上调节音量。

`sudo apt-get install alsa-utils`

执行`alsamixer`把音量调到最大。

- **定时播放**

下一步再说……
