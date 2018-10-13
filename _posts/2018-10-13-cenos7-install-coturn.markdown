---
layout: post
title:  "Centos7安装WebRtc打洞服务器Coturn方法"
date:   2018-10-13 22:21:49
categories: WebRtc
tags: WebRtc
---
# Centos7安装WebRtc打洞服务器Coturn方法
> 在使用WebRtc时，我们需要打洞服务器来打洞两部设备之间的通信，这里我们采用Coturn库。由于Turn服务器是Stun的一个拓展，Coturn包括了Turn和Stun，所有我们只需要部署Coturn就可以完成WebRtc的打洞环节啦。

## 克隆并安装
```bash
git clone https://github.com/coturn/coturn 
cd coturn 
./configure 
make 
sudo make install
```
如果你的电脑上没有安装LibEvent2，需要先安装`libevent-devel`
```bash
# Install the libevent-devel rpm package:
yum install libevent-devel
```
安装好之后使用`which turnserver`确保安装成功
```bash
[root@localhost local]# which turnserver
/usr/local/bin/turnserver
```
## 设置配置文件
在Coturn编译完成好之后会自动生成一个配置文件的模板，在`/usr/local/etc/turnserver.conf.default`，感兴趣的小伙伴可以仔细查看里面每一个配置项的含义。这里我们在新建一个新的配置文件`/usr/local/etc/turnserver.conf`，在这个里面编辑好之后使用`turnserver`命令会自动寻找到conf文件的位置。分享一个简单的配置文件的格式：
```
relay-device=enp1s0f0  //外网网卡的设备号
listening-ip=x.x.x.x //内网IP，没有填外网IP也可以
listening-port=3478 
relay-ip=x.x.x.x  //内网IP，没有填外网IP也可以
external-ip=x.x.x.x  //外网IP
relay-threads=500 
lt-cred-mech 
pidfile=”/var/run/turnserver.pid” 
min-port=49152 
max-port=65535 
user=xxx:123456
realm=AnHui
```
保存好之后使用命令
```bash
turnserver -o -a -f 
```
即可启动Coturn。
## 验证Coturn的可用性
有一个专门的网站可以检查打洞服务器的正确配置与否。
[Trickle ICE](https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/)
### 验证stun
输入 `stun:x.x.x.x:3478`
得到结果
```
Time	Component	Type	Foundation	Protocol	Address	Port	Priority
0.002	1	host	886443856	udp	10.80.1.131	49469	126 | 32542 | 255
0.104	1	host	2052453280	tcp	10.80.1.131	9	90 | 32542 | 255
0.288	1	srflx	2643034245	udp	112.27.203.124	49469	100 | 32542 | 255
0.312	Done
0.313
```
看到`srflx`后面就是你的电脑的外网IP，表示打洞成功。
### 验证turn
输入 `turn:x.x.x.x:3478`,`username:`,`password`
得到结果
```
0.003	1	host	886443856	udp	10.80.1.131	55831	126 | 32542 | 255
0.104	1	host	2052453280	tcp	10.80.1.131	9	90 | 32542 | 255
0.534	1	srflx	2643034245	udp	112.27.203.124	55831	100 | 32542 | 255
0.614	1	relay	3676437432	udp	x.x.x.x	56631	2 | 32542 | 255
0.878	Done
0.880
```
看到`relay`后面就是你的服务器的外网IP，表示可以使用Coturn的turn服务器进行转发。同时也可以看见srflx，这说明了turn服务是stun的一个拓展，turn和stun是包含的关系。

> Coturn的部署就简单的介绍到这里，如有不对的地方，还望指正，谢谢！
