---
layout:     post
title:      "PopupWindow"
date:       2016-04-09 19:50:30
author:     "afayp"
catalog:    true
tags:
    - Android
---



# 区别

Android的对话框有两种：PopupWindow和AlertDialog。它们的不同点在于：

<!--more-->

- AlertDialog的位置固定，而PopupWindow的位置可以随意
- AlertDialog是非阻塞线程的，AlertDialog弹出的时候，后台可是还可以做其他事情的;
- 而PopupWindow是阻塞线程的,这就意味着在我们退出这个弹出框之前，程序会一直等待


PopupWindow这个类用来实现一个弹出框，可以使用任意布局的View作为其内容，这个弹出框是悬浮在当前activity之上的。
# 基本使用
```java
//填充一个自定义的布局，作为显示的内容
View contentView = LayoutInflater.from(mContext).inflate(R.layout.pop_window, null);

//find出这个布局中需要的组件，比如设置按钮的点击事件
Button button = (Button) contentView.findViewById(R.id.button1);

//new出一个popupwindow实例，参数分别是：布局；宽；高；是否能获得焦点
PopupWindow popupWindow = new PopupWindow(contentView,LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT, true);

//一般都设置为true
popupWindow.setTouchable(true);

// 如果不设置PopupWindow的背景，无论是点击外部区域还是Back键都无法dismiss弹框
popupWindow.setBackgroundDrawable(getResources().getDrawable(R.drawable.selectmenu_bg_downward));

// 设置好参数之后再show
popupWindow.showAsDropDown(view);
```

# 细节

## 构造

```java
public PopupWindow(View contentView, int width, int height, boolean focusable)
```

contentView为要显示的view，width和height为宽和高，值为像素值，也可以是MATCHT_PARENT和WRAP_CONTENT；是否能获得焦点。  
有很多构造，一般都用这个。

## 改变PopupWindow的视图内容
```java
public void setContentView(View contentView)
```

用这个方法来改变popup的显示内容，也可以用来初始化PopupWindow的View，比如使用构造函数public PopupWindow (Context context)获得的Popupwindow就只能用setContentView来设置内容。
```java
PopupWindow popupWindow = new PopupWindow(context);
popupWindow.setContentView(contentview);
```

## 获得PopupWindow的视图内容
```java
public View getContentView()
```

## 判断是否在显示

```java
popupWindow.isShowing()
```

## 设置大小
有两种设置大小的方法：
### 调用有宽高参数的构造函数：
```java
LayoutInflater inflater = (LayoutInflater)getSystemService(Context.LAYOUT_INFLATER_SERVICE);  
View contentview = inflater.inflate(R.layout.popup_process, null);  
PopupWindow popupWindow = new PopupWindow(contentview,LayoutParams.WRAP_CONTENT,
LayoutParams.WRAP_CONTENT);  
```
### 通过setWidth和setHeight设置
```java
PopupWindow popupWindow = new PopupWindow(contentview);
popupWindow.setWidth(LayoutParams.WRAP_CONTENT);           
popupWindow.setHeight(LayoutParams.WRAP_CONTENT);
```

两种办法是等效的，不管采用何种办法，必须设置宽和高，否则不显示任何东西.  
这里的WRAP_CONTENT可以换成fill_parent 也可以是具体的数值，它是指PopupWindow的大小，也就是contentview的大小，注意popupwindow根据这个大小显示你的View，如果你的View本身是从xml得到的，那么xml的第一层view的大小属性将被忽略。相当于popupWindow的width和height属性直接和第一层View相对应。

## 显示位置
PopupWindow的位置按照有无偏移分，可以分为偏移和无偏移两种；按照参照物的不同，可以分为相对于某个控件（Anchor锚）和相对于父控件。具体如下

- showAtLocation()显示在指定位置，有两个方法重载：

	//相对于父控件的位置（例如正中央Gravity.CENTER，下方Gravity.BOTTOM等），可以设置偏移或无偏移
	public void showAtLocation(View parent, int gravity, int x, int y)
	//这个方法看不懂。。。
	public void showAtLocation(IBinder token, int gravity, int x, int y)
 

