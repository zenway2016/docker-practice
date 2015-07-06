#Docker实战之入门以及Dockerfile(二) 
上一篇[Docker实战之入门以及Dockerfile(一)](https://github.com/billycyzhang/docker-practice/blob/master/Docker%E5%AE%9E%E6%88%98%E4%B9%8B%E5%85%A5%E9%97%A8%E4%BB%A5%E5%8F%8ADockerfile%28%E4%B8%80%29.md)

文章内容，由【[Docker实训课程](https://csphere.cn/training)】[第一讲视频](http://pan.baidu.com/s/1hq2COGc)翻译整理而成

[培训代码](https://github.com/nicescale/docker-training) https://github.com/nicescale/docker-training

[虚拟机镜像](http://market.aliyun.com/products/56014007/jxsc000181.html) http://market.aliyun.com/products/56014007/jxsc000181.html
##中间件镜像

培训代码 https://github.com/nicescale/docker-training

虚拟机镜像 http://market.aliyun.com/products/56014007/jxsc000181.html
##csphere/php-fpm:5.4

```
# cd docker-training/php-fpm/
# ls 
Dockerfile          nginx_nginx.conf  supervisor_nginx.conf
nginx_default.conf  php_www.conf      supervisor_php-fpm.conf
```

各文件解释：

> nginx_nginx.conf 替换默认的nginx.conf文件

> nginx_default.conf 替换默认的default.conf文件

> php_www.conf 修改apache用户为nginx

> supervisor_nginx.conf 添加启动nginx的supervisor文件

> supervisor_php-fpm.conf 添加启动php-fpm的supervisor文件 

```
# cat Dockerfile 
#
# MAINTAINER        Carson,C.J.Zeong <zcy@nicescale.com>
# DOCKER-VERSION    1.6.2
#
# Dockerizing php-fpm: Dockerfile for building php-fpm images
#
FROM       csphere/centos:7.1
MAINTAINER Carson,C.J.Zeong <zcy@nicescale.com>

# Set environment variable
ENV	APP_DIR /app

RUN	yum -y install nginx php-cli php-mysql php-pear php-ldap php-mbstring php-soap php-dom php-gd php-xmlrpc php-fpm php-mcrypt && \ 
	yum clean all

ADD nginx_nginx.conf /etc/nginx/nginx.conf
ADD	nginx_default.conf /etc/nginx/conf.d/default.conf

ADD	php_www.conf /etc/php-fpm.d/www.conf
RUN	sed -i 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/' /etc/php.ini

RUN	mkdir -p /app && echo "<?php phpinfo(); ?>" > ${APP_DIR}/info.php

EXPOSE	80 443

ADD	supervisor_nginx.conf /etc/supervisor.conf.d/nginx.conf
ADD	supervisor_php-fpm.conf /etc/supervisor.conf.d/php-fpm.conf

ONBUILD ADD . /app
ONBUILD RUN chown -R nginx:nginx /app

```

命令解析：

`ONBUILD ADD . /app`
  
`ONBUILD` 在生成当前docker镜像的时候不生效，在子镜像生效；`ONBUILD`在产品发布时起着非常重要的作用！举例

> A镜像中有`ONBUILD`指令，在构建A镜像时`ONBUILD`指令不执行；B镜像`FROM A`,在构建B镜像时`ONBUILD`指令开始执行；

如何给docker镜像命名：

- registry-url: registry服务器的域名或者ip
- namespace: 
- image-name: docker镜像的名字
- tag: docker镜像的版本号，推荐使用应用服务的版本号来命名，如`php-fpm:5.4`

生成php-fpm镜像

`docker build -t csphere/php-fpm:5.4 .`

```
Step 12 : ONBUILD add . /app
 ---> Running in 9e21ede67350
 ---> 7541483a5a76
Removing intermediate container 9e21ede67350
Step 13 : ONBUILD run chown -R nginx:nginx /app
 ---> Running in ab55fc7a46a1
 ---> c61699e8c237
Removing intermediate container ab55fc7a46a1
Successfully built c61699e8c237
```
生成website容器：

`docker run -d -p 8080:80 --name website csphere/php-fpm:5.4`
`da30b15d3518320f4150b20ef329e59432a65610968977277879578b5fd8f4f7`

参数解释：

- -d 后台运行
- -p 8080:80 将宿主机的8080端口映射到容器的80端口
- --name website  给容器命名为website
- csphere/php-fpm:5.4 使用这个镜像镜像创建docker容器

使用浏览器访问：`http://your_ip:8080/info.php`

![](https://discuss.csphere.cn/uploads/default/original/2X/c/c002db15d4ed47329927a97066d485bce874ccfb.png)

如何进入一个正在运行的docker容器？

`docker exec -it website /bin/bash`

```
# supervisorctl    查看当前容器中使用supervisor启动了哪些服务 
nginx                            RUNNING   pid 9, uptime 0:23:15
php-fpm                          RUNNING   pid 10, uptime 0:23:15
```

##csphere/mysql:5.5

```
cat Dockerfile 
#
# MAINTAINER        Carson,C.J.Zeong <zcy@nicescale.com>
# DOCKER-VERSION    1.6.2
#
# Dockerizing Mariadb: Dockerfile for building Mariadb images
#
FROM csphere/centos:7.1
MAINTAINER Carson,C.J.Zeong <zcy@nicescale.com>

ENV DATA_DIR /var/lib/mysql

# Install Mariadb
RUN yum install -y mariadb mariadb-server && \
    yum clean all

ADD mysqld_charset.cnf /etc/my.cnf.d/

COPY scripts /scripts
RUN chmod +x /scripts/start

EXPOSE 3306

VOLUME ["/var/lib/mysql"]

ENTRYPOINT ["/scripts/start"]
```

命令解析：

`VOLUME ["/var/lib/mysql"]`
> `VOLUME`指令，宿主机文件目录和docker容器文件目录做映射

`ENTRYPOINT ["/scripts/start"]`
>`ENTRYPOINT`在每次启动docker容器时都会被执行，此例，是运行了一个shell脚本"/scripts/start"

每次启动都会运行`/scripts/start`脚本，脚本内容如下：

```
# cat start 
#!/bin/bash

set -e

#
# When Startup Container script
#

if [[ -e /scripts/firstrun ]]; then
	# config mariadb
	/scripts/firstrun_maria
    	rm /scripts/firstrun
else
	# Cleanup previous mariadb sockets
	if [[ -e ${DATA_DIR}/mysql.sock ]]; then
		rm -f ${DATA_DIR}/mysql.sock
	fi
fi

exec /usr/bin/mysqld_safe
```

脚本解析：

- `set -e` 脚本中只要有一行有错误，就会中断脚本执行

- 如果firstrun文件存在，执行firstrun_maria脚本，如果不存在，删除mysql.sock文件，并启动Mariadb

> firstrun_maira脚本是初始化Mariadb，以及设置数据库用户和密码，详情内容请自行阅读[脚本文件](http://git.oschina.net/dockerf/docker-training/blob/master/mysql/scripts/firstrun_maria?dir=0&filepath=mysql%2Fscripts%2Ffirstrun_maria&oid=788bfb61d8cc45a33b60cde5a0e98899ee08f808&sha=3a86a4767292c267af0794628efb76fe31e754e6)

构建mysql docker镜像

`docker build -t csphere/mysql:5.5 .`

###docker volume 保证删除容器后，数据不被删除

- 保存容器中的数据
- 数据共享

使用方法：

1.在Dockerfile中定义VOLUME["/data"]

2.通过`docker run -d -v <host_dir>:<container_dir>`

###案例：

1. 创建mysql容器，不挂载docker volume，删除后，数据是否存在
2. 创建mysql容器，挂载docker volume，删除后，数据是否存在

运行不挂载docker volume的mysql容器

`# docker run -d -p 3306:3306 --name dbserver csphere/mysql:5.5`
`0a3092395c1e6a84f0ecd5383799f210519c5aefc82cbb7ee2ed1a471fc463f5`

删除docker容器，容器里面的数据都会随着容器被删除而删除

```
# docker rm dbserver
Error response from daemon: Cannot destroy container dbserver: Conflict, You cannot remove a running container. Stop the container before attempting removal or use -f
Error: failed to remove containers: [dbserver]
```
参数解释：

- `docker rm` 删除状态为“Exited”的docker容器
- `docker rm -f` 强制删除docker容器

运行挂载docker volume的mysql容器

`docker run -d -p 3306:3306 -v /var/lib/docker/vfs/dir/mydata:/var/lib/mysql csphere/mysql:5.5`
`f49165d5e081b8bd8af9cb9c0bbbeb6545d45f857c1a852646c105`
`docker exec -it f49 /bin/bash`

登陆数据库创建mydb数据库

```
# mysql
# show databases;
# create database mydb;
Query OK, 1 row affected (0.00 sec)
# show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)
# exit
exit
```
 
查看主机文件目录下，是否已生成mydb数据库目录文件
```
# ls /var/lib/docker/vfs/dir/mydata/
aria_log.00000001  ibdata1      ib_logfile1  mysql       performance_schema
aria_log_control   ib_logfile0  mydb         mysql.sock  test
```

```
停止docker容器
# docker stop f49165d5e081
f49165d5e081
删除docker容器，查看`mydb`目录是否被删除
# docker rm f49165d5e081
f49165d5e081
# ls /var/lib/docker/vfs/dir/mydata/    验证，挂载docker volume后，容器被删除掉，数据还在
aria_log.00000001  ibdata1      ib_logfile1  mysql       performance_schema
aria_log_control   ib_logfile0  mydb         mysql.sock  test
```
新创建一个容器，挂载到刚才的数据目录下，是否可以把之前的数据库加载回来

`docker run -d -p 3306:3306 --name newdb -v /var/lib/docker/vfs/dir/mydata:/var/lib/mysql csphere/mysql:5.5`
`29418b93d4d4a00a86169c568b6f952e71d25b155d7f6b8012d953022691b2b8`

`docker exec -it newdb /bin/bash`
```
# mysql
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)
```

验证结果: 只要保证数据在，重新创建一个容器挂载回之前的数据目录，业务即可恢复（容器可随意删除、创建）

下一篇[Docker实战之入门以及Dockerfile(三)](https://github.com/billycyzhang/docker-practice/blob/master/Docker%E5%AE%9E%E6%88%98%E4%B9%8B%E5%85%A5%E9%97%A8%E4%BB%A5%E5%8F%8ADockerfile%28%E4%B8%89%29.md)
***
说明，文章由[cSphere-希云](https://csphere.cn)所有，转载请整体转载，并保留原文链接，且不得修改原文！

转载，请联系"cSphere"微信公众号

![](https://discuss.csphere.cn/uploads/default/original/2X/7/72cc34cb366c3c6ae4659bfeb6fc80f4e87be735.jpg)