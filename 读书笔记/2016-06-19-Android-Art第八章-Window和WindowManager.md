---
layout:     post
title:      "Android-Art第八章-Window和WindowManager"
date:       2016-06-19 22:31:52
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



# 简介
`Window`是抽象类，具体实现是`PhoneWindow`，通过`WindowManager`就可以创建`Window`。`WindowManager`是外界访问`Window`的入口，但是`Window`的具体实现是在`WindowManagerService`中，`WindowManager`和`WindowManagerService`的交互是一个`IPC`过程。Android所有的视图例如`Activity`、`Dialog`、`Toast`都是附加在`Window`上的，所以说`Window`实际上是`View`的直接管理者。例如，单机事件由`Window`传递给`DecorView`，再由`DecorView`传递给我们的`View`；`setContentView`在底层也是通过`Window`来完成的。

<!--more-->


# 8.1 Window和WindowManager
如何使用`WindowManager`添加一个`Window`？下面代码将一个`Button`添加到屏幕坐标为(100,300)的位置上
```java
mFloatingButton = new Button(this);
mFloatingButton.setText("test button");
mLayoutParams = new WindowManager.LayoutParams(
LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT, 0, 0,
PixelFormat.TRANSPARENT);//0,0 分别是type和flags参数，在后面分别配置了
mLayoutParams.flags = LayoutParams.FLAG_NOT_TOUCH_MODAL
| LayoutParams.FLAG_NOT_FOCUSABLE
| LayoutParams.FLAG_SHOW_WHEN_LOCKED;
mLayoutParams.type = LayoutParams.TYPE_SYSTEM_ERROR;
mLayoutParams.gravity = Gravity.LEFT | Gravity.TOP;
mLayoutParams.x = 100;
mLayoutParams.y = 300;
mFloatingButton.setOnTouchListener(this);
mWindowManager.addView(mFloatingButton, mLayoutParams);
```
`WindowMangaer.LayoutParams`的`flags`和`type`参数说明如下：

- `Flags`参数表示`Window`的属性，通过这些属性可以控制`Window`的显示特性，常用的有下面几个：
    - `FLAG_NOT_FOCUSABLE`：表示`window`不需要获取焦点，也不需要接收各种输入事件。此标记会同时启用`FLAG_NOT_TOUCH_MODAL`，最终事件会直接传递给下层的具有焦点的`window`；
    - `FLAG_NOT_TOUCH_MODAL`：在此模式下，系统会将`window`区域外的单击事件传递给底层的`window`，当前`window`区域内的单击事件则自己处理，一般都需要开启这个标记，否则其他`Window`无法收到单击事件；
    - `FLAG_SHOW_WHEN_LOCKED`：开启此模式可以让`Window`显示在锁屏的界面上；

- `type`参数表`示window`的类型，window共有三种类型：应用window，子window和系统window。
  应用window对应着一个`Activity`，子window不能独立存在，需要附属在特定的父window之上，比如`Dialog`就是子window。系统window是需要声明权限才能创建的window，比如`Toast`和系统状态栏这些都是系统window，需要声明的权限是：
  `<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />`
  另外window是分层的，每个window都对应着z-ordered，层级大的会覆盖在层级小的上面，应用window的层级范围是1~99，子window的层级范围是1000~1999，系统window的层级范围是2000~2999。注意，应用window的层级范围并不是1~999哟。
  这些层级范围对应type参数。如果想要Window位于所有Window的最顶层，可以用系统Window，type指定为`TYPE_SYSTEM_OVERLAY`或者`TYPE_SYSTEM_ERROR`,注意没忘了添加权限，否则会报错。更详细见：Android应用开发之（WindowManager类使用）


`WindowManager`继承自`ViewManager`，常用的只有三个方法：
>
> a. addView(View view,ViewGroup.LayoutParams params)、
> b. updateViewLayout(View view,ViewGroup.LayoutParams params)
> c. removeView(View view)

