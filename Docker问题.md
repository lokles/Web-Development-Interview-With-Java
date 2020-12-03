# Docker问题

## 1、是什么？

Docker是一个容器化平台，它以容器的形式将您的应用程序及其所有依赖项打包在一起，以确保您的应用程序在任何环境中无缝运行。

## 2、docker与虚拟机的区别？

1. docker不需要像VM一样去模拟计算机硬件环境，
2. 与VM相比，docker中的**镜像只保留核心功能**，如Linux镜像在docker中仅仅有170M。
3. 主机上的**所有容器共享主机的调度程序**，从而节省了额外资源的需求。

## 3、docker原理？

Docker分客户端和服务端概念，Docker服务端有一个**守护线程**以及多个工作线程概念（类似于nginx）。Docker客户端与Docker守护进程通信，**Docker守护进程负责构建，运行和分发Docker容器。**工作线程负责从仓库拉取镜像。

![image-20200820005743507](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200820005743507.png)

## 4、docker镜像是什么？

Docker镜像是Docker容器的源代码，Docker镜像用于创建容器。使用build命令创建镜像。

## 5、docker容器是什么?

Docker容器包括应用程序及其所有依赖项，作为操作系统的独立进程运行。

## 6、docker常用命令?

1. docker pull 从仓库中拉取镜像
2. docker push 将镜像推送到远程仓库
3. docker rm 删除容器
4. docker rmi 删除镜像
5. docker images 列出所有的镜像
6. docker ps 列出所有的容器
7. docker run 运行一个容器