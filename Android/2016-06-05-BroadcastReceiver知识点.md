---
layout:     post
title:      "BroadcastReceiver知识点"
date:       2016-06-05 20:13:18
author:     "afayp"
catalog:    true
tags:
    - Android
---



# 创建BroadcastReceiver

<!--more-->

新建类（或者使用内部类）继承BroadcastReceiver，重写onReceiver方法
```java
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
		intent.getAction()//判断收到的是哪个广播(可能会有多个action)
        //可以通过这个intent拿到发送过来的消息
        String msg = intent.getStringExtra("msg");
    }
}
```

# 注册

## 静态注册
静态注册是在AndroidManifest.xml文件中配置的
```java
<receiver android:name=".MyReceiver">  
    <intent-filter>  
        <action android:name="android.intent.action.MY_BROADCAST"/>  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter>  
	<intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>  
```
name决定注册哪个广播；action决定这个receiver接受什么广播  
静态注册表示这个程序未启动也会接收到对应的广播，然后运行onReceive方法里面的方法。（Android 3.1开始不再成立，原因见后面）  
一般上面那样写就可以了，下面把所有属性列一遍：
```java
<receiver android:enabled=["true" | "false"]
	android:exported=["true" | "false"]
	android:icon="drawable resource"
	android:label="string resource"
	android:name="string"
	android:permission="string"
	android:process="string" >
</receiver>
```

- android:exported  ——此broadcastReceiver能否接收其他App的发出的广播，这个属性默认值有点意思，其默认值是由receiver中有无intent-filter决定的，如果有intent-filter，默认值为true，否则为false。（同样的，activity/service中的此属性默认值一样遵循此规则）同时，需要注意的是，这个值的设定是以application或者application user id为界的，而非进程为界（一个应用中可能含有多个进程）；

- android:name  —— 此broadcastReceiver类名；
- android:permission  ——如果设置，具有相应权限的广播发送方发送的广播才能被此broadcastReceiver所接收；
- android:process  ——broadcastReceiver运行所处的进程。默认为app的进程。可以指定独立的进程（Android四大基本组件都可以通过此属性指定自己的独立进程）

## 动态注册
动态注册需要在代码中动态的指定广播地址并注册，通常我们是在Activity或Service注册一个广播
```java
MyReceiver receiver = new MyReceiver();  
          
IntentFilter filter = new IntentFilter();  
filter.addAction("android.intent.action.MY_BROADCAST");  
          
registerReceiver(receiver, filter);  
```
必须在某处解除注册
```java
protected void onDestroy() {  
    super.onDestroy();  
    unregisterReceiver(receiver);  
}  
```

# 发送广播

## 标准(普通)广播（Normal Broadcast）
每个接收者同时收到，接受者之间互不影响，无法拦截
```java
//可以直接将action写在intent的构造里面：
Intent intent = new Intent("android.intent.action.MY_BROADCAST");
//也可以用setAction(..)方法，来对应于BroadcastReceiver中的intentFilter中的action。
Intent intent2 = new Intent();
intent2.setAction("android.intent.action.MY_BROADCAST");

intent.putExtra("msg","hello");
sendBroadcast(intent);
```
sendBroadcast也是android.content.ContextWrapper类中的方法，它可以将一个指定地址和参数信息的Intent对象以广播的形式发送出去。

## 有序广播（Ordered Broadcast）
有序广播中的“有序”是针对广播接收者而言的，指的是发送出去的广播被BroadcastReceiver按照先后循序接收。有序广播的定义过程与普通广播无异，只是其的主要发送方式变为：
```java
sendOrderedBroadcast(intent, receiverPermission, ...)。
```
这个方法需要一个权限参数，如果为null则表示不要求接收者声明指定的权限，如果不为null，则表示接收者若要接收此广播，需声明指定权限。这样做是从安全角度考虑的，例如系统的短信就是有序广播的形式，一个应用可能是具有拦截垃圾短信的功能，当短信到来时它可以先接受到短信广播，必要时终止广播传递，这样的软件就必须声明接收短信的权限。下面举个例子：
```java
//在AndroidMainfest.xml中定义一个权限
<permission android:protectionLevel="normal"  
            android:name="scott.permission.MY_BROADCAST_PERMISSION" />  
//然后声明使用了此权限
<uses-permission android:name="scott.permission.MY_BROADCAST_PERMISSION" /> 
//发送
sendOrderedBroadcast(intent, "scott.permission.MY_BROADCAST_PERMISSION");  
```



