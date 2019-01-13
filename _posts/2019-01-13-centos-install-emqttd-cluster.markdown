---
layout: post
title:  "在CentOS7上使用Docker和Haproxy部署Emq集群"
date:   2019-01-13 22:51:49
categories: Emq
tags: Emq
---
## 1.安装Docker
由于yum默认是没有Docker源的，所以安装之前需要先安装Docker源，这一步参照[官方安装文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)，直接复制黏贴代码即可。
```bash
# update yum package index（更新yum索引）
sudo yum update
# Install packages to allow yum to install docker（安装相关工具）
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
# set up the stable repository（安装docker stable源）
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
# update yum package index（安装了新的源，所以再次更新索引）
sudo yum update
#Install the latest version of Docker CE（这里安装最新的Docker CE版本）
sudo yum install docker-ce
```
朝上面在安装Docker源的那一步在天朝需要**科学上网**，**科学上网**，**科学上网**，说三遍。一番操作之后Docker应该是安装完成了，使用`docker version`命令，没有报错则安装成功。
```bash
[root@localhost ~]# docker version
Client:
 Version:           18.09.1
 API version:       1.39
 Go version:        go1.10.6
 Git commit:        4c52b90
 Built:             Wed Jan  9 19:35:01 2019
 OS/Arch:           linux/amd64
 Experimental:      false
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
使用`sudo systemctl start docker`命令即可启动Docker。

## 2.使用Docker启动两个emqttd容器
这里我在Docker hub（需要科学上网）找到了一个别人已经构建好的镜像[tldzyx/emqttd](https://hub.docker.com/r/tldzyx/emqttd)（访问需要科学上网）。
我们拉下来只有后分别启动`emq1`和`emq2`的容器。
```bash
# 下载镜像
docker pull tldzyx/emqttd
# 启动emq1节点($emqttd_img是刚才pull的emqttd镜像的ID，通过docker images查看)
docker run -tid --name emq1 $emqttd_img
# 启动emq2节点
docker run -tid --name emq2 $emqttd_img
```
这样的话我们在docker中已经启动了两个emq节点，现在我们需要讲两个节点连接起来。
首先我们先进入节点emq1中，查看emq1节点的地址。
使用一下命令
```bash
# 进入emq1节点中，相当于ssh登录到到了emq1虚拟机上
sudo docker exec -it emq1 sh 
# 查看节点地址
sudo emqttd_ctl status
# /opt/emqttd $ emqttd_ctl status
# Node '579193a0262d@172.17.0.2' is started
# emqttd 2.3.8 is running

# 退出
exit
```
可以看到emq1节点的地址是`579193a0262d@172.17.0.2`，紧接着我们进入第二个节点emq2中，
```bash
# 登录emq2
sudo docker exec -it emq2 sh
# 寻找emq1，是emq1加入集群
sudo emqttd_ctl cluster join 579193a0262d@172.17.0.2
# 验证是否成功
sudo emqttd_ctl cluster status
# Cluster status: [{running_nodes,['a5ac028fba1e@172.17.0.3',
                                 '579193a0262d@172.17.0.2']}]
```
显而易见我们已经将两个节点加入到同一个集群中了。

## 3.使用Haproxy负载均衡两个节点
首先我们安装Haproxy
```bash
sudo yum install -y haproxy
```
紧接着我们使用利用这个拉取的镜像去构建一个我们自己的镜像。
```bash
mkdir emqtt-haproxy-docker
cd emqtt-haproxy-docker
touch haproxy.cfg
# 修改haproxy.cfg内容如下(将IP地址改成emq1,emq2的ip地址)
# 一些默认参数
defaults
  log                     global
  option                  dontlognull
  option http-server-close
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout connect         10s
  timeout client          1m
  timeout server          1m
  timeout http-keep-alive 10s
  timeout check           10s
# 负载均衡mqtt的tcp接口
frontend emqtt-front
  bind *:1883
  mode tcp
  default_backend emqtt-backend

backend emqtt-backend
   balance roundrobin
   server emq1 172.17.0.2:1883 check
   server emq2 172.17.0.2:1883 check
# 负载均衡mqtt的http管理员界面
frontend emqtt-admin-front
  bind *:18083
  mode http
  default_backend emqtt-admin-backend

backend emqtt-admin-backend
  mode http
  balance roundrobin
  server emq1 172.17.0.2:18083 check
  server emq2 172.17.0.3:18083 check
```
接下来我们使用一个Dockerfile来构建
```bash
touch Dockerfile

# 修改Dockerfile内容如下：
FROM haproxy:latest > Dockerfile
COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg >> Dockerfile

# 使用下面的命令构建新的镜像
docker build -t emqtt-haproxy .
```
这样我们使用`sudo docker ps`可以查看到我们刚才构建好的镜像,即`emqtt-haproxy`
```bash
[root@localhost ~]# docker  images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
emqtt-haproxy       latest              249538bcbccb        12 hours ago        72MB
haproxy             latest              d23194a3929a        4 days ago          72MB
mytomcat            latest              685e1c839eac        2 months ago        720MB
sebp/elk            latest              2fbf0a30426d        2 months ago        1.45GB
centos              7                   75835a67d134        3 months ago        200MB
tldzyx/emqttd       latest              6e7f1fc919fb        8 months ago        78.9MB
boldt/coturn        latest              2c927afe2958        17 months ago       189MB
```
下面来尝试启动代理服务器来负载均衡两个节点
```bash
docker run -it --rm --name haproxy-syntax-check emqtt-haproxy haproxy -c -f /usr/local/etc/haproxy/haproxy.cfg

# 请指定端口映射1833是tcp服务，18083是管理员界面dashboard
docker run -d --name emqtt-running-haproxy -p 1883:1883 -p 18083:18083 emqtt-haproxy
```

大功告成，在浏览器中输入 `http://your-server-ip:18083` ，登陆后即可查看emq集群的运行状态。

![emq_node](/assets/img/2019-01-13-emq-node.png)