---
layout: post
title: "在树莓派上搭建BT下载"
category: Raspberry Pi
---

`sudo apt-get install transmission-daemon`

`sudo service transmission-daemon stop`

`sudo vim /etc/transmission-daemon/settings.json`
可配置的选项有很多，这里挑几个常用的讲一下。
"port-forwarding-enabled" : true 设置端口转发
1> "download-dir" : "/home/pi/bt" 设置下载后的资源保存的路径。
2> "rpc-authentication-required" : true 远程控制时需要验证
3> "rpc-enabled" : true 开启远程控制    ip:9091/transmission/
4> "rpc-password" & "rpc-username" 这两项设置验证时的用户名和密码
5> "rpc-whitelist-enabled": false 关闭白名单功能(即允许从任何ip地址远程控制)

`sudo service transmission-daemon restart`










`sudo apt-get install aria2`
`sudo aria2c --conf-path=/etc/aria2/aria2.conf`
`sudo killall aria2c`

`sudo apt-get install nginx`
`sudo /etc/init.d/nginx start`

`sudo vim /etc/nginx/nginx.conf`
由于/var/log挂载在内存上，需要创建nginx/error.log才行，要不然reboot之后没有这个目录nginx会报错
`sudo vim /etc/rc.local`
在exit 0之前加入
`
mkdir /var/log/nginx
touch /var/log/nginx/error.log
`

`rm -rf /usr/share/nginx/html`
`git clone https://github.com/ziahamza/webui-aria2.git /usr/share/nginx/html`

http://blog.flag.moe/to-build-nas
http://www.jianshu.com/p/4cf37177fc62
