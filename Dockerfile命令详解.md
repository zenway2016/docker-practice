小时候爸爸妈妈经常讲故事，开头都是从前有一个人，然后爸爸妈妈说，想听后续故事，明天继续...

我们对Docker有了认识和了解，docker还有非常重要的一块那就是docker镜像，提到docker镜像必不可少的一个文件是**Dockerfile**
docker可以通过build命令自动从Dockerfile文件中读取并执行命令，最终生成docker镜像。使用Dockerfile非常简单，且重要！

# Dockerfile命令详解

写文章时docker版本号为：1.8.1 目前Dockerfile中共包含14个命令，文章内容详细介绍每个一个命令的含义和用法

### 格式如下
    INSTRUCTION arguments
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
RUN命令非常类似Shell命令，

## COPY

格式：COPY \<src\> \<dest\>
拷贝本地\<src\>目录下的文件到镜像中\<dest\>目录。

## ADD

格式：\<src\> \<dest\>
拷贝本地\<src\>目录下的文件到镜像\<dest\>目录。和COPYS非常相似，但比COPY功能多，ADD命令\<src\>除了是一个本地文件，也可以是一个网络文件的URL，并且\<src\>还可以是一个压缩文件，本地文件被拷贝到容器中时会被解压，网络文件不会解压

## EXPOSE




