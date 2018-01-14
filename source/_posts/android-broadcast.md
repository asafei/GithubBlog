---
title: Android之广播接收器
date: 2018-01-14 19:47:46
tags: Android
---
广播，之前也又学习过，但却未再项目中实战过，今天再回顾一下。

Android中每个应用程序都可以对自己感兴趣的广播进行注册，以获取对自己有用的信息。广播可以分两类：

* 标准广播：完全异步执行，所有接收器同一时刻接收
* 有序广播：同步执行，同一时刻只有一个接收器能接收，优先级高的先接收，并且可以决定是否可以截断该广播

## 一、接收系统广播
组册广播的方式又两种：

* 动态注册：在代码中注册
* 静态注册：在AndroidManifest.xml中注册

### 1.1 动态注册广播监听网络变化
#### 1.1.1 创建一个广播接收器
新建一个类继承自BroadcastReciever，重写onReceiver()方法，这样方广播到来时，就会执行onReceiver()方法：

```Java
public class MainActivity extends AppcompatActivity{
    private IntentFilter intentFilter;
    private NetworkChangeReceiver networkChangeReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //1、创建IntentFileter实例
        intentFilter=new IntentFileter();
        //2、添加对网络状态变化的监听
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        //3、创建NetworkChangeReceiver实例
        networkChangeReceiver=new NetworkChangeReceiver();
        //4、注册广播
        registerReceiver(networkChangeReceiver,intentFilter);
        
    }
    @Override
    protected void onDestroy(){
        super.onDestroy();
        //5、取消对网络变化的广播注册
        unregisterReceiver(networkChangeReceiver);
    }



    class NetworkChangeReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context,Intent intent){
            //do something
            
        }
    }
}
```
#### 1.1.2 处理广播事件
在NetworkChangeReceiver中重写onReceive()方法，那么怎么处理该事件呢，可以这么做，提示网络状态

```Java
class NetworkChangeReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context,Intent intent){
            //1、获取ConnectivityManager实例，一个管理网络连接的系统服务类
            ConnectivityManager connectivityManager=getSystemService(Context.CONNECTIVITY_SERVICE);
            //2、得到NetworkInfo实例，可以判断是否有网络
            NetworkInfo networkInfo=connectivityManager.getActiveNetworkInfo();
            if(networkInfo!=null && networkInfo.isAvailable()){
                Toast.makeText(context,"网络可用",Toast.LENGTH_SHORT).show();
            }else{
                Toast.makeText(context,"网络不可用",Toast.LENGTH_SHORT).show();
            }
            
        }
    }
```
需要注意的是，需要在AndroidManifest.xml中声明网络请求的权限，否则会直接崩溃

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```

### 1.2 静态注册广播实现开机启动
动态注册广播只有在程序启动之后才能接收广播，如果想不启动程序也能接收广播，可以吗？可以，使用静态注册的方式

#### 1.2.1 创建一个广播接收器
Android studio中右键-new-other-Broadcast Receiver；

* Exported属性表示允许这个广播接收器接收本程序以外的广播
* Enabled属性表示是否启用这个广播接收器

```Java
public class BootCompleteReceiver extends BoradcastReceiver{
    @Override
    public void onReceiver(Context context,Intent intent){
        Toast.makeText(context,"开机啦",Toast.LENGTH_SHORT).show();
    }
}
```
#### 1.2.2 在AndroidManifest.xml中注册该广播
在application中添加一个新标签`<receiver>`，Android studio中自动完成了。并且在内部添加`<intent-filter>`标签，添加开机启动的action，这样在开机启动发出广播时就可以被接收到了

```xml
<receiver
    android:name=".BootCompleteReceiver"
    android:enabled="true"
    android:exprted="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED">
    </intent-filter>
