小时候爸爸妈妈经常讲故事，开头都是从前有一个人，然后爸爸妈妈说，想听后续故事，明天继续...

我们对Docker有了认识和了解，docker还有非常重要的一块那就是docker镜像，提到docker镜像必不可少的一个文件是**Dockerfile**
docker可以通过build命令自动从Dockerfile文件中读取并执行命令，最终生成docker镜像。使用Dockerfile非常简单，且重要！

# Dockerfile命令详解

写文章时docker版本号为：1.8.1 目前Dockerfile中共包含14个命令，文章内容详细介绍每个一个命令的含义和用法

## Dockerfile

格式：INSTRUCTION arguments

instruction 不区分大小写，推荐用大写，以免和其他参数冲突

## FROM

格式：FROM \<image\>:\<tag\>

Dockerfile中的第一条指令必须是FROM，FROM指令指定了构建镜像的base镜像

## MAINTAINER

格式：MAINTAINER \<name\> \<Email\>

编写Dockerfile的作者信息

## ENV

格式：\<key\>\<value\> or \<key\>\<value\>

为容器声明环境变量，在Dockerfile文件中，可以直接使用环境变量$variable_name

## RUN

格式：RUN \<command\> or RUN ["executable", "param1", "param2"]

RUN命令非常类似Shell命令，通过格式我们明白RUN有2种方式去执行命令，**第一种** 类似/bin/sh -c的方式去执行，**第二种** 不使用shell，exec直接执行，exec命令的参数会看作是JSON数组被解析，所以要用双引号，不能用单引号。

## COPY

格式：COPY \<src\> \<dest\>

拷贝本地\<src\>目录下的文件到镜像中\<dest\>目录。

## ADD

格式：\<src\> \<dest\>

拷贝本地\<src\>目录下的文件到镜像\<dest\>目录。和COPYS非常相似，但比COPY功能多，ADD命令\<src\>除了是一个本地文件，也可以是一个网络文件的URL，并且\<src\>还可以是一个压缩文件，本地文件被拷贝到容器中时会被解压，网络文件不会解压

## EXPOSE


## CMD

格式：CMD \<commadn\> or CMD ["executable", "param1", "param2"] or CMD ["param1", "param2"] 为ENTRYPOINT命令提供参数

CMD命令非常重要，CMD命令是容器启动时加载的命令（启动服务）。在Dockerfile中如果有多个CMD命令，只有最后一个生效。CMD命令参数是一个动态值，信息会保存到镜像的JSON文件中。

## ENTRYPOINT

格式：ENTRYPOINT \<command\> or ENTRYPOINT ["executable", "param1", "param2"]

ENTRYPOINT命令也是容器启动时加载的命令，在Dockerfile中如果有多个ENTRYPOINT命令，只有最后一个生效。ENTRYPOINT命令和CMD命令非常类似，但有区别。如果ENTRYPOINT命令使用\<commmand\>格式，在启动容器的时候ENTRYPOINT会忽略任何CMD指令和docker run 参数，直接运行在/bin/sh -c 中,进程在容器的PID不会是1，而当我们执行docker stop $container_id的时候，进程接收不到SIGTERM信号。如果使用exec格式，docker run 传入的命令参数会覆盖CMD命令，并附加到ENTRYPOINT命令的参数中。推荐使用exec方式

## ONBUILD

格式：ONBUILD [INSTRUCTION]

ONBUILD命令在生成应用镜像时非常重要，一般会在中间件镜像中写一个触发器命令，当构建应用镜像FROM中间件镜像的时候，此触发器命令会在构建应用镜像的时候生效并执行。

<font color="red">显示的字<font>
