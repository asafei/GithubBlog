---
title: Android数据持久化---SharedPreferences
date: 2018-01-24 20:56:05
tags: Android
---
SharedPreferences是一种比文件更加简单的存储方式，它以键值对的形式将数据存储在`/data/data/<package-name>/shared_prefs/`文件夹下。可以存储整形、字符串或布尔型的数据。

###  一、存储数据到SharedPreferences中

网SharedPreferences中存储数据非常简单，分三步：

1. 获取SharedPreferences对象sp；
2. 通过sp获取SharedPreferences.Editor对象spe；
3. 通过spe添加数据
4. 通过spe的apply方法完成提交



#### 1.1 获取SharedPreferences对象 

获取SharedPreferences对象的方法有三种：

（1）Context类中的`getSharedPreference()`方法

>该方法中接收两个参数：
>
>* 第一个参数：指定SharedPreferences文件的名称，该文件存储在`/data/data/<package-name>/shared_prefs/`文件夹下；
>
>* 第二个参数：指定操作模式，目前只有MODE_PRIVATE方法可选，和传入0
>
>  效果相同，表示只有当前应用程序才可以使用这个文件进行读写。其它模式如：MODE_WORLD_READABLE和MODE_WORLD_WRITEABLE在Android4.2中已废弃，MODE_MULTI_PROCESS在Android6.0中已废弃；

（2）Activity中的`getPreference()`方法

> 和Context类中的`getSharedPreference()`方法类似，只接受一个操作模式作为参数，这种方式会自动将当前活动的类名作为SharedPreference的文件名。

（3）PreferenceManager类中的`getDefaultSharedPreference()`

> 这是一个静态方法，接受一个Context参数，自动使用当前包名作为SharedPreference的文件的前缀名



#### 1.2 获取SharedPreferences.Editor对象

通过获取到的SharedPreferences对象的`edit()`方法就可以获取SharedPreferences.Editor对象了，可以如下实现

```Java
SharedPreferences.Editor editor=getSharedPreference("data",MODE_PRIVATE).edit();
```

#### 1.3 向SharedPreferences.Editor对象中添加数据

获取到SharedPreferences.Editor对象后，就可以添加数据了，有三种方法：

> * putBoolean() ： 添加布尔数据
> * putString()： 添加字符串数据
> * putInt()： 添加整形数据

实例代码如下：

```Java
editor.putString("name","xiaoyang");
editor.putInt("age",27);
editor.putBoolean("married",false);
```

#### 1.4 数据提交

添加完数据，就可以使用`apply()`提交完成保存操作了

```Java
editor.apply();
```



此时数据就完成了存储，可以在`/data/data/<package-name>/shared_prefs/`文件夹下找到data文件进行查看，SharedPreferences是以xml格式来管理数据的。

## 二、从SharedPreferences中读取数据

从SharedPreferences中读取数据的方法和存储数据的方式对应，有三个如下所示，其中第一个参数表示要获取数据的键，第二个参数表示默认值，即当传入的键不存在时则以默认值返回。

- getBoolean() ： 获取布尔数据
- getString()： 获取字符串数据
- getInt()： 获取整形数据

实例代码如下：

```Java
SharedPreferences.Editor editor=getSharedPreference("data",MODE_PRIVATE).edit();
editor.getString("name","");
editor.getInt("age",0);
editor.getBoolean("married",false);
```