- showAsDropDown()显示在一个参照物View的周围，有三个方法重载：

	public void showAsDropDown(View anchor) //相对某个控件的位置（正左下方），无偏移
	
	public void showAsDropDown(View anchor, int xoff, int yoff) //相对某个控件的位置，有偏移
	
	//最后一种带Gravity参数的方法是API 19新引入的。
	public void showAsDropDown(View anchor, int xoff, int yoff, int gravity)


## 背景问题
先说结论：

- 设置了PopupWindow的background,点击Back键或者点击弹窗的外部区域,弹窗就会dismiss.

- 相反,如果不设置PopupWindow的background,那么点击back键和点击弹窗的外部区域,弹窗是不会消失的.

即使你设置了 setOutsideTouchable(true) ，但是没设置背景依然不行。要让点击PopupWindow之外的地方PopupWindow消失你需要调用setBackgroundDrawable(new BitmapDrawable());

如果不要背景，可以将背景设置为透明。
```java
setBackgroundDrawable(newColorDrawable(Color.TRANSPARENT));
```
为什么必须设置背景的解释：
> 如果有背景，则会在contentView外面包一层PopupViewContainer之后作为mPopupView，如果没有背景，则直接用contentView作为mPopupView。
而这个PopupViewContainer是一个内部私有类，它继承了FrameLayout，在其中重写了Key和Touch事件的分发处理

# 动画
很多时候我们把PopupWindow用作自定义的菜单，需要一个从底部向上弹出的效果，这就需要为PopupWindow添加动画。

方法：
```java
public void setAnimationStyle(int animationStyle)
```
在res/value/styles.xml添加一个sytle
```java
<style name="anim_menu_bottombar">
    <item name="android:windowEnterAnimation">@anim/menu_bottombar_in</item>
    <item name="android:windowExitAnimation">@anim/menu_bottombar_out</item>
</style>
```
在工程res下新建anim文件夹，在anim文件夹先新建两个xml文件
menu_bottombar_in.xml
```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
    <translate
        android:duration="250"
        android:fromYDelta="100.0%"
        android:toYDelta="0.0" />
</set>
```
menu_bottombar_out.xml
```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
    <translate
        android:duration="250"
        android:fromYDelta="0.0"
        android:toYDelta="100%" />
</set>
```
最后设置
```java
mPopupWindow.setAnimationStyle(R.style.menu_anim_bottombar);
```

# 用PopupWindow实现一个真正的模态对话框
在android中一个模态对话框应该是这样的：  
> 阻止屏幕上的其他View事件，且点击PopupWindow外面不会消失，但是能响应back事件（点击back消失），所以如果要让

PopupWindow有模态对话框的表现，则不能调用setBackgroundDrawable，因为setBackgroundDrawable会让点击PopupWindow外面PopupWindow消失。但是如果不调用setBackgroundDrawable则点击back键也不会消失，比模态对话框还变态。不过还是有解决的办法：
```java
LayoutInflater inflater = (LayoutInflater)getSystemService(Context.LAYOUT_INFLATER_SERVICE);
View contentview = inflater.inflate(R.layout.popup_process, null);
contentview.setFocusable(true); // 这个很重要
contentview.setFocusableInTouchMode(true);
final PopupWindow popupWindow = new PopupWindow(contentview,LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT);
popupWindow.setFocusable(true);
popupWindow.setOutsideTouchable(false);
contentview.setOnKeyListener(new OnKeyListener() {
    @Override
    public boolean onKey(View v, int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK) {
            popupWindow.dismiss();
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  
            return true;
        }
        return false;
    }
});
popupWindow.showAtLocation(view,  Gravity.CENTER|Gravity.CENTER_HORIZONTAL, 0, 0);
```

其中要注意的是PopupWindow 的视图contentview的设置，contentview.setFocusable(true);  这个很重要，注意不是PopupWindow的setFocusable，同时setOnKeyListener也是contentview调用的，而不是PopupWindow。setOnKeyListener方法中的代码很好理解，但是为啥是contentview调用就不是很清楚了，貌似没人去研究。其实为了是代码的相关性更低contentview.setOnKeyListener可以用popupWindow.getContentView.setOnKeyListener来代替。

话说如果PopupWindow需要如此这番周折才能实现一个模态对话框何不直接用Dialog呢？其实我只是将这些现象讲清楚，你可以根据这些知识将PopupWindow 应用在很苛刻的场景中。


# 参考链接
<http://www.cnblogs.com/mengdd/p/3569127.html>
<http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0702/1627.html>
