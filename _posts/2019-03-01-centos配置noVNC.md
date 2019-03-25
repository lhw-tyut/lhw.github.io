---
layout:     post
title:     centos 配置 noVNC
subtitle:   远程控制
date:       2019-03-01
author:     永泉狂客
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - VNC
---

## 1.关闭防火墙

systemctl stop firewalld

## 2.宿主机

**安装noVNC：**
git clone git://github.com/kanaka/noVNC

**创建安全连接（一路回车下去...）**

cd ./noVNC/utils/
openssl req -new -x509 -days 365 -nodes -out self.pem -keyout self.pem

## 3.客户机

**安装vncserver：**
yum install -y epel*
yum install -y git
yum install -y tigervnc-server

vncserver :1

## 操作示例:
客户机 ip:192.168.23.100
在客户机执行：vncserver :1

宿主机 ip:192.168.23.50
进入：noVNC目录下:

执行：./utils/launch.sh --vnc 192.168.23.100:5901

最后 浏览器:http://192.168.23.50:6080/vnc.html
![在这里插入图片描述](https://s2.ax1x.com/2019/03/25/AtyxzT.png)
*注：没有安装图形化界面的显示蓝屏*

## 补充：

vnc通常是连图形界面的。
所以你的服务器应该安装桌面系统（如gnome）之后，再通过vnc连接控制。如果你仅希望命令行控制，应该使用 ssh。ssh客户端 默认是文本界面的，并且在服务器端，ssh服务是系统默认安装的一个服务。

在安装了桌面系统的情况下按上述步骤就可实现，如果没有安装桌面系统，可能会有无法链接服务器的问题，在 /root/.vnc下的xstartup中
![在这里插入图片描述](https://s2.ax1x.com/2019/03/25/AtyvWV.gif)
将最后一行注释掉，这条语句会在启动vnc服务后就kill掉。
