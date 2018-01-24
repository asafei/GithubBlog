---
title: Android之menu的使用
date: 2018-01-24 21:06:30
tags: Android
---
### 一、如何给一个activity设置menu
1、在`res`目录下新建名为`menu`的文件夹；
2、在`menu`文件夹下新建一个名为`main`的 `Menu resource file`文件
3、在`main`文件中添加两个item，其中`android:title`表示名称；

```xml
<menu xmlns android="http:/schemas.android.com/apk/res/android">
    <item
        android:id="@+id/add_item"
        android:title="添加"/>
    <item
        android:id="@+id/remove_item"
        android:title="删除"/>
</menu>
```
4、在相应的activity文件中重写`onCreateOptionMenu()`方法，其中`getMenuInflater()`方法得到MenuInflater对象，inflate方法创建菜单，第一个参数是指定的资源文件，第二个参数指定菜单项添加到哪一个Menu对象中；返回`true`表示允许创建的菜单显示出来，false表示无法显示。

```Java
public boolean onCreateOptionMenu(Menu menu){
    getMenuInflater().inflate(R.menu.main,menu);
    return true;
}
```

5、定义菜单响应事件，重写`onOptionsItemSelected()`方法
```
public boolean onOptionsItemSelected(MenuItem item){
    switch(item.getItemId()){
        case R.id.add_item:
            //...
            break;
        case R.id.remove_item:
            //...
            break;
        default:
    }
    return true;
}
```

这时运行程序，在相应的activity页面右侧就会出现一个三点符号了，即菜单按钮。
