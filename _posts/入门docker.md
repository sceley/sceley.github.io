---
title: 入门docker
date: 2018-03-29 20:46:00
tags: docker
---

## 序言

如果让我用一句话来形容docker的话。它简单易用，只是简单的操作拉取镜像和创建容器，然后你可以使用mysql、mongodb、redis、nginx等各类服务了。

<!--more-->

## 安装

- [Mac](https://docs.docker.com/docker-for-mac/install/)
- [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Windows](https://docs.docker.com/docker-for-windows/install/)

## 命令

查看Docker的版本

```shell
docker version
```

Docker 需要用户具有 sudo 权限，为了避免每次命令都输入sudo，可以把用户加入 Docker 用户组

```shell
sudo usermod -aG docker $USER
```

启动docker

```shell
sudo service docker start
```

下载镜像

```shell
docker pull [imageName]
```

删除image镜像

```shell
sudo docker image rm [imageName]
```

列出docker镜像

```shell
docker image ls
```

生成容器	

```shell
docker container run -p 3306:3306 --rm=true --name=mysql -v ./mysql:/etc/mysql -it -d mysql /bin/bash
```

参数:
	- -p容器的 3306 端口映射到本机的 3306 端口。
	- -it 容器的 Shell 映射到当前的 Shell，然后你在本机窗口输入的命令，就会传入容器。
	- mysql 镜像的名称
	- /bin/bash：容器启动以后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell。
	- -v 挂载本地目录到容器目录
	- --rm=true 容器停止运行后，自动删除容器文件x
	- --name=** 给容器命名
	- -d 在后台运行

列出容器

```shell
docker container ls (--all)
```

删除容器
```shell
docker container rm [containerID/containerName]
```

启动容器(前面的docker container run命令是新建容器，每运行一次，就会新建一个容器。)

```shell
docker container start [containerID]
```

终止容器运行
```shell
docker container stop
```

进入一个正在运行的 docker 容器

```shell
docker container exec -it [containerID] /bin/bash
```

用于从正在运行的 Docker 容器里面，将文件拷贝到本机
```shell
docker container cp [containID]:[/path/to/file] .
```