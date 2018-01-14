---
title: Android之退出应用
date: 2018-01-14 19:38:24
tags: Android
---
### 一、activity的退出
对于单个的activity的退出，只需要简单的调用`finish()`方法即可。

### 二、应用程序的退出
如果又多个activity启动，如何在任意一个activity中退出该应用程序呢？

**思路：**创建一个activity的管理类，管理所有启动的activity。
**实现：**
1、创建一个活动管理器

```Java
public class ActivityControllor{
    public static List<Activity> activityList=new ArrayList<>();
    public static addActivity(Activity activity){
        activityList.add(activity);
    }
    
    public static void removeActivity(Activity activity){
        activityList.remove(activity);
    }
    
    public static void finishAll(){
        for(Activity activity:activityList){
            if(!activity.isFinishing()){
                activity.finish();
            }
        }
        activityList.clear();
    }
}
```

2、创建一个activity的基类，重写`onCreate`和`onDestroy`方法

```Java
public class BaseActivity extends AppCompatActivrty{
    @Override
    public void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        //添加此activity
        Activitycontroller.addActivity(this);
    }
    
    @Override
    public void onDestroy(){
        super.onDestroy();
        //移除该activity
        ActivityController.removeActivity(this);
    }
}
```

3、然后所有创建的Activity都继承该BaseActivity，若要退出，只需要调用静态的`finishAll()`方法即可,然后再杀掉当前进程

```Java
ActivityController.finishAll();
// killProcess()只能杀掉当前进程，mypid()方法获取当前进程
android.os.Process.killProcess(android.os.Process.myPid());
```

