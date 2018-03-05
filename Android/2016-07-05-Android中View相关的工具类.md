---
layout:     post
title:      "Android View相关工具类"
date:       2016-7-5 18:59:25
author:     "afayp"
catalog:    true
tags:
    - Android
---



> Android中View相关的工具类介绍

# Configuration

Configuration用来描述设备的配置信息,比如：

- 用户的配置信息：locale和scaling等等 
- 设备的相关信息：输入模式，屏幕大小， 屏幕方向等等 

<!--more-->

我们经常采用如下方式来获取需要的相关信息:
```java
	Configuration configuration = getResources().getConfiguration();
	//国家码
	int countryCode = configuration.mcc;
	//网络码
	int networkCode = configuration.mnc;
	//判断横竖屏
	if(configuration.orientation == configuration.ORIENTATION_LANDSCAPE){
	
	}else {
	
	}
```


# ViewConfiguration 
ViewConfiguration提供了一些自定义控件用到的标准常量，比如尺寸大小，滑动距离，敏感度等等。 
可以利用ViewConfiguration的静态方法获取一个实例
`ViewConfiguration viewConfiguration = ViewConfiguration.get(this);`
常用的几个对象方法：
```
//获取touchSlop。该值表示系统所能识别出的被认为是滑动的最小距离
int scaledTouchSlop = viewConfiguration.getScaledTouchSlop();
//获取Fling速度的最小值和最大值
int scaledMaximumFlingVelocity = viewConfiguration.getScaledMaximumFlingVelocity();
int scaledMinimumFlingVelocity = viewConfiguration.getScaledMinimumFlingVelocity();
//判断是否有物理按键
boolean isHavePermanentMenuKey = viewConfiguration.hasPermanentMenuKey();
```
常用的几个静态方法：
```java
//双击间隔时间.在该时间内是双击，否则是单击
int doubleTapTimeout = ViewConfiguration.getDoubleTapTimeout();
//按住状态转变为长按状态需要的时间
int longPressTimeout = ViewConfiguration.getLongPressTimeout();
//重复按键的时间
int keyRepeatTimeout = ViewConfiguration.getKeyRepeatTimeout();
```

# GestureDetector
我们可以在onTouchEvent()中自己处理手势，但有些复杂手势onTouchListener处理不了，其实Android系统也给我们提供了一个手势处理的工具，这就是GestureDetector手势监听类。利用GestureDetector可以简化许多操作，轻松实现一些常用的功能。
常见的手势：

- 按下（onDown）： 刚刚手指接触到触摸屏的那一刹那，就是触的那一下。
- 抛掷（onFling）： 手指在触摸屏上迅速移动，并松开的动作。
- 长按（onLongPress）： 手指按在持续一段时间，并且没有松开。
- 滚动（onScroll）： 手指在触摸屏上滑动。
- 按住（onShowPress）：手指按在触摸屏上，它的时间范围在按下起效，在长按之前。单击了还未松开或拖动
- 抬起（onSingleTapUp）：手指离开触摸屏的那一刹那。
- 双击（onDoubleTap）：两次连续的单击，双击的第二下down时触发
- 双击（onDoubleTapEvent）：表示发生了双击的行为，双击的第二下down和up都会触发，可用e.getAction()区分。

说明：
onDown，onLongPress这两者相比较， onDown只要Touch down一定立刻触发。 而Touch down后过一会没有滑动先触发onShowPress再是onLongPress。
所以Touch down后一直不滑动，会按照```onDown->onShowPress->onLongPress```这个顺序触发。

使用：
首先得到GestureDetector对象，实现OnGestureListener接口：
```java
GestureDetector gestureDetector = new GestureDetector(this, new GestureDetector.OnGestureListener() {
	
	    //触摸屏幕时均会调用该方法
	    @Override
	    public boolean onDown(MotionEvent e) {
	        return false;
	    }
	
	    //手指在屏幕上按下,且未移动和松开时调用该方法
	    @Override
	    public void onShowPress(MotionEvent e) {
	
	    }
	
	    //轻击屏幕时调用该方法
	    @Override
	    public boolean onSingleTapUp(MotionEvent e) {
	        return false;
	    }
	
	    //手指在屏幕上滚动时会调用该方法
	    @Override
	    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
	        return false;
	    }
	
	    //手指长按屏幕时均会调用该方法
	    @Override
	    public void onLongPress(MotionEvent e) {
	
	    }
	
	    //手指在屏幕上拖动时会调用该方法
	    @Override
	    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
	        return false;
	    }
	});
```
一般用```SimpleOnGestureListener```这个类，可以选择性的实现想要的方法，而不用全部实现。

然后将Touch事件交给GestureDetector处理。
比如将Activity的Touch事件交给GestureDetector处理（在Activity中重写onTouch方法）
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    return gestureDetector.onTouchEvent(event);
}
```
比如将View的Touch事件交给GestureDetector处理:
```java
mButton=(Button) findViewById(R.id.button);  
mButton.setOnTouchListener(new OnTouchListener() {            
   @Override  
   public boolean onTouch(View arg0, MotionEvent event) {  
          return mGestureDetector.onTouchEvent(event);  
      }  
});  
```
又或者在自定义view中设置手势：
```java
public class GestureView extends View{
    private GestureDetector mDetector;  

    public GestureView(Context context, AttributeSet set){
        super(context, set);
        mDetector = new GestureDetector(context, new MyGestureListener());
        setLongClickable(true);
        this.setOnTouchListener(new OnTouchListener(){
        @Override
        public boolean onTouch(View v, MotionEvent event){
            return mDetector.onTouchEvent(event);
        }
        });
    }
}
```
在View中设置手势有两点需要注意：
> 
 1. View必须设置longClickable为true，否则手势识别无法正确工作，只会返回Down, Show, Long三种手势。
 2. 必须在View的onTouchListener中调用手势识别，而不能像Activity一样重载onTouchEvent，否则同样手势识别无法正确工作。

# VelocityTracker
速度追踪，用于追踪屏幕触摸事件中手指在滑动过程中的速度，包括水平和竖直两个方向的速度。
使用方法：
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    //追踪当前点击事件的速度
    VelocityTracker velocityTracker = VelocityTracker.obtain();
    velocityTracker.addMovement(event);

    //计算出速度,参数为事件间隔，表示在这段时间内滑过的像素数。
    velocityTracker.computeCurrentVelocity(1000);

    //获取速度
    float xVelocity = velocityTracker.getXVelocity();
    float yVelocity = velocityTracker.getYVelocity();
    
    //最后要记得回收
    velocityTracker.clear();
    velocityTracker.recycle();

    return super.onTouchEvent(event);
}
```
# ViewDragHelper
参考[Android ViewDragHelper完全解析 自定义ViewGroup神器](http://blog.csdn.net/lmj623565791/article/details/46858663)
# Scroller