</receiver>
```
#### 1.2.3 添加权限声明
此时还不能实现接收开机广播，还需要添加权限声明

```xml
<uses-permission android:name="android.permission.RECEIVER_BOOT_COMPLETED"/>
```

**注意：在onReceive()方法中不要进行复杂耗时的操作，广播接收器中不允许开启线程，若允许长时间就会报错。通常是发通知或启动服务**


## 二、发送自定义广播
会接收广播，就得会发送广播，
### 2.1 发送标准广播
（1） 定义广播接收器

```Java
public class MyBroadcastReceiver extends BoradcastReceiver{
    @Override
    public void onReceiver(Context context,Intent intent){
        Toast.makeText(context,"自己的广播",Toast.LENGTH_SHORT).show();
    }
}
```
（2）在AndroidManifest.xml中对该接收器修改

```xml
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exprted="true">
    <intent-filter>
        <action android:name="com.example.broadcast.My_Broadcast">
    </intent-filter>
</receiver>
```
（3）发送广播
此时就可以在相应的地方通过Context发送该广播了，可以给一个按钮添加监听，将广播发送出去，此时所有监听“com.example.broadcast.My_Broadcast”的接收器都可以收到一条标准广播了

```Java
button.setOnclickListener(
    new View.OnClickListener(){
        @Override
        public void onClick(View v){
            Intent i=new Intent("com.example.broadcast.My_Broadcast");
            sendBroadcast(i);
        }
    }
);
```

### 2.2 发送有序广播
标准广播可以被多个进程同时接收，如何发送有序广播呢？
（1）修改广播的发送方式

```Java
button.setOnclickListener(
    new View.OnClickListener(){
        @Override
        public void onClick(View v){
            Intent i=new Intent("com.example.broadcast.My_Broadcast");
            //实行Context的sendOrderedBroadcast方法发送有序广播；第二个参数是与权限有关的字符串
            sendOrderedBroadcast(i,null);
        }
    }
);
```

（2）在AndroidManifest.xml中对该接收器添加优先级别，这样优先级越高的就先接到该广播。（100优先级最高）

```xml
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exprted="true">
    <intent-filter android:priority="100">
        <action android:name="com.example.broadcast.My_Broadcast">
    </intent-filter>
</receiver>
```

（3）截断广播
如果优先级较高的广播接收器，在接收到该广播之后不想让后边优先级低的接收器继续接收，在onReceive中使用`abortBrodcast()`方法，这样该广播就不会继续传递下去了。

```Java
public class MyBroadcastReceiver extends BoradcastReceiver{
    @Override
    public void onReceiver(Context context,Intent intent){
        Toast.makeText(context,"自己的广播",Toast.LENGTH_SHORT).show();
        abortBrodcast();//截断该广播
    }
}
```

## 三、使用本地广播
上边使用的都是全局广播，很容易引起垃圾广播，如何只让广播在应用程序内部传播呢，很简单只需要使用一个`LocalBroadcastManager`对广播进行管理就可以了

```Java
public class MainActivity extends AppcompatActivity{
    private IntentFilter intentFilter;
    private LocalReceiver localReceiver;
    private LocalBroadcastManager localBroadcastManager;
    
    
    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        localBroadcastManager=LocalBroadcastManager.getInstance(this);
        Button btn=...
        btn.setOnClickListener(new OnClickListener(){
            @Override
            public void onClick(){
                Intent i=new Intent("com.example.broadcast.LOCAL_BROADCAST");
                localBroadcastManager.sendBroadcast(i);
            }
        });
        
        //1、创建IntentFileter实例
        intentFilter=new IntentFileter();
        //2、添加对网络状态变化的监听
        intentFilter.addAction("com.example.broadcast.LOCAL_BROADCAST");
        //3、LocalReceiver
        localReceiver=new LocalReceiver();
        //4、注册本地广播
        localBroadcastManager.registerReceiver(localReceiver,intentFilter);
        
    }
    @Override
    protected void onDestroy(){
        super.onDestroy();
        //5、取消对网络变化的广播注册
        localBroadcastManager.unregisterReceiver(localReceiver);
    }

    class LocalReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context,Intent intent){
            //do something
            
        }
    }
}
```

**小技巧：一般如果要实现在任何一个界面都可以直接退出应用程序的功能时，可以创建一个BaseActivity，在里面添加相应的操作，然后所有的活动都继承于该BaseActivity即可**
