---
title: hexo创建github博客
date: 2017-07-14 01:19:52
tags: hexo
---
### 初识Hexo
在之前接触github的时候就听说过hexo这个东西了，感觉挺高端同时心里面也有一些不自信，就一直没接触。直到在公司由同事再一次推荐hexo，拖到现在才着手。还是那一句话，对于一下好的工具，作为程序员不应该排斥，工具都是为了更好的提高作业效率的，不应心生畏惧，甩开膀子，尽管上吧！


### 一、github博客repository的创建
  * 账号登陆github；
  * create a new respositiry，名字一定要设为“yourname+github+io”;
  * 至于秘钥，可设可不设，为了简单此处不设置。
 
### 二、hexo的安装
#### 2.1 hexo的安装
##### 2.1.1 命令行输入：
```
$ nmp install hexo -g
```
关于npm的安装将会有单独的一章进行介绍；这里说一下后缀的-g表示全局的意思；
##### 2.1.2 安装好之后，可以初始化一个文件夹了，命名为blog_hexo，依次输入命令：
```
$ hexo init blog_hexo
$ cd blog_hexo
$ npm install
$ hexo server
```
其中第一行目的是初始化一个文件夹blog_hexo，里面将存放一些配置型的文件；
第4行的目的是启动该启动服务，可以在浏览器输入http://localhost:4000 访问初始化的hexo博客了；

此时，我们的blog_hexo文件夹下有如下文件列表：

 - _config_yml // 注配置文件
 - db.json // 数据
 - debug.log // 调试日志
 - _node_mudules // nodejs 相关依赖
 - package.json // 配置依赖
 - scaffolds // 脚手架 - 也就是一个工具模板
 - source // 存放blog正文的 地方
 - themes // 存放皮肤的地方

注意：此时可能有些同学的页面打不开，原因可能是网络访问权限的问题，hexo默认使用google的提供的数据，因此需要做一些修改：

（1）进入blog_hexo/themes/landscape/layout/_partial 目录下，找到after-footer.ejs文件，将里面的:
```
<script src="//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"> </script>
```
修改为：
```
<script src="http://cdn.bootcss.com/jquery/2.1.1/jquery.min.js“ > </script>
```

（2）找到header.ejs文件，删除里面下行：
```
<link href="//fonts.googleapis.com/css?family=Source+Code+Pro" rel=”stylesheet” type=”text/css”>
```
再次执行：
```
hexo server
```
就可以访问 http://localhost:4000 就可以正常的打开了。
此时本地的server已经部署完毕；
下面看一下如何添加新的文章：

### 三、新建文章
思路：自己编写md文件，通过hexo实现静态化，就可以自动生成相应的html页面了。

#### 3.1 编写md文件
回到blog_hexo目录下，新建主题为hellohexo的blog：
```
hexo new hellohexo
```
此时会在source/_posts/下新建hellohexo.md的文件，然后就可以对该文件进行相应的文字编写工作了，采用的是markdown格式（markdown语法会有单独篇章总结）。

#### 3.2 静态化
因为最终要将blog部署在github静态服务器上，所以需要静页面进行静态化。回到blog_hexo跟目录，执行：
```
$ hexo g
```
经过一些提示，就在blog_hexo/public文件夹下面生成了相应的静态文件。

### 四、将blog部署到github上
#### 4.1 修改_config.yml文件
因为hexo已经集成好了发布到github的配置，所以只需要修改一下blog_hexo根目录下面的
_config.yml文件即可，将里面最下面的内容：
```
deploy:
  type: github
  repo:
```
修改成自己的repo
```
deploy:
  type: git
  repo: https://github.com/asafei/asafei.github.io.git
  branch: master
```
注意：每个冒号 “：”一定需要跟一个空格。
#### 4.2 回到blog_hexo根目录，执行
```
npm install hexo-deployer-git --save //没有这句会报错
hexo deploy
```
就会发现将public目录下的页面发布到github gh_pages分支上了。
#### 4.3 访问博客
在浏览器输入
```
asafei.github.io
```
就可以看到自己的博客了。

此时利用hexo来搭建github的博客已经可以了，其余的关于域名指向、插件安装等内容暂时先不讲。

**本文参考：**
[http://www.netpi.me/实用/hexo/](http://www.netpi.me/实用/hexo/)
[http://www.jianshu.com/p/e99ed60390a8](http://www.jianshu.com/p/e99ed60390a8)
