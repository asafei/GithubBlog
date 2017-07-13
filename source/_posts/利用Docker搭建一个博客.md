---
title: 利用Docker搭建一个博客
date: 2017-07-14 02:40:04
tags: docker
---
### 前言
对于docker最早是在天猫双十一之后的一个直播视频中看到了，当时始终觉得这是都是很牛逼的人才玩的转的东西，也就没放在心上。后来公司的同事做了一个docker的分享培训，对docker有了进一步的了解。直到最近分到一个任务——配置一个开源论坛然后部署到公司服务器上，为了避免多次部署的麻烦，就要选择docker，至此铁下心来学习docker。

### 1、docker是干什么的？
作为程序员不知道你有没有这种感觉，每当你要部署一套系统时都要花费大量的时间和精来来配置各种运行环境？明明在自己机器上运行好好的系统，在别人机器上出现了很多问题，然后花费大量时间和精力去调试？很难说这种方式对我们的变成能力能提高多少，那怎么解决这个问题呢？答案就是daocker，它可以让你将你的系统连带运行环境一并“打包分发”。（说的优点low，但是有助于理解）

### 2、docker的安装
要使用docker，肯定是先安装docker了请参考下面的文章, 这里只提醒一点，一定要使用64位的操作系统，不然会遇到很多坑。另外windows、linux和mac都可以安装docker，windows10直接支持docker的安装而之前版本的windows需要通过docker官方提供的工具；mac安装最简单；推荐使用linux环境安装。

### 3、docker的基础
首次接触docker，对所谓的image、container、registry、client、server等等肯定会眼花。本文抛开对这些概念的重复，先让大家动手来感受一下docker能为我们做什么？我们要做一个在tomcat环境下运行的网站，并且支持mysql数据库。

### 4、搭建之旅
#### 4.1 准备工作
首先去 jpress下载一个war文件，作为我们要运行的项目。结下来是不是要搭建java环境、搭建tomcat、搭建mysql数据了？这些全是重复性的劳动，有了docker就可以省去这些繁琐的过程。

#### 4.2 docker环境下安装tomcat
docker按住哪个tomcat环境，分如下几步：
 
 - 搜索tomcat的image；
 - 下载tomcat的image；
 - 运行tomcat的image.
 
在linux通过快捷键 control + alt + t 调出终端，通过下面命令进入root模式：
```
su
```
根据提示输入密码，就进入了root模式，然后启动docker服务：
```
service start docker
```
接着在终端输入下面命令即可执行对tomcat的image的搜索：
```
docker search tomcat
```
就会出现下面的搜索结果：
```
root@afei-virtual-machine:/home/afei# docker search tomcat
NAME                           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
tomcat                         Apache Tomcat is an open source implementa...   1407      [OK]       
tomee                          Apache TomEE is an all-Apache Java EE cert...   38        [OK]       
dordoka/tomcat                 Ubuntu 14.04, Oracle JDK 8 and Tomcat 8 ba...   37                   [OK]
davidcaste/alpine-tomcat       Apache Tomcat 7/8 using Oracle Java 7/8 wi...   19                   [OK]
consol/tomcat-7.0              Tomcat 7.0.57, 8080, "admin/admin"              16                   [OK]
cloudesire/tomcat              Tomcat server, 6/7/8                            15                   [OK]
tutum/tomcat                   Base docker image to run a Tomcat applicat...   7                    
jeanblanchard/tomcat           Minimal Docker image with Apache Tomcat         7                    
andreptb/tomcat                Debian Jessie based image with Apache Tomc...   7                    [OK]
fbrx/tomcat                    Minimal Tomcat image based on Alpine Linux      4                    [OK]
bitnami/tomcat                 Bitnami Tomcat Docker Image                     3                    [OK]
picoded/tomcat                 tomcat 8 with java 8, and MANAGER_USER / M...   2                    [OK]
antoineco/tomcat-mod_cluster   Apache Tomcat with JBoss mod_cluster            1                    [OK]
qasymphony/tomcat              Tomcat images                                   1                    
rstiller/tomcat                Java 1.6, 1.7, 1.8 (OpenJDK & Oracle) for ...   1                    
dreaminsun/tomcat              optimized tomcat                                1                    [OK]
foobot/tomcat                                                                  0                    [OK]
s390x/tomcat                   Apache Tomcat is an open source implementa...   0                    
awscory/tomcat                 tomcat                                          0                    
ppc64le/tomcat                 Apache Tomcat is an open source implementa...   0                    
produtec/tomcat                Oracle JDK and Tomcat                           0                    
techangels/tomcat                                                              0                    
i386/tomcat                    Apache Tomcat is an open source implementa...   0                    
hegand/tomcat                  docker-tomcat                                   0                    [OK]
antoineco/tomcat               Extra OS variants for the official Tomcat ...   0                    [OK]
root@afei-virtual-machine:/home/afei# ^C
```
你可以选择上面的任意一个image进行下载，在此我们下载第一个名为tomcat的image，执行下面的命令：
```
docker pull tomcat
```
然后等待，直到成功的提示出现，我们的tomcat的环境就下载好了，当然里面自动包括java环境了。

