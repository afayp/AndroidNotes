---
layout:     post
title:      "Toast和Snackbar"
date:       2016-06-08 14:40:27
author:     "afayp"
catalog:    true
tags:
    - Android
---

  

# Toast
Toast用于向用户显示一些帮助/提示。下面展示了5中效果。

<!--more-->

## 默认效果

```java
Toast.makeText(getApplicationContext(), "默认Toast样式",
     Toast.LENGTH_SHORT).show();
```

## 自定义显示位置

```java
toast = Toast.makeText(getApplicationContext(),
     "自定义位置Toast", Toast.LENGTH_LONG);
   toast.setGravity(Gravity.CENTER, 0, 0);
   toast.show();
```

## 带图片

```java
toast = Toast.makeText(getApplicationContext(),
     "带图片的Toast", Toast.LENGTH_LONG);
   toast.setGravity(Gravity.CENTER, 0, 0);
   LinearLayout toastView = (LinearLayout) toast.getView();
   ImageView imageCodeProject = new ImageView(getApplicationContext());
   imageCodeProject.setImageResource(R.drawable.icon);
   toastView.addView(imageCodeProject, 0);
   toast.show();
```

## 完全自定义效果
```java
LayoutInflater inflater = getLayoutInflater();
   View layout = inflater.inflate(R.layout.custom,
     (ViewGroup) findViewById(R.id.llToast));
   ImageView image = (ImageView) layout
     .findViewById(R.id.tvImageToast);
   image.setImageResource(R.drawable.icon);
   TextView title = (TextView) layout.findViewById(R.id.tvTitleToast);
   title.setText("Attention");
   TextView text = (TextView) layout.findViewById(R.id.tvTextToast);
   text.setText("完全自定义Toast");
   toast = new Toast(getApplicationContext());
   toast.setGravity(Gravity.RIGHT | Gravity.TOP, 12, 40);
   toast.setDuration(Toast.LENGTH_LONG);
   toast.setView(layout);
   toast.show();
```
![](http://pic002.cnblogs.com/images/2010/133128/2010111012464850.png)


## 其他线程

```java
new Thread(new Runnable() {
    public void run() {
     showToast();
    }
   }).start();
```

# Snackbar

## 基本使用
使用Snackbar要导入com.android.support:design库。

Snackbar显示在所有屏幕其它元素之上(屏幕最顶层)，同一时间只能显示一个snackbar。

```java
Snackbar.make(coordinatorLayout,"这是massage", Snackbar.LENGTH_LONG).setAction("这是action", new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Toast.makeText(MainActivity.this,"你点击了action",Toast.LENGTH_SHORT).show();
     }
 }).show();
```
make()方法是生成Snackbar的。Snackbar需要一个控件容器view用来容纳，官方推荐使用CoordinatorLayout来确保Snackbar和其他组件的交互。显示时间duration有三种类型LENGTH_SHORT、LENGTH_LONG和LENGTH_INDEFINITE。

setAction()方法可设置Snackbar右侧按钮，增加进行交互事件。如果不使用setAction()则只显示左侧message。


## 颜色样式

修改Snackbar的背景颜色和message字体颜色

```java
public static void setSnackbarColor(Snackbar snackbar, int messageColor, int backgroundColor) {
    View view = snackbar.getView();//获取Snackbar的view
    if(view!=null){
        view.setBackgroundColor(backgroundColor);//修改view的背景色
        ((TextView) view.findViewById(R.id.snackbar_text)).setTextColor(messageColor);//获取Snackbar的message控件，修改字体颜色
    }
}
```

## 增加图标

官方不建议这么玩。具体看参考链接。

## SnackbarUtil
```java
/**
 * Created by 赵晨璞 on 2016/5/1.
 */
public class SnackbarUtil {

public static final   int Info = 1;
public static final  int Confirm = 2;
public static final  int Warning = 3;
public static final  int Alert = 4;


public static  int red = 0xfff44336;
public static  int green = 0xff4caf50;
public static  int blue = 0xff2195f3;
public static  int orange = 0xffffc107;

/**
 * 短显示Snackbar，自定义颜色
 * @param view
 * @param message
 * @param messageColor
 * @param backgroundColor
 * @return
 */
public static Snackbar ShortSnackbar(View view, String message, int messageColor, int backgroundColor){
    Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_SHORT);
    setSnackbarColor(snackbar,messageColor,backgroundColor);
    return snackbar;
}

/**
 * 长显示Snackbar，自定义颜色
 * @param view
 * @param message
 * @param messageColor
 * @param backgroundColor
 * @return
 */
public static Snackbar LongSnackbar(View view, String message, int messageColor, int backgroundColor){
    Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_LONG);
    setSnackbarColor(snackbar,messageColor,backgroundColor);
    return snackbar;
}

/**
 * 自定义时常显示Snackbar，自定义颜色
 * @param view
 * @param message
 * @param messageColor
 * @param backgroundColor
 * @return
 */
public static Snackbar IndefiniteSnackbar(View view, String message,int duration,int messageColor, int backgroundColor){
    Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_INDEFINITE).setDuration(duration);
    setSnackbarColor(snackbar,messageColor,backgroundColor);
    return snackbar;
}

/**
 * 短显示Snackbar，可选预设类型
 * @param view
 * @param message
 * @param type
 * @return
 */
public static Snackbar ShortSnackbar(View view, String message, int type){
    Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_SHORT);
    switchType(snackbar,type);
    return snackbar;
}

/**
 * 长显示Snackbar，可选预设类型
 * @param view
 * @param message
 * @param type
 * @return
 */
public static Snackbar LongSnackbar(View view, String message,int type){
    Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_LONG);
    switchType(snackbar,type);
    return snackbar;
}

/**
 * 自定义时常显示Snackbar，可选预设类型
 * @param view
 * @param message
 * @param type
 * @return
 */
public static Snackbar IndefiniteSnackbar(View view, String message,int duration,int type){
    Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_INDEFINITE).setDuration(duration);
    switchType(snackbar,type);
    return snackbar;
}

//选择预设类型
private static void switchType(Snackbar snackbar,int type){
    switch (type){
        case Info:
            setSnackbarColor(snackbar,blue);
            break;
        case Confirm:
            setSnackbarColor(snackbar,green);
            break;
        case Warning:
            setSnackbarColor(snackbar,orange);
            break;
        case Alert:
            setSnackbarColor(snackbar,Color.YELLOW,red);
            break;
    }
}

/**
 * 设置Snackbar背景颜色
 * @param snackbar
 * @param backgroundColor
 */
public static void setSnackbarColor(Snackbar snackbar, int backgroundColor) {
    View view = snackbar.getView();
    if(view!=null){
        view.setBackgroundColor(backgroundColor);
    }
}

/**
 * 设置Snackbar文字和背景颜色
 * @param snackbar
 * @param messageColor
 * @param backgroundColor
 */
public static void setSnackbarColor(Snackbar snackbar, int messageColor, int backgroundColor) {
    View view = snackbar.getView();
    if(view!=null){
        view.setBackgroundColor(backgroundColor);
        ((TextView) view.findViewById(R.id.snackbar_text)).setTextColor(messageColor);
    }
}

/**
 * 向Snackbar中添加view
 * @param snackbar
 * @param layoutId
 * @param index 新加布局在Snackbar中的位置
 */
public static void SnackbarAddView( Snackbar snackbar,int layoutId,int index) {
    View snackbarview = snackbar.getView();
    Snackbar.SnackbarLayout snackbarLayout=(Snackbar.SnackbarLayout)snackbarview;

    View add_view = LayoutInflater.from(snackbarview.getContext()).inflate(layoutId,null);

    LinearLayout.LayoutParams p = new LinearLayout.LayoutParams( LinearLayout.LayoutParams.WRAP_CONTENT,LinearLayout.LayoutParams.WRAP_CONTENT);
    p.gravity= Gravity.CENTER_VERTICAL;

    snackbarLayout.addView(add_view,index,p);
}

}
```




# 参考链接

<http://www.cnblogs.com/salam/archive/2010/11/10/1873654.html>
<http://www.jianshu.com/p/cd1e80e64311>