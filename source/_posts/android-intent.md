---
title: Android之Intent
date: 2018-01-14 19:28:45
tags: Android
---

### 一、显式使用Intent
显式使用Intent很简单，如下所示就可以从MainActivity中启动secondActivity

```Java
//第一个参数表示上下文，第二个参数表示目标活动
Intent intent=new
Intent(MainActivity.this,SecondActivity.class);
startActivity(intent);
```

### 二、隐式使用Intent
隐式即是 不直接通过activity的类名来启动activity。怎么做呢？
1、在AndroidManifest.xml中找到要启动的Activity，添加intent-filter

```xml
<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.example.activitytest.ACTION_START"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity>
```
2、只有在<action>和<category>同时匹配上Intent时，才能启用该活动，在创建Intent时直接将action的字符串`com.example.activitytest.ACTION_START`传递进去，表示想要启动过能够响应这个字符串的Activity，下面的代码没有添加category，是因为`android.intent.category.DEFAULT`是默认的category，在调用startActivity时会自动添加到Intent中。

```Java
Intent intent=new
Intent(“com.example.activitytest.ACTION_START”);
startActivity(intent);
```

3、自定义category
如果要启动的category有自己的category，怎么办呢，如下

```xml
<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.example.activitytest.ACTION_START"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="com.example.activitytest.My_CATEGORY"/>
    </intent-filter>
</activity>
```
使用Intent的`addCategory`方法，就可以启动指定的activity了

```Java
Intent intent=new
Intent(“com.example.activitytest.ACTION_START”);
intent.addCategory("com.example.activitytest.My_CATEGORY");
startActivity(intent);
```

**注意：没个intent只能指定一个action，但是可以指定多个category**

### 三、隐式启用系统的服务
如何隐式的调用系统浏览器呢，如下：

```Java
Intent intent=new
Intent(Intent.ACTION_VIEW);
intent.setData("http://www.baidu.com");
startActivity(intent);
```
其中action`Intent.ACTION_VIEW`是一个系统内置的动作，常量值为“android.intent.action.VIEW”;
<p>'setData'方法指定当前Intent正在操作的数据，通常以字符串的形式传如到Uri.parse()方法中解析产生。</p>

可以给`<intent-filter/>`标签配置`<data>`标签，更精确的指定当前活动能够响应什么类型的数据，`<data>`标签可以配置如下内容：

* anroid:scheme    指定数据的协议
* android:host     指定数据的主机名，
* android:port     指定数据的端口
* android:path     指定主机鸣鹤端口之后的部分
* android:mimetype 指定处理的数据类型

只有`<data>`标签指定的内容和intent中携带的Data一致时，当前活动才能响应该intent。不过一般`<data>`标签不会指定过多的内容，如要启动浏览器，只需要指定Android:acheme为http即可，
如，给一个activity指定：

```xml
<activity android:name=".ThirdActivity">
    <intent-filter>
        <action android:name="android.intent.action.View"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:scheme="http"/>
    </intent-filter>
</activity>
```


这是如过再隐式调用浏览器服务，就会弹出让你选择的项。

除了调用系统的浏览器，还可以调用系统的一系列的其它服务，如调用系统的电话拨号界面：

```Java
Intent intent=new
Intent(Intent.ACTION_DIAL);
intent.setData("tel:10086");
startActivity(intent);
```

### 四、向下一个Activity传递数据
1、启动新的activity时传递数据

```Java
//第一个参数表示上下文，第二个参数表示目标活动
Intent intent=new
Intent(MainActivity.this,SecondActivity.class);
String data="hello 你好！"
//第一个参数是键，第二个是值
intetn.putExtra(“extra_data”,data);
startActivity(intent);
```
2、然后就能在SecondActivity的create方法中获取了

```Java
Intent intent=getIntent();
//又键获取相应的值
String data=intent.getStringExtra("extra_data");
```

* 这里如果值是字符串就使用getStringExtra；
* 如果是整型数据，就使用getIntExtra；
* 如果是布尔型数据，就使用getBooleanExtra;

当然也可以传递bundle对象。

### 五、向上一个Activity返回数据
如何给上一个activity中回传一个数据呢？
1、在第一个activity中使用`startActivityForResult`方法；

```Java
Intent intent=new Intent(FirstActivity.this,SecondActivity.class);
//1是请求码
startActivityForResult(intent,1);
```

然后重写`onActivityResult`方法,三个参数，第一个就是启动时传入的请求码，第二个是返回数据时传入的处理结果，第三个是携带着返回数据的Intent：

```Java
@Override
protected void onActivityResult(int requestCode,int resultCode,Intent data){
    switch(requestCode){
        case:1
            if(resultCode==RESULT_OK){
                String requestData=data.getStringExtra("data_return");
                //...
            }
            break;
        default:
    }
}
```
2、在要请求的SecondActivity中相应位置，组装intent，然后调用finish()销毁当前activity即可：

```Java
    btn.setOnClickListener(new OnCickListener(){
        @Override
        public void onClick(View v){
            Intent i=new Intent();
            i.putExtra("data_return","我是数据！");
            setResult(RESULT_OK,intent);
            finish();
        }
    });
```
这时，点击相应按钮，就可以在FirstActivity中收到回传的数据了。

如果SecondActivity页面是通过按返回键销毁的呢，只需要重写`onBackPressed`方法即可：

```Java
@Override
public void onBackPressed(){
     Intent i=new Intent();
     i.putExtra("data_return","我是数据！");
     setResult(RESULT_OK,intent);
     finish();
}
```

