---
title: Android之引入布局
date: 2018-01-14 19:42:00
tags: Android
---
### 一、引入xml文件
如果你要自定义标题栏，是不是要为每一个页面都重复的编写标题栏配置呢，当然不用，使用引入布局就可以实现一次编写处处使用的效果。
1、首先在layouy文件夹下新建title.xml的布局文件

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@drawable/title_bg">
    <Button
        android:id="@+id/title_back"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/back_bg"
        android:text="back"
        android:textColor="#fff"/>
    <TextView
        android:id="@+id/title_txt"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_weight="1"
        android:garvity="center"
        android:text="title"
        android:textColor="#fff"
        android:textSize="24sp"/>
    <Button
        android:id="@+id/title_edit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/edit_bg"
        android:text="edit"
        android:textColor="#fff"/>
</LinearLayout>
    
```
2、在Activity的布局文件中通过关键字include引入该布局

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <include layout="@layout/title" />
</LinearLayout>
```

3、在Activity的java文件中隐藏默认标题栏

```Java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContent(R.layout.activity_main);
    
    ActionBar actionBar=getSupportActionBar();
    if(actionBar!=null){
        actionBar.hide();
    }
}
```

### 二、引入带监听事件的控件
仅仅引入xml文件，如果给title添加监听事件岂不是还要做大量的重复工作，对于一些通用的监听事件也可以独立出来。

1、新建一个类继承LinearLayout, 并且在里面添加监听按钮事件

```Java
public class TitleLayout extends LinearLayout{
    public TitleLayout(Context context , AttributeSet attrs){
        super(context,attrs);
        //inflate方法动态加载一个布局文件
        LayoutInflater.from(context).inflate(R.layout.title,this);
        
        //给返回按钮，添加监听
        Button btn1=findViewById(R.id.title_back);
        btn1.setOnClickListener(new OnClickListener(){
            @Override
            public void onClick(View v){
                ((Activity)getContext()).finish();
            }
        });
    }
}
```

2、在activity_main.xml文件中添加这个控件即可

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <com.example.ui.TitleLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_contetn"/>
</LinearLayout>
```
