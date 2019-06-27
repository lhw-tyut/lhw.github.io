---
layout:     post
title:      virsh-install命令
subtitle:   创建kvm虚拟机
date:       2019-05-10
author:     永泉狂客
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - linux
---
ubuntu环境配置：sudo apt-get install -y qemu libvirt-bin bridge-utils virt-manager


使用virt-install手动创建qcow2镜像并安装ISO

virt-install是一个使用libvirt库构建新虚拟机的命令行工具，此工具使用串行控制台，SDL（Simple DirectMedia Layer）图形或者VNC客户端/服务器，来支持命令行和图形安装
1.下载一个iso镜像  rhel-server-7.6-x86_64-dvd.iso
2.qemu-img和virt-install
1)创建qcow2格式文件为块设备
    qemu-img create -f qcow2 centos.qcow2 20G
2)使用virt-manager 或virt-install命令启动安装过程。如果使用virt-install命令，请不要忘记将VNC客户端连接到虚拟机。
    sudo virt-install --virt-type kvm --name $NAME --ram 4096 \
      --disk /tmp/redhat7.6-image.qcow2,format=qcow2 \
      --network network=default \
      --graphics vnc,listen=0.0.0.0 --noautoconsole \
      --os-type=linux --os-variant=rhel7 \
      --cdrom=/tmp/rhel-server-7.6-x86_64-dvd.iso


日志 vi /var/log/libvirt/qemu/centos.log
查看bridge     brctl show
虚拟机实例配置文件    cat /etc/libvirt/qemu/centos.xml

3.使用virsh vncdisplay vm-name 可以看到vm-name对应的端口号
可以使用VNC Viewer等VNC连接工具通过  宿主机ip:vm-name对应的端口号访问到虚拟机，此时就可以在这个虚拟机中安装操作系统了
4.virsh list --all 查看虚拟机

关机: virsh shutdown vm-name
开机: virsh start vm-name
yum install gdisk
