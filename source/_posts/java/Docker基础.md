---
 title: 什么是Docker？
categories:
- 好好学习
tags:
  - Docker
date: 2019-08-08 21:19:46
---


## 什么是Docker？

Docker的思想来自于集装箱

云计算就好比大货轮。docker就是集装箱。

docker就是用来存放应用的一个容器。

![](https://www.runoob.com/wp-content/uploads/2016/04/docker01.png)

<!-- more -->

说实话关于Docker是什么并太好说，下面我通过四点向你说明Docker到底是个什么东西。

- **Docker 是世界领先的软件容器平台。**
- **Docker** 使用 Google 公司推出的 **Go 语言**  进行开发实现，基于 **Linux 内核** 的cgroup，namespace，以及AUFS类的**UnionFS**等技术，**对进程进行封装隔离，属于操作系统层面的虚拟化技术。** 由于隔离的进程独立于宿主和其它的隔离的进 程，因此也称其为容器。**Docke最初实现是基于 LXC.**
- **Docker 能够自动执行重复性任务，例如搭建和配置开发环境，从而解放了开发人员以便他们专注在真正重要的事情上：构建杰出的软件。**
- **用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。**




## 为什么要使用Docker？

运维在把你的软件从开发环境转移到生产环境的时候就会遇到一些Ubuntu转centos的问题

docker你就可以把开发环境直接封装转移给运维

在服务器负载方面，如果你单独开一个虚拟机，那么虚拟机会占用空闲内存的，docker部署的话，这些内存就会利用起来。

## 图解物理机、虚拟机与容器

关于虚拟机与容器的对比在后面会详细介绍到，这里只是通过网上的图片加深大家对于物理机、虚拟机与容器这三者的理解。

| 物理机                                                       | 虚拟机                                                       | 容器                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![物理机](https://user-gold-cdn.xitu.io/2018/6/18/1641129f0ecdf8ff?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) | ![虚拟机](https://user-gold-cdn.xitu.io/2018/6/18/164112a72a917f4a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) | ![容器](https://user-gold-cdn.xitu.io/2018/6/18/164112ac76e6f693?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) |

通过上面这三张抽象图，我们可以大概可以通过类比概括出： **容器虚拟化的是操作系统而不是硬件，容器之间是共享同一套操作系统资源的。虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统。因此容器的隔离级别会稍低一些。**

## Docker相关概念

| Docker相关名词 | 解释                                                         |
| -------------- | ------------------------------------------------------------ |
| container      | 容器，是指image的运行时，包含了文件资源（image展开）和系统资源（变成process存在于系统中） |
| image          | 镜像，是指将应用打包好之后的存储方式，一个image包含多层layer |
| Repository     | 仓库，集中存放镜像的地方                                     |
| layer          | 在Dockerfile中每一步都会产生一层layer，每一步的结果产出变成文件 |
| Dockerfile     | 一种构建image的文件的DSL                                     |
| docker         | 可以通过Dockerfile构建image，也可以将image运行，使其变成container |
|                |                                                              |
| moby           | docker项目的新名字，docker公司的一步棋                       |
| docker-compose | Python写的一个docker编排工具                                 |
| docker swarm   | docker公司推出的容器调度平台                                 |
| kubernetes     | google主导的容器调度平台                                     |
| 容器调度平台   | 一般来说是由M个master和N个worker节点组成的一个集群，上面可以整合宿主机资源，完成网络、存储、CPU、内存等资源的管理，将容器运行在不同的主机上，已达成“人多力量大”，“大力出奇迹”和“各种骚操作”的目的 |

![](http://stepimagewm.how2j.cn/9104.png)

## 如何安装使用Docker

### 安装运行Docker

1. 更新yum

   ```shell
   yum -y update
   ```

2. 安装 Docker

   ```shell
   yum install docker -y
   ```

3. 启动 Docker

   ```shell
   systemctl start docker.service # 启动
   systemctl status docker.service # 查看docker状态
   ```

4. docker 的生命周期管理常用命令

   ```shell
   systemctl stop docker.service 
   systemctl start docker.service
   systemctl restart docker.service
   systemctl status docker.service
   ```

### Docker镜像管理

镜像管理常见的有这么些：
1. search 查看仓库里有些什么镜像
2. pull 拉取镜像
3. images 查看本地有些什么镜像
4. rmi 删除本地镜像
5. 修改本地镜像名称
6. push , 把镜像提交到仓库

用docker跑一个tomcat举例

1. 首先通过关键字从仓库搜索tomcat

   ```
   docker search tomcat
   ```

2. 从仓库中拉取一个tomcat的镜像

   ```
   docker pull tomcat:8.0
   ```

3. 启动容器

   ```
   docker run -dit --privileged  -p80:8080  --name mytomcat docker.io/tomcat:8.0 
   ```

| 参数         | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| docker run   | 表示运行一个镜像                                             |
| -dit         | 是 -d -i -t 的缩写。 -d ，表示 detach，即在后台运行。 -i 表示提供交互接口，这样才可以通过 docker 和 跑起来的操作系统交互。 -t 表示提供一个 tty (伪终端)，与 -i 配合就可以通过 ssh 工具连接到 这个容器里面去了 |
| --privileged | 启动容器的时候，把权限带进去。 这样才可以在容器里进行完整的操作 |
| -p 80:8080   | 第一个80，表示在CentOS 上开放80端口。 第二个8080 表示在容器里开放8080 端口。 这样当访问CentOS 的80端口的时候，就会间接地访问到容器里了 |
| --rm         | 表示如果容器已经存在了，自动删除容器                         |

​	4. 进入docker容器

```
docker exec -it mytomcat /bin/bash
```

### Docker容器管理

 

| 命令                        | 解释                                             |
| --------------------------- | ------------------------------------------------ |
| run                         | 运行                                             |
| exec attach                 | 进入                                             |
| pause, unpause, stop, start | 生命周期管理， 暂停，恢复，停止，启动            |
| ps                          | 查看所有的容器                                   |
| inspect                     | 检查某个具体的容器                               |
| rm                          | 删除容器                                         |
| commit                      | 对容器做了修改后，把改动后的容器，再次转换为镜像 |

![](http://stepimagewm.how2j.cn/9116.png)

## 参考资料

如何通俗解释Docker是什么？（https://www.zhihu.com/question/28300645）

可能是把Docker的概念讲的最清楚的一篇文章（https://juejin.im/post/5b260ec26fb9a00e8e4b031a）