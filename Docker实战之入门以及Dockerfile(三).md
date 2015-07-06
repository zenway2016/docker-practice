#Docker实战之入门以及Dockerfile(三) 
[Docker实战之入门以及Dockerfile(一)](https://github.com/billycyzhang/docker-practice/blob/master/Docker%E5%AE%9E%E6%88%98%E4%B9%8B%E5%85%A5%E9%97%A8%E4%BB%A5%E5%8F%8ADockerfile%28%E4%B8%80%29.md)

[Docker实战之入门以及Dockerfile(二)](https://github.com/billycyzhang/docker-practice/blob/master/Docker%E5%AE%9E%E6%88%98%E4%B9%8B%E5%85%A5%E9%97%A8%E4%BB%A5%E5%8F%8ADockerfile%28%E4%BA%8C%29.md)

文章内容，由【[Docker实训课程](https://csphere.cn/training)】[第一讲视频](http://pan.baidu.com/s/1hq2COGc)翻译整理而成

[培训代码](https://github.com/nicescale/docker-training) https://github.com/nicescale/docker-training

[虚拟机镜像](http://market.aliyun.com/products/56014007/jxsc000181.html) http://market.aliyun.com/products/56014007/
##应用镜像

##csphere/wordpress:4.2

```
# cd docker-training/wordpress/
# ls -a
.              license.txt           wp-config-sample.php  wp-login.php
..             readme.html           wp-content            wp-mail.php
Dockerfile     wp-activate.php       wp-cron.php           wp-settings.php
.dockerignore  wp-admin              wp-includes           wp-signup.php
index.php      wp-blog-header.php    wp-links-opml.php     wp-trackback.php
init.sh        wp-comments-post.php  wp-load.php           xmlrpc.php

/docker-training/wordpress# cat Dockerfile 
from csphere/php-fpm:5.4

add init.sh /init.sh

entrypoint ["/init.sh", "/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]
```
使用docker后，在项目代码目录下，写Dockerfile文件，非常方便把项目代码直接打包到docker镜像中，如有哪些文件不想被打包进去，可以在`.dockerignore`文件中定义

Dockerfile解析：

- wordpress镜像是基于csphere/php-fpm:5.4来进行构建
- `ONBUILD`指令生效，把代码文件拷贝到网站根目录下
- `init.sh`脚本对WordPress连接mysql数据库进行配置，固运行wordpress镜像后，只需要进行配置WordPress即可，数据库已准备就绪！

生成WordPress镜像

`docker build -t csphere/wordpress:4.2 .`

查看当前主机本地都有哪些docker镜像

`docker images`

创建WordPress准备

查看主机ip地址

`ifconfig eth0`
192.168.1.20

创建WordPress容器：

`docker run -d -p 80:80 --name wordpress -e WORDPRESS_DB_HOST=192.168.1.20 -e WORDPRESS_DB_USER=admin -e WORDPRESS_DB_PASSWORD=csphere2015 csphere/wordpress:4.2` 
`49d0cddb4e6998a43285fe09165030ba80485065867b9cb8fae9fbdb97cd077f`

参数解析：

- -d 后台运行
- -p 80:80 将宿主机的80端口映射到容器的80端口
- --name wordpress 给容器命名为wordpress
- -e WORDPRESS_DB_HOST=192.168.1.20 数据库主机的ip地址（或者域名）
- -e WORDPRESS_DB_USER=admin 数据库的用户，默认是admin
- -e WORDPRESS_DB_PASSWORD=csphere2015 登陆数据的密码，默认是csphere2015
- csphere/wordpress:4.2使用此镜像创建WordPress容器

访问`http://your_ip`，选择语言，并进行设置`wordpress`

![](https://discuss.csphere.cn/uploads/default/optimized/2X/5/5e887120f584095151915ef2ddbc6c6f6d8e4885_1_422x500.png)

## ENTRYPOINT和CMD的区别
ENTRYPOINT解析
 
定义：

An ENTRYPOINT allows you to configure a container that will run as an executable

运行一个Docker容器像运行一个程序一样

ENTRYPOINT的使用方法：

1.ENTRYPOINT ["executable", "param1", "param2"] (the preferred exec form)
> 推荐使用1方法，启动起来后，pid为1

2.ENTRYPOINT command param1 param2 (shell form) 
>启动起来后，pid号为shell命令执行完的pid号

CMD解析

CMD的使用方法：

1.CMD ["executable","param1","param2"] (exec form, this is the preferred form)
>运行一个可执行的文件并提供参数

2.CMD ["param1","param2"] (as default parameters to ENTRYPOINT) 
>为ENTRYPOINT指定参数

3.CMD command param1 param2 (shell form) 
>是以”/bin/sh -c”的方法执行的命令

实战测试`CMD`

```
vim Dockerfile
FROM centos:centos7.1.1503

CMD ["/bin/echo", "This is test cmd"]
```
生成cmd镜像
`docker build -t csphere/cmd:0.1 .`
生成cmd容器，进行测试
`docker run -it --rm csphere/cmd:0.1`
`This is test cmd`
测试是否可以替换`cmd`的命令
`docker run -it csphere/cmd:0.1 /bin/bash`
`[root@c1963a366319 /]#`

测试结果，在Dockerfile中定义的`CMD`命令,在执行`docker run`的时候，`CMD`命令可以被替换。

实战测试`ENTRYPOINT`
```
FROM centos:centos7.1.1503

ENTRYPOINT ["/bin/echo", "This is test entrypoint"]
```
生成ent(entrypoint)镜像
`docker build -t csphere/ent:0.1 .`

生成ent容器，进行测试
` docker run -it csphere/ent:0.1`
`This is test entrypoint`
测试是否可以替换`entrypoint`的命令
`docker run -it csphere/ent:0.1 /bin/bash`
`This is test entrypoint /bin/bash`

测试结果，在Dockerfile定义的`ENTRYPOINT`命令，通过以上方式不能被替换

实战再次测试`ENTRYPOINT`

`docker run -it --entrypoint=/bin/bash csphere/ent:0.1`
测试结果，ENTRYPOINT命令也可以被替换，需要在执行`docker run`时添加`--entrypoint=`参数，此方法多用来进行调试

##更多精彩内容，访问：[cSphere-希云社区](https://discuss.csphere.cn)

说明，文章由[cSphere-希云](https://csphere.cn)所有，转载请整体转载，并保留原文链接，且不得修改原文！

转载，请联系"cSphere"微信公众号

![](https://discuss.csphere.cn/uploads/default/original/2X/7/72cc34cb366c3c6ae4659bfeb6fc80f4e87be735.jpg)