---
title: 树莓派安装系统
date: 2020-11-5 14:53:11
tags:
- 树莓派
- 烧录
categories:
- 树莓派
---
## 官方系统

[系统镜像下载地址](https://www.raspberrypi.org/downloads/raspberry-pi-os/)

1. 插上读卡器
2. 查看存储设备 `df -h`
3. 卸载sd卡: `sudo diskutil unmount /Volumes/BOOT`
4. 烧录系统 `sudo dd if=/Users/chenbai/Downloads/2020-08-20-raspios-buster-armhf.img of=/dev/rdisk2 bs=128m`
5. 根目录创建空ssh文件，以及`wpa_supplicant.conf`配置文件

```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="624net"
    psk="imsosorry"
    priority=1
}
```

6. sd卡插入树莓派 启动
7. 获取ip  用ssh连接



## ubuntu server系统

[Raspberry Pi 安装 Ubuntu Server 20.04 LTS (无显示器, 无网线)]([Raspberry Pi 安装 Ubuntu Server 20.04 LTS (无显示器, 无网线) – 极夜喵 (h-xie.ren)](https://h-xie.ren/2020/raspberry-pi-安装-ubuntu-server-20-04-lts-无显示器-无网线/))

