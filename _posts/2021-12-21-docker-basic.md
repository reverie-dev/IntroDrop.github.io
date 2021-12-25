---
layout:     post
title:      "docker基础"
subtitle:   "docker"
date:       2021-12-21 01:00:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- cloud native
---
# 镜像和容器的基本操作

**基本操作**

**拉取镜像**

`docker pull`

**运行**

如果我们打算启动里面的 bash 并且进行交互式操作的话，可以执行下面的命令。

`docker run -it --rm {镜像名称} bash`

- it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
- -rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用`-rm`可以避免浪费空间。一般不使用这个命令。
- ubuntu:16.04：这是指用 ubuntu:16.04 镜像为基础来启动容器。
- bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash

后台运行

**列出镜像**

`docker image ls
docker ps`

**停止/重启容器**

`docker stop {镜像名称}
docker restart {镜像名称}`

**进入容器**

`docker exec -it {镜像名称} bash`

只用`-i`参数时，由于没有分配伪终端，界面没有我们熟悉的`Linux`命令提示符，但命令执行结果仍然可以返回。 当`-i -t`参数一起使用时，则可以看到我们熟悉的 `Linux`命令提示符。

**删除容器**

`docker image rm {镜像名称}`

**commit**

对镜像内进行修改后，可以对镜像进行commit

```docker
docker commit \
    --author "reverie" \
    --message "add start.sh" \
    containerId \
    nginx:v2
```