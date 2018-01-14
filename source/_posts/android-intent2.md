---
title: Android之再谈Intent
date: 2018-01-14 19:40:18
tags: Android
---
1、在使用Intent启动新的activity时需要传递参数，如下面这样

```Java
Intent intent=new
Intent(MainActivity.this,SecondActivity.class);
String data="hello 你好！"
//第一个参数是键，第二个是值
intetn.putExtra(“extra_data”,data);
startActivity(intent);
```
但是对要传递的数据却不清楚，要么阅读SecondActivity代码，要么询问写SecondActivity的人，费时费力，如何减少这种麻烦呢？

2、可以在SecondActivity中添加一个静态方法

```Java
public static void actionStart(Context context,String data1,String data2){
    Intent i=new Intent(context,SecondActivity.class);
    i.putExtra("param1",data1);
    i.putExtra("param2",data2);
    context.startActivity(intent);
}
```
3、然后在MainActivity中可以直接调用SecondActivity中的方法来启动SecondActivity并进行传参

```Java
SecondActivity.actionStart(MainActivity.this,"data1","data2");
```
