---
layout: post
title: "在树莓派安装XX-Net"
category: Raspberry Pi
---

`mkdir xxnet`
`cd xxnet`
https://github.com/XX-net/XX-Net/blob/master/code/default/download.md
找到稳定版的下载地址
`wget https://codeload.github.com/XX-net/XX-Net/zip/3.2.8`
`unzip 3.2.8`
`cd XX-Net-3.2.8`
`./start`
提示未安装pyOpenSSL
`sudo apt-get install python-openssl`
`./start`
设置允许远程连接Web控制端
`cd data/launcher`
`vim config.yam`
修改allow_remote_connect为1，即允许远程访问Web控制端http://raspberry-pi-ip:8085
`./start`
按照
https://github.com/XX-net/XX-Net/wiki/how-to-create-my-appids
配置自己的Google App ID

`nohup ./start &`
`jobs -l`
`kill 807`

设置 xx-net 开机自动运行: 在 /etc/rc.local 中 exit 0之前加入
~/xxnet/XX-Net-3.2.8/start


http://wiki.guoruei.org/computer/software/linux/install-shadowsocks-on-raspberry-pi-at-local