intent-filter中可以设置优先级。-1000到1000，数值越大，优先级越高。
```java
<receiver android:name=".FirstReceiver">  
    <intent-filter android:priority="1000">  
        <action android:name="android.intent.action.MY_BROADCAST"/>  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter>  
</receiver>  
```

- 多个具当前已经注册且有效的BroadcastReceiver接收有序广播时，是按照先后顺序接收的，先后顺序判定标准遵循为：将当前系统中所有有效的动态注册和静态注册的BroadcastReceiver按照priority属性值从大到小排序，对于具有相同的priority的动态广播和静态广播，动态广播会排在前面。

- 先接收的BroadcastReceiver可以对此有序广播进行截断，使后面的BroadcastReceiver不再接收到此广播，也可以对广播进行修改，使后面的BroadcastReceiver接收到广播后解析得到错误的参数值。当然，一般情况下，不建议对有序广播进行此类操作，尤其是针对系统中的有序广播。

### 截断有序广播
只需在onReceive中添加
```java
abortBroadcast();  
```
### 修改广播
onReceive方法中调用setResultExtras方法将一个Bundle对象设置为结果集对象，传递到下一个接收者那里，这样以来，优先级低的接收者可以用getResultExtras获取到最新的经过处理的信息集合。
```java
@Override  
public void onReceive(Context context, Intent intent) {  
    String msg = intent.getStringExtra("msg");  
    Log.i(TAG, "FirstReceiver: " + msg);  
      
    Bundle bundle = new Bundle();  
    bundle.putString("msg", msg + "@FirstReceiver");  
    setResultExtras(bundle);  
}  

getResultExtras(true).getString("msg"); 

```

## 本地广播（Local Broadcast）
广播的发送者和接收者都同属于一个App，使用方式上与通常的全局广播几乎相同，只是注册/取消广播接收器和发送广播时将主调context变成了LocalBroadcastManager的单一实例。  
另外还有一点，本地广播无法静态注册，只能动态注册。本地广播相对来说更加高效。

```java
LocalBroadcastManager localBroadcastManager = LocalBroadcastManager.getInstance(this);
//发送本地广播
Intent intent = new Intent();
intent.setAction(BROADCAST_ACTION);
localBroadcastManager.sendBroadcast(intent);

//注册本地广播接收器
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction("...");
localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);

//取消注册应用内广播接收器
localBroadcastManager.unregisterReceiver(mBroadcastReceiver);
```

对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册的ContextReceiver才有可能接收到（静态注册或其他方式动态注册的ContextReceiver是接收不到的）。


# 接收广播
当发送的广播被接收器监听到后，会调用它的onReceive()方法，并将包含消息的Intent对象传给它。onReceive中代码的执行时间不要超过5s，否则Android会弹出超时dialog。默认情况下，广播接收器也是运行在UI线程，因此，onReceive方法中不能执行太耗时的操作。否则将因此ANR。

## 不同注册方式的广播接收器回调onReceive(context, intent)中的context具体类型

- 对于静态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是ReceiverRestrictedContext；

- 对于全局广播的动态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是Activity Context；

- 对于通过LocalBroadcastManager动态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是Application Context。


# 不同API版本的改变
”静态注册的广播接收器即使app已经退出，主要有相应的广播发出，依然可以接收到，但此种描述自Android 3.1开始有可能不再成立“

Android 3.1开始系统在Intent与广播相关的flag增加了参数，分别是

- FLAG_INCLUDE_STOPPED_PACKAGES：包含已经停止的包（停止：即包所在的进程已经退出）

- FLAG_EXCLUDE_STOPPED_PACKAGES：不包含已经停止的包

主要原因如下：

自Android3.1开始，系统本身则增加了对所有app当前是否处于运行状态的跟踪。在发送广播时，不管是什么广播类型，系统默认直接增加了值为FLAG_EXCLUDE_STOPPED_PACKAGES的flag，导致即使是静态注册的广播接收器，对于其所在进程已经退出的app，同样无法接收到广播。

详情参加Android官方文档：<http://developer.android.com/about/versions/android-3.1.html#launchcontrols>

由此，对于系统广播，由于是系统内部直接发出，无法更改此intent flag值，因此，3.1开始对于静态注册的接收系统广播的BroadcastReceiver，如果App进程已经退出，将不能接收到广播。

