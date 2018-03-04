---
layout:     post
title:      "Android群英传读书笔记-第十二章Android5.X 新特性详解"
date:       2016-07-24 23:14:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



# 1. Android 5.X UI设计初步

1.材料的形态模拟
2.更加真实的动画
3.大色块的使用
此外，还有更多的设计风格，比如悬浮按钮，聚焦大图，无框按钮，波纹效果等新特性，这里就不一一列举了。

<!--more-->
# 2.Material Design主题
我们先来看看如何使用主题，MD一共有三种默认的主题可以设置
> 
@android:style/Theme.Material (dark version)        
@android:style/Theme.Material.Light (light version)     
@android:style/Theme.Material.Light.DarkActionBar   

# 3.Palette
Android5.X创新的使用了Palette来提取颜色，从而让主题能够动态适应当前页面的色调，使得整个app的颜色基本和谐统一

Android内置了几种提取颜色的种类:
> 
Vibrant（充满活力的）
Vibrant dark（充满活力的黑）
Vibrant light（充满活力的白）
Muted(柔和的)
Muted dark(柔和的黑)
Muted light(柔和的白)

# 4.视图与阴影
Material Design的一个很重要的特点就是拟物扁平化，如果说IOS的扁平化设计太过于超前，让很多人来不及从拟物转变成扁平，那么Material Design则是把IOS往回拉了一点，通过展现生活中的材料效果，恰当的使用阴影和光线，配合完美的动画效果，模拟出一个动感十足又美丽大胆的视觉效果。

以往的Android View通常有两个属性——X和Y，而在Android5.X中，Google为其增加了一个新的属性——Z，对应垂直方向上的高度变化。
View的Z值由两部分组成，`elevation`和`translationZ`(他们都是Android5.X新引入的属性)，`elevation`是静态的成员，`translationZ`可以在代码中使用来实现动画效果，他们的关系
	`Z = elevation + translationZ;`
