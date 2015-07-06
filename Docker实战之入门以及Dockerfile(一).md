
#Docker实战之入门以及Dockerfile(一) 
文章内容，由【[Docker实训课程](https://csphere.cn/training)】[第一讲视频](http://pan.baidu.com/s/1hq2COGc)翻译整理而成

[培训代码](https://github.com/nicescale/docker-training) https://github.com/nicescale/docker-training

[虚拟机镜像](http://market.aliyun.com/products/56014007/jxsc000181.html) http://market.aliyun.com/products/56014007/

## 一、Docker是什么？

###首先Docker是软件工业上的集装箱技术

###回顾，在没有集装箱出现以前，传统运输行业中，会存在这些问题：

- 在运输过程中，货物损坏
- 装卸、运输货物，效率低下
- 运输手续繁多及运输环节多
- 劳动强度大，及船舶周转慢

###在集装箱出现后，完全改变了这种状况，是由于集装箱：

- 规则标准化，大大减少了包装费用
- 大大提升了货物装卸效率、及运输效率
- 不同种运输工具之间转换更容易
 
###所以，集装箱出现是传统行业中的一次重大变革

###传统软件行业中存在的问题

- 软件更新发布低效
- 业务无法敏捷
- 环境一致性，难于保证
- 不同环境之间迁移成本太高
- 软件开发商，交付实施周期长---成本高

###有了Docker，以上问题，有望或者说在很大程度上可以得到解决

## 二、Docker的组成

#### Docker是一个C/S架构
![](https://discuss.csphere.cn/uploads/default/optimized/2X/4/4ae4f9f4583f00766f8126e232d0d5e60042a363_1_666x500.jpg)

- Docker Client： Docker的客户端
- Docker Server： Docker daemon的主要组成部分，接收用户通过Docker Client发送的请求，并按照相应的路由规则实现路由分发
- Docker Registry： Registry是Docker镜像的中央存储仓库（pull/push）

> 通过docker pull命令可以把Registry上的docker镜像，下载到服务器本地

> 通过docker push命令可以把服务器本地的docker镜像，上传到Registry上
> 
**Registry在构建自动化平台，起着非常重要的作用！**

![](https://discuss.csphere.cn/uploads/default/optimized/2X/6/6643a54f646047a9dbf31cce424408b4d3fe5ee3_1_666x500.jpg)


_提示：Docker镜像运行之后会成为Docker容器----通过 docker run命令_

####Docker容器启动速度非常快，体现在2个方面；

1.磁盘占用空间小，因为docker镜像采用了分层技术，构建的镜像大小，只有自身的大小，不包含父镜像的大小

2.内存消耗少，docker容器共享的宿主机的内核，没有操作的进程消耗

##Docker实战准备

1. 首先登陆[OSChina Git](http://git.oschina.net)
2. 将[docker-training](http://git.oschina.net/dockerf/docker-training)项目Fork到自己的仓库
3. 使用自己熟悉的SSH工具连接到服务器
4. 执行`git clone https://git.oschina.net/*YOURNAME*/docker-training.git`，将你的远程仓库clone到服务器

[Git 使用指南](http://git.oschina.net/progit/)

后续会构建4个docker镜像，分别为：
> centos7 (基础镜像)

> php-fpm mysql（中间件镜像）

> worpdress(应用镜像)

####什么是Dockerfile？
![](https://discuss.csphere.cn/uploads/default/original/2X/8/8f12e062a5b2c17e8f502b01fc87948ee77453df.jpg)
> Dockerfile是自动构建docker镜像的配置文件，Dockerfile中的命令非常类似linux shell下的命令

> Dockerfile,可以让用户自定义构建docker镜像，支持以 # 开头的注释行

> 一般，Dockerfile分为4部分

- 基础镜像（父镜像）信息
- 维护者信息
- 镜像操作命令
- 容器启动命令

####为何把Dockerfile存放到git仓库中，并为每个项目创建git仓库？

> 方便通过自动化平台，自动构建docker镜像

##三、Dockerfile介绍

###基础镜像csphere/centos:7.1

```
#
# MAINTAINER        Carson,C.J.Zeong <zcy@nicescale.com>
# DOCKER-VERSION    1.6.2
#
# Dockerizing CentOS7: Dockerfile for building CentOS images
#
FROM       centos:centos7.1.1503
MAINTAINER Carson,C.J.Zeong <zcy@nicescale.com>

ENV TZ "Asia/Shanghai"
ENV TERM xterm

ADD aliyun-mirror.repo /etc/yum.repos.d/CentOS-Base.repo
ADD aliyun-epel.repo /etc/yum.repos.d/epel.repo

RUN yum install -y curl wget tar bzip2 unzip vim-enhanced passwd sudo yum-utils hostname net-tools rsync man && \
    yum install -y gcc gcc-c++ git make automake cmake patch logrotate python-devel libpng-devel libjpeg-devel && \
    yum install -y --enablerepo=epel pwgen python-pip && \
    yum clean all

RUN pip install supervisor
ADD supervisord.conf /etc/supervisord.conf

RUN mkdir -p /etc/supervisor.conf.d && \
    mkdir -p /var/log/supervisor

EXPOSE 22

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]
```

`FROM       centos:centos7.1.1503` 
>基于**父镜像**构建其他docker镜像，_父镜像_：可以通过docker pull 命令获得，也可以自己制作

`MAINTAINER Carson,C.J.Zeong <zcy@nicescale.com>`	
>Dockerfile维护者

`ENV TZ "Asia/Shanghai"`
>ENV（environment）设置环境变量，一个Dockerfile中可以写多个。以上例子是：设置docker容器的时区为Shanghai

######Dockerfile中有2条指令可以拷贝文件

`ADD aliyun-mirror.repo /etc/yum.repos.d/CentOS-Base.repo` 
>拷贝本地文件到docker容器里，还可以拷贝URL链接地址下的文件，ADD还具有解压软件包的功能(支持gzip, bzip2 or xz)

`COPY test /mydir` 
>拷贝本地文件到docker容器

`RUN yum install -y curl wget....` 
>RUN命令，非常类似linux下的shell命令 `(the command is run in a shell - /bin/sh -c - shell form)`

> 在Dockerfile中每执行一条指令（ENV、ADD、RUN等命令），都会生成一个docker image layer

`RUN pip install supervisor` 
> supervisor进程管理系统，推荐使用

`ADD supervisord.conf /etc/supervisord.conf`
>添加supervisor的主配置文件，到docker容器里

`RUN mkdir -p /etc/supervisor.conf.d`
> 创建存放启动其他服务"supervisor.conf"的目录，此目录下的所有以.conf结尾的文件，在启动docker容器的时候会被加载

`EXPOSE 22`
> 端口映射 `EXPOSE <host_port>:<container_port>`

> 推荐使用`docker run -p <host_port>:<container_port>` 来固化端口

`ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]`
>一个Dockerfile中只有最后一条`ENTRYPOINT`生效，并且每次启动docker容器，都会执行`ENTRYPOINT`

######以上文件就是用来生成第一个docker镜像的Dockerfile，通过`docker build`指令来生成docker镜像

`docker build -t csphere/centos:7.1 .`
>如果Dockerfile在当前目录下，输入点`.`就可以进行加载当前目录下的`Dockerfile`

>如果不在当前目录下需要运行`docker build -t csphere/centos:7.1 <Dockerfile_dir>`加载相对路径下的`Dockerfile`

docker镜像的命名规则 `registry_url/namespace/image_name:tag` 默认`tag`是`latest`

>在构建Docker镜像时，如果有自己内部的yum源，替换成自己内部的yum源地址，可以加快构建速度。

>如果第一次构建失败，会有部分镜像layer生成，第二次构建会基于第一次构建所生成的layer（use cache），继续构建

```
Step 10 : EXPOSE 22
 ---> Running in 0ed1c5479ebc
 ---> c57a5bac41c8
Removing intermediate container 0ed1c5479ebc
Step 11 : ENTRYPOINT /usr/bin/supervisord -n -c /etc/supervisord.conf
 ---> Running in e16c7ac2fd45
 ---> 185ef7b101a8
Removing intermediate container e16c7ac2fd45
Successfully built 185ef7b101a8
```
可以看到每执行一条`Dockerfile`的指令都会生成一个镜像的layer`c57a5bac41c8` `185ef7b101a8` 最后`185ef7b101a8`这个是docker镜像的ID,`185ef7b101a8`是由`c57a5bac41c8` `185ef7b101a8`...layers叠加而成，体现了docker镜像是分层的
```
# docker images    查看当前主机本地有哪些docker镜像 
REPOSITORY                             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
csphere/centos                         7.1                 185ef7b101a8        40 minutes ago      451.9 MB
```

通过docker镜像生成一个docker容器

`docker help run `  #查看`docker run`命令的使用方法

#####介绍日常工作中经常用到的参数:

`docker run -it` #启动docker容器在前端
`docker run -d`  #启动docker容器在后台

`docker run -p`
`docker run -P`

> 在Dockerfile中有一条指令是EXPOSE 22，如果使用`-P`，宿主机会随机选择一个`没有被使用的端口`和docker`容器的22端口`做`端口映射`,如果docker主机或者容器重启后，宿主机又会随机选择一个没有被使用的端口和docker容器的22端口做端口映射，这样端口会发生`变化`

> 如果使用`-p`,比如`2222:22`，这样不管是docker主机或者容器重启后，2222:22端口都是这样来映射，`不会发生改变` 

生成docker容器

`docker run -d -p 2222:22 --name base csphere/centos:7.1`

`37ac69acf47d501ffc61d8883ae2ba362a132d11e46897212a92b1936e0a0593`

参数说明：

- -d 后台运行
- -it 前台交互式运行
- -P 22 将宿主机的一个未使用的随机端口映射到容器的22端口
- -p 2222:22 将宿主机的2222端口映射到容器的22端口
- --name base 给容器命名为base
- csphere/centos:7.1 使用这个镜像镜像创建docker容器

查看Docker容器

`docker ps `
>`ps`默认只会显示容器在“running”的状态的，容器列表

`docker ps -a`
>`ps -a` 会查看到所有的容器列表

下一篇[Docker实战之入门以及Dockerfile(二)](https://github.com/billycyzhang/docker-practice/blob/master/Docker%E5%AE%9E%E6%88%98%E4%B9%8B%E5%85%A5%E9%97%A8%E4%BB%A5%E5%8F%8ADockerfile%28%E4%BA%8C%29.md)
***
说明，文章由[cSphere-希云](https://csphere.cn)所有，转载请整体转载，并保留原文链接，且不得修改原文！

转载，请联系"cSphere"微信公众号

![](https://discuss.csphere.cn/uploads/default/original/2X/7/72cc34cb366c3c6ae4659bfeb6fc80f4e87be735.jpg)
