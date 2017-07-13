---
title: Android开发中用到的jar和aar文件的区别
date: 2017-07-14 01:45:15
tags: Android
---
### 前言
 在开发java应用时经常会引用第三方的jar包文件，而在开发Android应用时发现又多出了一个aar结尾的可引用文件。它们之间什么区别呢？

### 一、定义
 jar：Java Application Resource；<br />
 aar：Android Application Resource。
###### 可以看出jar文件是给java应用程序使用的，而aar是专门针对Android应用提供共的引用文件。

### 二、区别
jar：只包含了class文件与清单文件 ，不包含资源文件，如图片等所有res中的文件；<br />
aar：包含jar包和资源文件，如图片等所有res中的文件。

### 三、导入
#### 3.1 jar包的导入 
   - 把工程切换到 **Project** 视图下，在module目录下可以看到 **libs** 目录；
   - 将相应的jar文件拷贝到 **libs**　文件夹中；
   - 更改module的build.gradle配置文件
 ```
 dependencies {
  compile fileTree(include: ['*.jar'], dir: 'libs')
 }
 ```
  在studio新版本中会自动生成，所有你只需直接拷贝.jar到lib目录下编译既可
   - 最后sync一下工程即可（或者rebuild一下工程）
 <br />
 
#### 3.2 aar包的导入 
   - 将*.aar拷贝到app中的lib下；
   - 更改build.gradle 配置文件,在该module的 build.gradle中添加一个本地仓库，并把libs作为仓库地址：
 ```
 repositories {
    flatDir {
       dirs 'libs'
    }
  }
 dependencies {
     compile(name: '你aar文件的名字', ext: 'aar')
 }
 ```
   - 最后sync一下工程即可（或者rebuild一下工程）
   
### 四、导出
 在Android studio中可以同时生成jar文件和aar文件，步骤如下：
 - 新建库，File——New——New Module——Android Library
 - 编译或生成工程，Build——Make Module即可得到相应的文件
 ```
  jar包路径：build\intermediates\bundles\release\class.jar
  arr包路径：build\outputs\aar\包名.aar
 ```
 


### 五、参考
[http://www.aichengxu.com/android/11086139.htm](http://www.aichengxu.com/android/11086139.htm);
[http://jingyan.baidu.com/article/2a13832890d08f074a134ff0.html](http://jingyan.baidu.com/article/2a13832890d08f074a134ff0.html);
[http://www.w2bc.com/article/214269](http://www.w2bc.com/article/214269).

