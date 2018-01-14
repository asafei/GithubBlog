---
title: Android之启动模式
date: 2018-01-14 19:35:00
tags: Android
---
### 一、Android中启动模式的设置
1、可以在AndroidManifest.xml中给activity的标签指定`android:launchMode`属性来选择启动模式；

```xml
<activity
    android:name=".FirstActivity"
    android:launchMode="singleTask"
    android:label="标题">
<activity>
```
2、也可以在代码上进行修改

### 二、不同启动模式的区别
* standard:默认方式，每次启动都会创建新的实例；
* singleTop:启动时如果栈顶就是，则直接调用栈顶的activity；
* singleTask:每次启动时都检查栈内是否已存在，存在则将该activity之上的activity全部清除，然后调用该activity；
* singleInstance:启用一个新的返回栈来管理这个活动。（可以解决共享实例问题）

