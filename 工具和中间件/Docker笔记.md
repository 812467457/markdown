# Docker

## 一、简介

Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、 [bare metal](http://www.whatis.com.cn/word_5275.htm)、OpenStack 集群和其他的基础应用平台。

Docker是一容器虚拟化技术。



## 二、Docker的组成

![image-20200811165816275](http://picture.youyouluming.cn/image-20200811165816275.png)

* 镜像（image）：是一个只读的模板。镜像可以用来创建Docker容器，一个镜像可以创建多个容器。
* 容器（Container）：Docker利用容器独立运行一个或一组应用。运行是用镜像创建的运行实例。容器可以被启动、开始、停止、删除。容器可以看作是一个简易版的Linux环境，包括环境配置和应用。
* 仓库（repository）：是集中存放镜像文件的场所，仓库和仓库注册服务是有区别的，仓库注册服务存在多个仓库，每个仓库又包含了多个镜像，每个镜像都有不同的标签。仓库分公有私有。

## 三、Docker安装

### 1、需要的软件包

> yum install -y yum-utils device-mapper-persistent-data lvm2

### 2、配置阿里云docker镜像下载

> yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

### 3、更新yum软件包索引

> yum makecache fast

### 4、查看本机可用版本

> yum list docker-ce --showduplicates | sort -r

### 5、安装docker

> sudo yum install docker-ce-17.12.0.ce-1.el7.centos

### 6、启动服务

> systemctl start docker
>
> 查看是启动状态
>
> systemctl status docker
>
> 测试
>
> docker run hello-world

### 7、阿里云镜像加速器

* 创建目录：`mkdir -p /etc/docker`
* 创建加速器的文件：`vim /etc/docker/daemon.json`
* 配置daemon文件：`{  "registry-mirrors": ["https://自己阿里云中的配置.mirror.aliyuncs.com"] }`
* 刷新配置文件：`systemctl daemon-reload`
* 重启Docker：`systemctl restart docker`



## 四、Docker常用命令

### 1、帮助命令

* 查看版本信息：`docker version`
* 查看docker信息：`docker info`
* 帮助文档：`docker help`

### 2、镜像命令

* 查看本地镜像信息：`docker images [options]` ，OPTIONS，`-a`列出本地所有镜像、`-q`只显示镜像id、`--digest`显示镜像的摘要信息、`--no-trunc`显示完整镜像信息
* 搜索镜像：`docker search [options] xxx镜像` ，options，`-s`筛选指定点赞数大于多少
* 下载镜像：`docker pull 需要下载的镜像 [可以指定版本，默认最新版]`，先下载一个centos
* 删除镜像：`docker rmi -f 要删除的镜像ID`，删除多个`docker rmi -f 要删除的镜像ID 要删除的镜像ID`，删除全部`docker rmi -f $(docker.images -qa)`

### 3、容器命令

* 新建并启动容器：`docker run [options] `  ，options{`-name`=可以指定容器名字，`-d`=后台运行，启动守护进程，`-i`=以交互式模式运行，`-t`=为容器分配一个伪输入终端经常和-i一起用，`-P`=随机端口映射，`-p`=指定端口映射(主机端口号:容器端口号)}
* 查询正在运行的容器：`docker ps [options]`，options{`-a`=列出正在运行和历史上运行过的，`-l`=显示最近创建的容器，`-n`=显示最近n个创建的容器，`-q`=静默模式只显示编号}
* 退出容器：容器停止退出=`exit` 和容器不停止退出=CTRL+P+Q
* 启动容器：`docker start 容器id`
* 重启容器：`docker resatrt 容器id`
* 停止容器：`docker stop 容器id`
* 强制停止：`docker kill 容器id`
* 删除容器：`docker rm 容器id`，删除所有`docker rm -f $(docker ps -a -q)`

### 4、容器其他命令

* 后台启用容器：使用`docker run -d 容器名`，会有一个情况就是，这个容器启动后没有使用，就会被自动退出。在启动时使用一个死循环输出日志`docker run -d centos /bin/sh -c "while true;do echo hello zzyy;sleep 2;done"`，这样就会在一启动就使用容器，不会被自动退出。
* 查看日志：`docker logs [options] 容器id`，options{`-t`=是否加入时间戳，`-f`=跟随最新日志打印，`--tail`=数字显示最后多少条}
* 查看容器内的进程：`docker top 容器id`
* 进入正在运行的容器并以命令行交互：`docker exec -it 容器id 需要执行的命令`，切换到该容器的命令方式`docker exec -it e93d4c808e50 /bin/bash`
* 重新进入容器：`docker attach 容器id`，不会启动新的进程

