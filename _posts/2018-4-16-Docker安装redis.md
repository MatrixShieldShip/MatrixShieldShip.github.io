---
title: 从头开始使用Windows版Docker安装redis
description: Windows版Docker的基础使用
categories:
 - Docker
tags:
---
# 安装Docker

直接去Docker官网下载一个Windows版本的Docker，安装就行。然后需要注册一个Docker的账号。

# 部署阿里云加速

当我们下载好Docker后，由于国内的网络特点，建议部署阿里云加速，这样每次我们下载镜像都会从阿里云下载，否则会访问Docker官网，速度可能较慢。可以参考[这里](https://yq.aliyun.com/articles/29941)。在阿里控制台完成账号注册之后，访问镜像加速器会获得一个专属加速器地址，类似``https://abcd1234.mirror.aliyuncs.com``。接下来的步骤该博文中是在Linux下的情况，在windows下我们可以使用图形界面来更方便的进行配置。对着Docker图标点击右键，进入``settings - Daemon``，在``Registry mirrors``中添加该地址。如图所示。

# 通过redis官方镜像来安装redis

redis提供了docker的镜像，我们可以通过直接下载该镜像然后派生实例的方式来快速创建redis实例。

## 下载redis镜像

打开powershell，输入
``` powershell
docker pull redis
```

当返回的结果最后一行是``Status: Downloaded newer image for redis:latest``代表已经下载好了最新的镜像，接着我们就可以部署了。

## 从镜像派生实例

Docker的一个镜像可以看作一个系统的一个快照，可以通过其加载出无数实例。因此，我们往往需要指定每个实例的运行端口和名称以方便我们以后寻找该实例。


    docker run -d -p 6379:6379 --name redis01 redis

``-d``表示在后台运行，不阻塞命令行界面，让我们可继续输入其它命令，是detach单词缩写。

``-p`` 表示端口号，左边的6379表示win10系统的端口（自已换其它的也随便），右边的则表表容器中redis端口。

``--name``表示运行redis镜像的一个实例名称。

``redis``则是镜像名称，我们刚刚已经下载了该镜像。

---

我们还可以再通过该镜像启动一个实例，这次使用6380端口。

    docker run -d -p 6380:6379 --name redis02 redis

使用``docker ps``命令可以查看当前已经启动的docker实例。

## 其他命令

停止实例：``docker stop [Name]``

删除实例：``docker rm [Name]``

进入实例：

``docker attach [Name]``，当执行``exit``退出时可能会使容器一起退出

``docker exec -it [Name] /bin/bash``

## 在过程中可能出现的错误

在windows下使用镜像时，有时会出现打不开端口的错误。

当使用``docker run -d -p 6379:6379 --name redis01 redis``时，容器并没有成功被创建，报错信息如下：

```
8d2aa5881e338bca1c46a126298ea1b974c3687db825470c03c1951aba0e7ee9
C:\Program Files\Docker\Docker\Resources\bin\docker.exe: Error response from daemon: driver failed programming external connectivity on endpoint redis01 (39a051934c9650a9fedc3c08b81fa25342116c4d6bf0193d7ce94a998c949988): Error starting userland proxy: mkdir /port/tcp:0.0.0.0:6379:tcp:172.17.0.2:6379: input/output error.
```

如果再次运行同样的命令，会出现：

```
C:\Program Files\Docker\Docker\Resources\bin\docker.exe: Error response from daemon: Conflict. The container name "/redis01" is already in use by container "0daf4e146caca44468a3d1e4f6c795bc834b2bc28a2098bdf1dea59feaf1abbd". You have to remove (or rename) that container to be able to reuse that name.
```

这是因为其实Docker已经创建了名为redis01的实例，不能重复创建。可以通过前面提过的docker命令来查看镜像``docker ps -a``

要处理这个问题，可以参考[StackOverFlow](https://stackoverflow.com/questions/40668908/running-docker-for-windows-error-when-exposing-ports/40727613#40727613)。我的做法如下：首先需要删除redis01:

    docker rm redis01

然后直接重启Docker问题就解决了。

如果问题没有解决，再试一试通过命令重启操作系统。

    shutdown /s /t 0

因为win10在重启后与自动恢复重启前的程序状态，因此必须使用命令来进行系统重启。

# 使用工具连接redis

redis01地址：127.0.0.1：6379

redis02地址：127.0.0.1:6380

