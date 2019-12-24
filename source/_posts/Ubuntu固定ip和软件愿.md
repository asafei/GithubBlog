---
title: Ubuntu固定ip和软件愿
date: 2019-12-24 22:41:39
tags: linux
---
## 一、固定ip
### 1.1 设置静态ip
在`compute-allsetting-network`中选择`wired`在右下角`options`中选择`IPv4 Setting`中
* Method中选择Manual；
* Address中点击右侧add按钮，在里面添加Address、Netmask、Geteway；
* DNS servers中添加相应的地址。
其中dns servers的地址可以在任意一台电脑上cmd中输入`ipconfig -all`的本地连接中查看，可以有多个。
可以参考：[ubuntu静态IP教程](https://jingyan.baidu.com/article/b7001fe18f85fe0e7282ddaf.html)

### 1.2 关闭防火墙
如果是虚拟机的话，首先将网络连接方式设为`桥接模式(B)：直接连接物理网络`，并且勾选`复制物理网络连接状态(p)`。
接下来就需要命令行了，首先进入管理员模式，
> 1. 关闭防火墙：
> ```
> ufw disable
> ```
> 
> 2. 开启防火墙
> ```
> ufw enable
> ```
一般这样就可以了，可以ping一下其它地址试一试，不行的话就重启一下系统。

## 二、更换软件源
软件源文件位于`/etc/apt/sources.list`,
2.1 最好先做一下备份
```
cd /etc/apt

cp sources.list sources.list.bak
```
2.1 然后编辑sources.list
```
vi sources.list sources.list
```
2.3 然后替换里面的地址，可以使用批量替换的命令
`:%s#旧字符#新字符#g`
其中g表示全部替换.
2.4 最后更新一下软件源：
```bash
sudo apt-get update

sudo apt-get upgrade
```
