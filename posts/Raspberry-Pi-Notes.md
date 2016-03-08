---
date: 2016-01-15
layout: post
title: Raspberry Pi notes
permalink: '/2016/raspberry-pi-notes.html'
categories:
- Hardware
- Blog
tags:
- Hardware
- RaspberryPi

---

最近又入手了一个Raspberry Pi2，大致使用记录一下

## 安装Raspbian ##
直接到官网下载img [http://www.raspberrypi.org/downloads ](http://www.raspberrypi.org/downloads )
windows通过win32 disk imager来写入tf卡或者sd卡（raspberry pi B+为SD卡）

## 配置Raspberry Pi ##
通过**raspi-config**来配置，可以expeand filesystem，或者修改启动类型，或者地区，或者修改密码。当然这些也都可以通过命令行来解决。

## 更换sources ##
由于没有更换软件源，导致下载略慢，改用国内镜像，那速度杠杠滴。
官网有镜像地址列表: [http://www.raspbian.org/RaspbianMirrors](http://www.raspbian.org/RaspbianMirrors), 这里就选母校的mirror吧
[http://mirrors.zju.edu.cn/raspbian/raspbian/](http://mirrors.zju.edu.cn/raspbian/raspbian/)

那就用工具来修改/etc/apt/sources.list

`sudo vi /etc/apt/sources.list`

保存好之后，需要执行`sudo apt-get update`来更新软件列表。

## IP怎么获取 ##
* 通过路由器获取ip（得有权限，如果是智能路由器，可以app上直接查）
* 可以通过脚本把ip发到邮件，每次启动就收到ip的邮件；

[树莓派Raspbian开机自动发ip邮件的解决方案](http://www.tuicool.com/articles/2AvIn2)

[让树莓派自动上报IP地址到邮箱](http://shumeipai.nxez.com/2014/03/18/let-raspberry-pi-ip-address-is-automatically-reported-to-the-mailbox.html)

[自动发送本机ip到指定邮箱](http://www.opstool.com/article/299)

* 通过arp scan来查局域网内的设备信息（公司里面查找就略麻烦）
[HOWTO discover the ip address of a Raspberry Pi](http://blog.remibergsma.com/2013/05/03/howto-discover-the-ip-address-of-a-raspberry-pi-on-dhcp/)


## 登录Raspberry Pi ##
> 默认账号为 pi/raspberry

Enjoy it~


