---
title: Linux安装tomcat
date: 2017-07-14 01:27:44
tags: tomcat
---
### 1、官网下载文件
官方网站下载最新的tomcat：
```
http://tomcat.apache.org/download-80.cgi
```
在ubuntu上，我们下载zip和tar.gz。

### 2、解压压缩文件包
解压tomcat8，用下面的命令（我下载的是tar.gz格式的）：
```
tar -zxvf apache-tomcat-8.5.16.tar.gz
```
### 3、移动到opt路径下
将解压后的文件移动到到 /opt 目录
```
sudo cp -r apache-tomcat-8.5.16 /opt
```
### 4、修改配置文件
进入 /opt/apache-tomcat-8.5.16/bin 目录
```
cd /opt/apache-tomcat-8.5.16/bin
```
编辑下面的startup.sh文件：
```
vim startup.sh
```
在文件中的esac下面添加：
```
JAVA_HOME=/usr/lib/jvm/java-8-oracle
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME
CLASSPATH=.:$JRE_HOME/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
TOMCAT_HOME=/opt/apache-tomcat-8.5.16
```
### 5、启动tomcat
在bin目录下输入：
```
sudo startup.sh
```
即可启动tomcat，此时可以在浏览器中输入：
```
localhost:8080
```
若要关闭tomcat，可以在bin文件夹下输入：
```
sudo shutdown.sh
```
*小提示:*
linux查找JAVA_HOME的路径命令为：
```
echo $JAVA_HOME
```

*本文参考：*
[http://www.cnblogs.com/zhangmingcheng/p/5576499.html](http://www.cnblogs.com/zhangmingcheng/p/5576499.html)
[http://www.cnblogs.com/kerrycode/archive/2015/08/27/4762921.html](http://www.cnblogs.com/kerrycode/archive/2015/08/27/4762921.html)