但是对于自定义的广播，可以通过复写此flag为FLAG_INCLUDE_STOPPED_PACKAGES，使得静态注册的BroadcastReceiver，即使所在App进程已经退出，也能能接收到广播，并会启动应用进程，但此时的BroadcastReceiver是重新新建的。
```java
Intent intent = new Intent();
intent.setAction(BROADCAST_ACTION);
intent.addFlags(Intent.FLAG_INCLUDE_STOPPED_PACKAGES);
intent.putExtra("name", "qqyumidi");
sendBroadcast(intent);
```

注1：对于动态注册类型的BroadcastReceiver，由于此注册和取消注册实在其他组件（如Activity）中进行，因此，不受此改变影响。

注2：在3.1以前，相信不少app可能通过静态注册方式监听各种系统广播，以此进行一些业务上的处理（如即时app已经退出，仍然能接收到，可以启动service等..）,3.1后，静态注册接受广播方式的改变，将直接导致此类方案不再可行。于是，通过将Service与App本身设置成不同的进程已经成为实现此类需求的可行替代方案。


# 常见的广播例子
## 开机启动服务
接收到开机广播后我们就可以启动自己的服务了。我们来看一下BootCompleteReceiver和MsgPushService的具体实现：  

Receiver:
```java
public class BootCompleteReceiver extends BroadcastReceiver {  
      
    private static final String TAG = "BootCompleteReceiver";  
      
    @Override  
    public void onReceive(Context context, Intent intent) {  
        Intent service = new Intent(context, MsgPushService.class);  
        context.startService(service);  
        Log.i(TAG, "Boot Complete. Starting MsgPushService...");  
    }  
  
}  
```
Service:

```java
public class MsgPushService extends Service {  
  
    private static final String TAG = "MsgPushService";  
      
    @Override  
    public void onCreate() {  
        super.onCreate();  
        Log.i(TAG, "onCreate called.");  
    }  
      
    @Override  
    public int onStartCommand(Intent intent, int flags, int startId) {  
        Log.i(TAG, "onStartCommand called."); //开机要实现的代码 
        return super.onStartCommand(intent, flags, startId);  
    }  
  
    @Override  
    public IBinder onBind(Intent arg0) {  
        return null;  
    }  
}  
```
AndroidManifest.xml：
```java
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" /> 
 
<receiver android:name=".BootCompleteReceiver">  
    <intent-filter>  
        <!-- 注册开机广播地址-->  
        <action android:name="android.intent.action.BOOT_COMPLETED"/>  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter>  
</receiver>  
 
<service android:name=".MsgPushService"/>  
```

## 网络状态变化

```java
//receiver的action：
<action android:name="android.net.conn.CONNECTIVITY_CHANGE"/> 
//权限：
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>  
```

## 电量变化

Receiver:
```java
public class BatteryChangedReceiver extends BroadcastReceiver {  
  
    private static final String TAG = "BatteryChangedReceiver";  
      
    @Override  
    public void onReceive(Context context, Intent intent) {  
        int currLevel = intent.getIntExtra(BatteryManager.EXTRA_LEVEL, 0);  //当前电量  
        int total = intent.getIntExtra(BatteryManager.EXTRA_SCALE, 1);      //总电量  
        int percent = currLevel * 100 / total;  
        Log.i(TAG, "battery: " + percent + "%");  
    }  
}  
```
注册：
```java
<receiver android:name=".BatteryChangedReceiver">  
    <intent-filter>  
        <action android:name="android.intent.action.BATTERY_CHANGED"/>  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter>  
</receiver>  
```
这样当电量发生变化时就可以收到通知了，如果想主动获取：
```java
Intent batteryIntent = getApplicationContext().registerReceiver(null,  
        new IntentFilter(Intent.ACTION_BATTERY_CHANGED));  
int currLevel = batteryIntent.getIntExtra(BatteryManager.EXTRA_LEVEL, 0);  
int total = batteryIntent.getIntExtra(BatteryManager.EXTRA_SCALE, 1);  
int percent = currLevel * 100 / total;  
Log.i("battery", "battery: " + percent + "%");  
```

# 参考链接
<http://blog.csdn.net/liuhe688/article/details/6955668>
<http://www.cnblogs.com/lwbqqyumidi/p/4168017.html>