`WindowManager`操作`Window`的其实就是操作Window中的View，例如：创建一个Window并向其添加View（如上面所示）；更新Window中的View；删除一个Window只需删除里面的View即可。

如要实现可以拖动的Window的效果只需根据手指的位置来设定`LayoutParams`中的x，y值即可改变Window的位置。首先给View设置`onTouchListener`，在`onTouch`方法中不断更新`View`的位置即可：
```java
@Override
public boolean onTouch(View v, MotionEvent event) {
    int rawX = (int) event.getRawX();
    int rawY = (int) event.getRawY();
    switch (event.getAction()){
        case MotionEvent.ACTION_MOVE:
            mLayoutParams.x = rawX;
            mLayoutParams.y = rawY;
            mWindowManager.updateViewLayout(button,mLayoutParams);
            break;
        default:
            break;
    }
    return false;
}
```

# 8.2 Window的内部机制
Window是一个抽象的概念，每个Window都对应着一个`View`和一个`ViewRootImpl`，Window和View通过`ViewRootImpl`来建立联系，所以说Window不是实际存在的，它也是以View的形式存在。在实际使用中无法直接访问Window，只能通过`WindowManager`才能访问`Window`。

## 8.2.1Window添加过程
`Window`的添加由`WindowManager`的实现类`WindowManagerImpl`来完成，`WindowManagerImpl`则是全部交给了`WindowManagerGlobal`来处理，`WindowManagerGlobal`以工厂形式向外提供自己的实例（`WindowManagerGlobal.getInstance()`），`WindowManagerGlobal`的`addView`方法主要分以下几步：
> 
1. 检查参数是否合法，如view是否为空，LayoutParams是否正确等。
2. 创建ViewRootImpl并将View添加到列表中
3. 通过ViewRootImpl来更新界面，完成Window的添加，这步由ViewRootImpl的setView方法来完成，setView内部会通过requestLayout来完成异步刷新请求。

## 8.2.2 Window的删除过程
和添加类似，删除也是先通过`WindowManagerImpl`后，再通过`WIndowManagerGlobal`来实现的。真正删除View的逻辑在`dispatchDetachedFromWindow`方法的内部实现。`dispatchDetachedFromWindow`方法主要做四件事：
> 
1. 垃圾回收的工作，比如清除数据和消息，移除回调。
2. 通过Session的remove方法删除Window，mWindowSession.remove(mWindow)，这同样是一个IPC过程，最终会调用WindowManagerService的removeWindow方法
3. 调用View的dispatchDetachedFromWindow方法，在内部调用View的onDetachedFromWindow()以及onDetachedFromWindowInternal()。
4. 调用WindowManagerGlobal的doRemoveView方法刷新数据，包括mRoots、mParams以及mDyingViews，需要将当前Window所关联的这三类对象从列表中删除。

## 8.2.3Window的更新过程
首先需要更新`View`的`LayoutParams`并替换掉老的`LayoutParams`，接着再更新`ViewRootImpl`中的`LayoutParams`，这一步是通过`ViewRootImpl`的`setLayoutParams`方法来实现的。在`ViewRootImpl`中会通过`scheduleTrversals`方法来对`View`重新布局，包括测量、布局、重绘三个过程。除了`View`本身的重绘以外，`ViewRootImpl`还会通过`WindowSession`来更新`Window`的视图，这个过程最终是由`WindowManagerService`的`relayoutWindow()`来具体实现的，同样是一个IPC过程。
更多源码分析见书P297—P304

# 8.3 Window的创建过程
`View`是Android中视图的呈现方式，但`View`不能单独存在，必须附在`Window`这个抽象概念上面，所以有视图的地方就有`Window`，那视图有哪些？Activity、Dialog、Toast、PopupWindow、菜单等。他们都分别对应着一个Window。