#### 4.3 将war文件放进tomcat的image中,构建自己的image
tomcat的image下载好之后，我们就有了tomcat的运行环境了，该如何把自己的war文件放入这个tomcat的运行环境呢？首先我们进入war包所在目录：
```
root@ahui-virtual-machine:/# cd home/ahui/demos
root@ahui-virtual-machine:/home/ahui/demos# ls
jpress.war
root@ahui-virtual-machine:/home/ahui/demos#
```
在当前目录下，新建一个Dockerfile的文件：
```
root@ahui-virtual-machine:/home/ahui/demos# vim Dockerfile
```
在弹窗中按 i键进入编辑模式，输入：
```
from tomcat
MAINTAINER yangyahui afei_ask@163.com
COPY jpress.war /usr/local/tomcat/webapps
~
~
```
然后按esc键，退出编辑模式，输入下面命令退出窗口
```
:wq!
```
**说明一下：**里面的
 - “ from tomcat ” 就是选择刚才我们下载tomcat的image，如果你下载的事其它的image，就换做你下载的那个image 的名称；
 - “ MAINTAINER yangyahui afei_ask@163.com ” 是作者信息；
 - “ COPY jpress.war /usr/local/tomcat/webapps ”意思是将当前目录下的jpress.war文件拷贝到tomcat的image中的/usr/local/tomcat/webapps的目录下，若要查看image中的具体目录位置，可以进入该image进行查看，命令如下：
```
docker exec -it imageID bash
```
最后就可以执行image的构建了，输入下面命令执行image的构建：
```
root@ahui-virtual-machine:/home/ahui/demos# docker build -t jpress .
```
其中的 “ -t jpress ” 表示构建的image的名字为jpress；最后的 “ . ” 表示在当前目录执行构建操作。

完成后，在终端输入一下命令查看image是否构建成功:
```
docker images
```
该命令的意思是查看当前所拥有的image。看到下面有 “ jpress ” 出现，即表示构建成功
```
root@ahui-virtual-machine:/home/ahui/demos# docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
jpress                         latest              d059622def4d        3 hours ago         355MB
```

### 4.4 运行我们自己的image
构建好了自己的jpress的image，如何运行它呢，在终端输入：
```
docker run -d -p 8888:8080 jpress
```
其中 “ -d ” 表示在后台运行；“ -p 8888:8080 ” 表示外部的8888映射image中的8080端口；

此时jpress的image就已经运行了，可以通过下面的命令查看：
```
docker ps
```
该命令表示当前正在运行的container；“ docekr ps -a ” 可以查看所有存在的containers。

这时就可以在浏览器中输入地址 “ loalhost:8888 ” 就可以进入tomcat的默认主页了，然后在浏览器中输入 “ loalhost:8888/jpress ” 就会进入jpress博客的主页，点击下一步，要求我们链接数据库；

**注意：**一个image一旦运行了，就转变成一个container了（如果多次执行运行image的命令就会创建多个不同container，可通过不同的端口区分）；以后再启动该container可以通过下面的命令执行
```
docker start containerID
```
关闭某个运行的container可以通过以下命令执行：
```
docker stop containerID
```
删除一个已有的container可以通过以下命令执行：
```
docker rm containerID
```
而删除一个image的命令是：
```
docker rmi imageID
```

### 4.5 docker 搭建一个mysql数据库的image
在终端输入：
```
docker search mysql
```
然后选择一个image进行下载：
```
docker pull mysql
```
运行这个mysql的image：
```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=000000 -e MYSQL_DATABASE=jpress mysql
```
其中 ： “ -e MYSQL_ROOT_PASSWORD=000000 -e MYSQL_DATABASE=jpress ” 表示环境变量mysql的数据库的密码为00000；创建一个jpress的数据库；

这时我的的mysql环境久安装好了，然后就可以回到 “ loalhost:8888/jpress ”页面 继续jpress博客的数据库的配置情况了；此时需要注意在填写数据库ip时，不能填写 “ localhost ” ，应为mysql的iage的ip不一定和本地ip一致，可以在终端输入 “ ifconfig ” 查看image的ip，然后填写上去。最后对tomcat的image执行重启久完成了：
```
docker restart containerID
```

### 后记
这一篇文章在好几天前都写好了，上传时发生了 “rm –rf ~”的悲剧。损失惨重，就不多说了，在此提示大家一定要常常备份。
