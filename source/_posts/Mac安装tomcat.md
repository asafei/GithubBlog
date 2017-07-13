---
title: Mac安装tomcat
date: 2017-07-14 01:26:22
tags: tomcat
---
### 步骤：
- 去tomcat官网下载最新版的tomcat，格式选择tar.gz(当前最新版时9.0.0.M21)；

- 将该压缩文件解压到Library文件夹里，为了方便改名Tomcat9.0.0
注意：Library在Mac中为隐藏文件，可以在finde-前往-前往文件夹中填写：
```
~/Library
```

- 进入Tomcat/bin文件夹下，启动tomcat
```
sudo sh startup.sh
```

- 在浏览器中输入
```
localhost:8080
```
即可看到tomcat的主页