在XML中可以通过 `android:elevation="2dp"`来设置elevation的值。
例如下面的例子：
```java
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="vertical">
	 
	    <TextView
	        android:layout_width="100dp"
	        android:layout_height="100dp"
	        android:layout_margin="10dp"
	        android:background="@mipmap/ic_launcher" />
	 
	    <TextView
	        android:layout_width="100dp"
	        android:layout_height="100dp"
	        android:layout_margin="10dp"
	        android:background="@mipmap/ic_launcher"
	        android:elevation="2dp" />
	 
	    <TextView
	        android:layout_width="100dp"
	        android:layout_height="100dp"
	        android:layout_margin="10dp"
	        android:background="@mipmap/ic_launcher"
	        android:elevation="10dp" />
	</LinearLayout>
```
效果：
![](http://oeiu2t0ur.bkt.clouddn.com/20160430215158885.png)

在程序中，我们也可以使用代码改变视图高度
	 `view.setTranslationZ(xxx);`
通常也会使用属性动画来为视图高度改变的时候增加动画效果
```java
if(flag){
    view.animate().translationZ(100);
    flag = false;
}else {
    view.animate().translationZ(0);
    flag = true;
}
```

# 5.Tinting 和 Clipping
Android5.X的两个对操作图像的新功能——Tinting（着色）和Clipping（裁剪）
## Tinting（着色）
	Tinting的使用非常的简单，只要在XML中配置好tint和tintMode就可以了，下面设置了几种不同的tint和tintMode效果，XML代码如下
```java
		<?xml version="1.0" encoding="utf-8"?>
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		    xmlns:tools="http://schemas.android.com/tools"
		    android:layout_width="match_parent"
		    android:layout_height="match_parent"
		    android:orientation="vertical">
		 
		    <ImageView
		        android:layout_width="100dp"
		        android:layout_height="100dp"
		        android:layout_gravity="center"
		        android:elevation="5dp"
		        android:src="@mipmap/ic_launcher" />
		 
		    <ImageView
		        android:layout_width="100dp"
		        android:layout_height="100dp"
		        android:layout_gravity="center"
		        android:elevation="5dp"
		        android:src="@mipmap/ic_launcher"
		        android:tint="@android:color/holo_blue_bright" />
		 
		    <ImageView
		        android:layout_width="100dp"
		        android:layout_height="100dp"
		        android:layout_gravity="center"
		        android:elevation="5dp"
		        android:src="@mipmap/ic_launcher"
		        android:tint="@android:color/holo_blue_bright"
		        android:tintMode="add" />
		 
		 
		    <ImageView
		        android:layout_width="100dp"
		        android:layout_height="100dp"
		        android:layout_gravity="center"
		        android:elevation="5dp"
		        android:src="@mipmap/ic_launcher"
		        android:tint="@android:color/holo_blue_bright"
		        android:tintMode="multiply" />
		</LinearLayout>
```
效果：
![](http://oeiu2t0ur.bkt.clouddn.com/20160430220325791.png)

Tint通过修改图像的Alpha遮罩来修改图像的颜色，从而达到重新着色的目的，这一功能在一些图片处理的APP使用起来还是十分的方便的。

## Clipping（裁剪）

Clipping可以让我们改变一个视图的外形，要使用Clipping，首先需要使用`ViewOutlineProvider`来修改outline作用给视图。
下面这个例子，将一个正方形的textview通过Clipping裁剪成一个圆形的正方形和一个圆，XML代码如下:
```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical">
 
    <TextView
        android:id="@+id/tv_rect"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="20dp"
        android:elevation="1dp"
        android:gravity="center" />
 
    <TextView
        android:id="@+id/tv_circle"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="20dp"
        android:elevation="1dp"
        android:gravity="center" />
</LinearLayout>
```
Java代码很简单：
```java
private void setClipping() {
    final View v1 = findViewById(R.id.tv_rect);
    View v2 = findViewById(R.id.tv_circle);
 
    //获取Outline
    ViewOutlineProvider vlp1 = new ViewOutlineProvider() {
        @Override
        public void getOutline(View view, Outline outline) {
            //修改outline为特定形状
            outline.setRoundRect(0, 0, view.getWidth(), view.getHeight(), 30);
        }
    };
 
    //获取Outline
    ViewOutlineProvider vlp2 = new ViewOutlineProvider() {
        @Override
        public void getOutline(View view, Outline outline) {
            //修改outline为特定形状
            outline.setOval(0, 0, view.getWidth(), view.getHeight());
        }
    };
    //重新设置形状
    v1.setOutlineProvider(vlp1);
    v2.setOutlineProvider(vlp2);
}
```
效果： 
	![](http://oeiu2t0ur.bkt.clouddn.com/55476.png)

# 6.列表和卡片
> RecyclerView和CardView
	
# 7.Activity过渡动画
曾经的Android在activity进行跳转的时候，只是很生硬的切换，即使通过 `overridePendingTransition( int inId, int outId)`这个方法来给Activity增加一些切换动画，效果也只是差强人意，而在Android5.X中提供了三种`Transition`类型
> 
• 进入：一个进入的过渡动画决定Activity中的所有视图怎么进入屏幕
• 退出：一个退出的过渡动画决定Activity中的所有视图怎么退出屏幕
• 共享元素：一个共享元素过渡动画决定两个Activity之间的过渡，怎么共享他们的视图
	
其中，进人和退出效果包括
> 
• explode(分解)一一从屏幕中间进或出,移动视图
• slide(滑动)——从屏幕边缘进或出，移动视图
• fade(淡出) 一一通过改变屏幕上视图的不透明度达到添加或者移除视图

共享元素包括
> 
• changeBounds——改变目标视图的布局边界
• changeCliBounds——裁剪目标视图边界
• changeTransfrom——改变目标视图的缩放比例和旋转角度
• changeImageTransfrom——改变目标图片的大小和缩放比例

首先来看看普通的三种Activity过渡动画， 要使用这些动画非常简单，例如从ActivityA转到ActivityB，只需要在ActivityA中将基本的`startActivity(intent)`方法改为如下代码即可：
```java
//可以使用ActivityOptionsCompat兼容到API16
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(MainActivity.this).toBundle());
```
在AchvityB中，只需要设置下如下所示代码
```java
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
```
或者在样式文件中设置如下所示代码
```java
 <item name="android:windowContentTransitions">true</item>
```
接下来就可以在ActivityB中设置进人ActivityB的具体的动画效果了， 代码如下所示（必须放在setContentView前面？）
```java
getWindow().setEnterTransition(new Explode());
getWindow().setEnterTransition(new Slide());
getWindow().setEnterTransition(new Fade());
```
以及离开ActivityB时的动画：
```java
getWindow().setExitTransition(new Explode());
getWindow().setExitTransition(new Slide());
getWindow().setExitTransition(new Fade());
```
下面说说**共享元素**动画。所谓的共享元素就是Activity1和Acitvity2都拥有的元素，共享元素转场动画就是指这个都拥有的元素如何从A中过渡显示到B中。
要想在程序中使用共享元素的动画效果也很简单，首先需要在他的activity1布局中设置共享的元素，给他增加相应的属性。
	`android:transitionName="XXX"`
同时在activity2中，给共享元素也增加相同的属性，跟上面一样，注意要保证这两个名字一样，这样系统才能找到这个元素。
	`android:transitionName="XXX"`
如果只要一个共享元素，那么在activity1中可以这样写
```java	
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this,view,"share").toBundle());
```
使用的参数就是前面普通动画的基础上增加了共享的的view和前面取的名字，如果有多个共享元素，那么我们可以通过 `Pair.create()`来创建多个共享元素,然后
```java
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this, Pair.create(view,"share"),Pair.create(fab,"fab")).toBundle());
```
# 7.Material Design动画效果

## 1.Ripple效果
在Android5.X中，Material Design大量的使用了Ripple动画，即点击后的波纹效果，可以通过如下代码设置波纹的背景
```java
//波纹有边界
android:background="?android:attr/selectableItemBackground"
//波纹无边界
android:background="?android:attr/selectableItemBackgroundBorderless"
```
波纹有边界是指波纹被限制在控件的边界中（四方形），而波纹超出边界，则是不会受到控件的限制，以圆形发散出去。
我们也可以在XML文件中直接创建一个具有Ripple效果的XML文件：
```java
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@android:color/holo_blue_dark">
    <item>
        <shape android:shape="oval">
            <solid android:color="@color/colorPrimary" />
        </shape>
    </item>
</ripple>
```
把它设置给background即可使用。

## 2.Circular Reveal
这个动画效果是在Google I/O 大会上演示了好多次的，具体表现为一个View以圆形的形式展开，揭示出来，通过`ViewAnimationUtils.createCircularReveal()`来创建动画，代码如下：
```java
public static Animator createCircularReveal(View view, int centerX, int centerY, float startRadius, float endRadius) {
    return new RevealAnimator(view,centerX,centerY,startRadius,endRadius);
}
```
RevealAnimator的使用特别简单，主要就是几个关键的坐标点
> 
• centerX 动画开始的中心点X
• centerY 动画开始的中心点Y
• startRadius 动画开始半径
• endRadius 动画结束半径

例子：
```java
final View oval = findViewById(R.id.oval);
    oval.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Animator animator = ViewAnimationUtils.createCircularReveal(oval, oval.getWidth() / 2, oval.getHeight() / 2, oval.getWidth(), 0);
            animator.setInterpolator(new AccelerateDecelerateInterpolator());
            animator.setDuration(2000);
            animator.start();
        }
    });
 
 
    final View rect = findViewById(R.id.rect);
    rect.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Animator animator = ViewAnimationUtils.createCircularReveal(rect, 0, 0, 0, (float) Math.hypot(rect.getWidth(), rect.getHeight()));
            animator.setInterpolator(new AccelerateInterpolator());
            animator.setDuration(2000);
            animator.start();
        }
    });
```	
## 3. View state changes Animation
在Android5.X中，系统提供了视图与状态改变来设置一个视图状态的切换

### StaetListAnimator 
`StaetListAnimator`作为视图改变时的动画效果，通常会使用`Seletor`来进行设置，但是以前我们设置`Seletor`的时候，通常是修改他的背景来达到反馈的效果，但是再现在Android5.X中就不需要这样了，可以使用动画来实现，我们用小例子来具体看看怎么实现的吧
在XML中定义一个StaetListAnimator：
```java
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <set>
            <objectAnimator android:duration="2000" android:property="rotationX" android:valueTo="360" android:valuyeType="floatType" />
        </set>
    </item>
 
    <item android:state_pressed="false">
        <set>
            <objectAnimator android:duration="2000" android:property="rotationX" android:valueTo="0" android:valuyeType="floatType" />
        </set>
    </item>
</selector>
```
然后在XML中使用：
```java
<Button
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:stateListAnimator="@drawable/anim_change" />
```	
### animated-selector 
	animated-selector同样是一个改变动画效果的动画,例如我们熟悉的checkbox的动画效果就是通过这个实现的，要实现这个效果需要一组状态切换的图，具体使用看书P289。


# 9.Toolbar

# 10.Notification

