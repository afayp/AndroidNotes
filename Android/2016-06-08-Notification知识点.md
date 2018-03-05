---
layout:     post
title:      "Notification知识点"
date:       2016-06-08 15:12:36
author:     "afayp"
catalog:    true
tags:
    - Android
---

  

Android在appcompat-v7库中提供了一个NotificationCompat类来处理新老版本的兼容问题，我们在编写通知功能时都使用NotificationCompat这个类来实现，appcompat-v7库就会自动帮我们做好所有系统版本的兼容性处理了

<!--more-->

# 常见样式

准备代码
```java
public static final int TYPE_Normal = 1;
public static final int TYPE_Progress = 2;
public static final int TYPE_BigText = 3;
public static final int TYPE_Inbox = 4;
public static final int TYPE_BigPicture = 5;
public static final int TYPE_Hangup = 6;
public static final int TYPE_Media = 7;
public static final int TYPE_Customer = 8;
private NotificationManager manger = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
```
权限
```java
<uses-permission android:name="android.permission.VIBRATE" />
<uses-permission android:name="android.permission.FLASHLIGHT"/>
```

## 普通通知

![](http://img.blog.csdn.net/20160424084923570)
![](http://img.blog.csdn.net/20160424090204106)

代码：
```java
private void simpleNotify(){
    //为了版本兼容  选择V7包下的NotificationCompat进行构造
    NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
    //Ticker是状态栏显示的提示
    builder.setTicker("简单Notification");
    //第一行内容  通常作为通知栏标题
    builder.setContentTitle("标题");
    //第二行内容 通常是通知正文
    builder.setContentText("通知内容");
    //第三行内容 通常是内容摘要什么的 在低版本机器上不一定显示
    builder.setSubText("这里显示的是通知第三行内容！");
    //ContentInfo 在通知的右侧 时间的下面 用来展示一些其他信息
    //builder.setContentInfo("2");
    //number设计用来显示同种通知的数量和ContentInfo的位置一样，如果设置了ContentInfo则number会被隐藏
    builder.setNumber(2);
    //可以点击通知栏的删除按钮删除
    builder.setAutoCancel(true);
    //系统状态栏显示的小图标
    builder.setSmallIcon(R.mipmap.ic_launcher);
    //下拉显示的大图标
    builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.push));
    Intent intent = new Intent(this,SettingsActivity.class);
    PendingIntent pIntent = PendingIntent.getActivity(this,1,intent,0);
    //点击跳转的intent
    builder.setContentIntent(pIntent);
    //通知默认的声音 震动 呼吸灯 
    builder.setDefaults(NotificationCompat.DEFAULT_ALL);
    Notification notification = builder.build();
    manger.notify(TYPE_Normal,notification);
}
```

以上设置在不同的系统版本显示有很多差异，使用时需要注意

- setTicker 通知到来时低版本上会在系统状态栏显示一小段时间 5.0以上版本好像没有用了
- setContentInfo和setNumber同时使用 number会被隐藏
- setSubText显示在通知栏的第三行文本，在低版本上不显示，比如4.0系统
- setVibrate设置震动 参数是个long[]｛震动时长，间隔时长，震动时长，间隔时长…｝单位毫秒 设置提醒声音 setSound(Uri sound) 一般默认的就好
- builder.setLights()设置呼吸灯的颜色 并不是所有颜色都被支持 个人感觉没什么用
- 清除通知栏特定通知 manager.cancel(id) id即为manger.notify()的第一个参数


## 下载进度的样式通知
![](http://img.blog.csdn.net/20160424091518738)

```java
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
builder.setSmallIcon(R.mipmap.ic_launcher);
builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.push));
//禁止用户点击删除按钮删除
builder.setAutoCancel(false);
//禁止滑动删除
builder.setOngoing(true);
//取消右上角的时间显示
builder.setShowWhen(false);
//进度显示在Title上
builder.setContentTitle("下载中..."+progress+"%");
//设置进度
builder.setProgress(100,progress,false);
//builder.setContentInfo(progress+"%");

Intent intent = new Intent(this,DownloadService.class);
intent.putExtra("command",1);
Notification notification = builder.build();
manger.notify(MainActivity.TYPE_Progress,notification);
```
注意点：

1. setProgress的第三个bool类型的参数表示progressbar的Indeterminate属性 指是否使用不确定模式 
2. 高版本上progressbar的进度值可以在setContentInfo显示，但是低版本上使用这个属性会导致progressbar不显示，setContentText一样



## BigTextStyle通知

点击前：

![](http://img.blog.csdn.net/20160424091538386)

点击后：

![](http://img.blog.csdn.net/20160424091553144)

代码：

```java
private void bigTextStyle(){
    NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
    builder.setContentTitle("BigTextStyle");
    builder.setContentText("BigTextStyle演示示例");
    builder.setSmallIcon(R.mipmap.ic_launcher);
    builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.notification));
    android.support.v4.app.NotificationCompat.BigTextStyle style = new android.support.v4.app.NotificationCompat.BigTextStyle();
    style.bigText("这里是点击通知后要显示的正文，可以换行可以显示很长很长很长很长很长很长很长很长很长很长很长很长很长很长很长很长很长很长");
    style.setBigContentTitle("点击后的标题");
    //SummaryText没什么用 可以不设置
    style.setSummaryText("末尾只一行的文字内容");
    builder.setStyle(style);
    builder.setAutoCancel(true);
    Intent intent = new Intent(this,SettingsActivity.class);
    PendingIntent pIntent = PendingIntent.getActivity(this,1,intent,0);
    builder.setContentIntent(pIntent);
    builder.setDefaults(NotificationCompat.DEFAULT_ALL);
    Notification notification = builder.build();
    manger.notify(TYPE_BigText,notification);
}
```
注意点：

1. 使用类 Android.support.v4.app.NotificationCompat.BigTextStyle 
2. 在低版本系统上只显示点击前的普通通知样式 如4.4可以点击展开，在4.0系统上就不行 
3. 点击前后的ContentTitle、ContentText可以不一致，bigText内容可以自动换行 好像最多5行的样子



## InboxStyle
![](http://img.blog.csdn.net/20160424091614082)
与bigTextStyle类似，点击前显示普通通知样式，点击后展开 
效果图 (点击后) 

代码：
```java
public void inBoxStyle(){
    NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
    builder.setContentTitle("InboxStyle");
    builder.setContentText("InboxStyle演示示例");
    builder.setSmallIcon(R.mipmap.ic_launcher);
    builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.notification));
    android.support.v4.app.NotificationCompat.InboxStyle style = new android.support.v4.app.NotificationCompat.InboxStyle();
    style.setBigContentTitle("BigContentTitle")
            .addLine("第一行，第一行，第一行，第一行，第一行，第一行，第一行")
            .addLine("第二行")
            .addLine("第三行")
            .addLine("第四行")
            .addLine("第五行")
            .setSummaryText("SummaryText");
    builder.setStyle(style);
    builder.setAutoCancel(true);
    Intent intent = new Intent(this,SettingsActivity.class);
    PendingIntent pIntent = PendingIntent.getActivity(this,1,intent,0);
    builder.setContentIntent(pIntent);
    builder.setDefaults(NotificationCompat.DEFAULT_ALL);
    Notification notification = builder.build();
    manger.notify(TYPE_Inbox,notification);
}
```

注意事项   

1. 使用类android.support.v4.app.NotificationCompat.InboxStyle 
2. 每行内容过长时并不会自动换行 
3. addline可以添加多行 但是多余5行的时候每行高度会有截断 
4. 同BigTextStyle 低版本上系统只能显示普通样式

## BigPictureStyle
点击后可以显示一个大图的通知   
效果图(点击后) 

![](http://img.blog.csdn.net/20160424091631957)
```java
public void bigPictureStyle(){
    NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
    builder.setContentTitle("BigPictureStyle");
    builder.setContentText("BigPicture演示示例");
    builder.setSmallIcon(R.mipmap.ic_launcher);
    builder.setDefaults(NotificationCompat.DEFAULT_ALL);
    builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.notification));
    android.support.v4.app.NotificationCompat.BigPictureStyle style = new android.support.v4.app.NotificationCompat.BigPictureStyle();
    style.setBigContentTitle("BigContentTitle");
    style.setSummaryText("SummaryText");
    style.bigPicture(BitmapFactory.decodeResource(getResources(),R.drawable.small));
    builder.setStyle(style);
    builder.setAutoCancel(true);
    Intent intent = new Intent(this,ImageActivity.class);
    PendingIntent pIntent = PendingIntent.getActivity(this,1,intent,0);
    //设置点击大图后跳转
    builder.setContentIntent(pIntent);
    Notification notification = builder.build();
    manger.notify(TYPE_BigPicture,notification);
}
```

注意事项 

1. 使用类android.support.v4.app.NotificationCompat.BigPictureStyle 
2. style.bigPicture传递的是个bitmap对象 所以也不应该传过大的图 否则会oom 
3. 同BigTextStyle 低版本上系统只能显示普通样式

## hangup横幅通知 

## MediaStyle

## 自定义通知栏布局

## 仿 QQ 音乐的 Notification

代码见第一个参考链接。。。


# 问题

将targetSdkVersion指定成21或者更高的话，通知图标可能会变成白白的一个圆了

详情见第二个参考链接


# 参考链接

<http://blog.csdn.net/w804518214/article/details/51231946>

<http://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650235923&idx=1&sn=af1fc1a6b60282732d94b0e7a354488f&scene=1&srcid=0517c0t12GnMgc5tWAkEMHNs#>

<http://blog.csdn.net/kkkkkxiaofei/article/details/17356233>

<http://www.sunnyang.com/219.html>