## 8.3.1 Activity的Window创建过程
首先要了解`Activity`的创建过程，`Activity`的启动过程很复杂，最终会由`ActivityThread`中的`performLaunchActivity`来完成整个启动过程，在这个方法内部会通过类加载器创建`Activity`的实例对象，并调用它的`attach`方法为其关联运行过程中所依赖的一系列上下文环境变量；
​	
在`Activity`的`attach`方法里，系统会通过`PolicyManager`的`makeNewWindow`方法创建出`Activity`所属的`Window`对象并为其设置回调接口，当`Window`接受到外界的状态发生改变时就会回调`Activity`的方法。回调接口中有很多方法，如`onAttachedToWindow、onDetachedFromWindow、dispatchTouchEvent`等。
​	
`PolicyManager`的实现类是`Policy`，它的`makeNewWindow`方法返回的是`PhoneWindow`对象，可以看出`Activity`对应的Window对象是`PhoneWindow`。

`Activity`的视图是如何附属到`PhoneWindow`上的？就要看`PhoneWindow的setContentView()`方法了，大致分以下几个步骤：
> 
1. 如果没有DecorView，就创建它，DecorView是Activity的顶级View，是一个FrameLayout。DecorView的创建由installDecor方法来完成，内部又通过generateDecor方法直接new出DecorView，这是的DecorView是一个空白的FrameLayout。然后PhoneWindow通过generateLayout方法来加载具体的布局文件到DecorView中，这个布局文件和系统版本和主题有关。
2. 将Activity的视图添加到DecorView的mContentParent中：mLayoutInflater.inflate(layoutResID,mContentParent)
3. 回调Activity的onContentChanged方法通过Activity视图发生了改变。

经过上面三个步骤，`DecorView`已经被创建并初始化完毕，`Activity`的布局文件也添加到了`DecorView`的`mContentParent`中，但这个时候的`DecorView`还没有被正式添加到`Phonewindow`中，因为这个`DecorView`还没有被`WindowManager`识别，这时候的`Window`无法提供功能，接受外界输入。在`ActivityThread`的`handleResumeActivity`中，会调用`Activity`的`onResume`方法，然后调用`makeVisible()`，在这个方法中`DecorView`才真正完成了添加和显示这两个过程，Activity的视图才能被用户看到。
```java
ViewManager vm = getWindowManager();
vm.addView(mDecor, getWindow().getAttributes());
mWindowAdded = true;
```
## 8.3.2 Dialog的创建过程
和上面类似，要注意的是必须采用`Activity`的`Context`，如果采用`Application`的`Context`会报错。原因是`Application`没有应用`token`，应用`token`一般是`Activity`拥有的。[service貌似也有token?]。另外系统`Window`的话就不需要`token`，所以弹系统Window的就可以用`Application`的`Context`了，别忘了权限：
```java
Dialog dialog = new Dialog(getApplicationContext());
TextView textView = new TextView(this);
textView.setText("This is dialog");
dialog.setContentView(textView);
dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ERROR);
dialog.show();
```
## 8.3.3 Toast的Window创建过程
`Toast`属于系统Window，它内部的视图由两种方式指定：一种是系统默认的演示；另一种是通过`setView`方法来指定一个自定义的View。

`Toast`具有定时取消功能，所以系统采用了`Handler`。Toast的显示和隐藏是IPC过程，都需要`NotificationManagerService（NMS`）来实现，在Toast和NMS进行IPC过程时，NMS会跨进程回调Toast中的TN类中的方法，TN类是一个`Binder`类，运行在Binder线程池中，所以需要通过Handler将其切换到当前发送Toast请求所在的线程，所以Toast无法在没有Looper的线程中弹出。

对于非系统应用来说，`mToastQueue`最多能同时存在50个`ToastRecord`，这样做是为了防止`DOS(Denial of Service，拒绝服务)`。因为如果某个应用弹出太多的Toast会导致其他应用没有机会弹出Toast。


​	
​	
​	
​	
​	
​	
​	
​	
​	
​	






