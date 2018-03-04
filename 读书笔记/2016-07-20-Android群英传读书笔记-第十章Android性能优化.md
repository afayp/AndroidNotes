---
layout:     post
title:      "Android群英传读书笔记-第十章Android性能优化"
date:       2016-07-20 23:12:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---


本章节将为大家展示几种性能优化的方法。

# 一.布局优化

<!--more-->

## 1.Android UI渲染机制
人眼所看到的流畅画面，需要画面的帧数达到40帧每秒到60帧每秒，相信玩过PC游戏的都应该对帧有一个详细的概念，最佳的ftp在60左右。在Android中，系统通过VSYNC信号出发对UI的渲染、重绘，其间隔时间是16ms。这个16ms其实就是1000ms中显示60帧画面的单位时间。即1000/60，如果系统每次渲染都保持在16ms之内，那么我们看到的UI将十分的流畅，但这也是需要将所有的逻辑都保证在16ms里，如果16ms不能完成绘制，那么就会造成丢帧的现象，即当前该重绘的帧被未完成的逻辑阻塞。例如一次任务耗时20ms，那么在16ms系统发出VSYNC信号时就无法绘制，该帧就被丢弃，等待下次信号才开始绘制，导致16*2ms内都是同一帧，这就是画面卡顿的原因。

Android系统提供了检测UI渲染时间的工具，打开“开发者选项”，选择“Profile GPU Rendering”（我的手机是“GPU呈现模式分析”），选中“On screen as bars”（我的为“在屏幕上显示为条形图”）。每一条柱状线都包括三部分，蓝色代表测量绘制Display List的时间，红色代表OpenGL渲染Display List所需要的时间，黄色代表CPU等待GPU处理的时间，中间绿色横线代表VSYNC时间16ms，需要尽量将所有条形图都控制在这条绿线之下。

## 2.避免Overdraw
Overdraw,过渡绘制会浪费很多CPU、GPU资源，例如系统默认会绘制Activity的背景，而如果再给布局绘制了重叠的背景，那么默认Activity的背景就属于无效的过渡绘制。Android系统在开发者选项中提供了这样一个检测工具–“Enable GPU Overdraw”。借助它可以判断Overdraw的次数。尽量增大蓝色的区域，减少红色的区域,这里，我们用一张Google开发者博客上的图片来表示

![](http://oeiu2t0ur.bkt.clouddn.com/20160430134934006.jpg)

通过这个工具可以查看当前区域中绘制的次数，从而尽量优化绘图层次，尽量增大蓝色的区域，减少红色的区域。

## 3.优化布局层级
在Android中系统对View的测量、布局和绘制都是通过遍历View树来进行的，如果View树太高，就会影响其速度，因此优化布局的第一个方法就是减低view树的高度，Google也在其api文档中建议view树的结构不宜超过十层
不知道是否读者有注意到，在早期的Android版本中，Google使用线性布局作为默认布局，而在现在的Android使用的是相对布局（默认），原因就是通过扁平的相对布局来降低通过线性布局所产生的树的高度，从而提高布局的效率

## 4.避免嵌套过多无用的布局
嵌套的布局会让view树的高度越来越高，所以在布局中，需要根据自身布局的特点，来选择不同的layout组件，从而避免通过某一种layout组件来实现功能时的局限性，从而造成嵌套过多的情况发生。

### 使用< include>标签重用布局
一些共同的UI，比如toolbar什么的，我们就可以使用`<include>`标签重用布局
```java
<include
    layout="@layout/toolbar"
    android:layout_width="100dp"
    android:layout_height="50dp" />
```
有一点要注意的是，如果你要在`<inclde>`标签中覆盖类似原布局中的`android:layout_XXX`属性，就必须在<include>标签中同时指定`android:layout_width和android:layout_height`属性。

### 使用< ViewStud>实现view的延迟加载
```java
<ViewStub
    android:id="@+id/viewStub"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_centerInParent="true"
    android:layout="@layout/activity_main" />
```
用viewStub延迟加载不太常用的布局（与Gone相比，加载时没有添加到布局树中）

如何显示？
首先找到控件：` ViewStub viewStub = (ViewStub) findViewById(R.id.viewStub);`
接下来有两种方式：
• VISIBLE—— `viewStub.setVisibility(View.VISIBLE);`
• inflate——`View inflateView = viewStub.inflate();`
两种方式唯一的区别是inlalte方法可以返回引用的布局，从而可以再通过`View.findViewById()`方法来找到子控件。
不管使用哪种方式，一旦`<ViewStub>`被设置为了可见或是inflate了，`<ViewStub>`就不存在了，取而代之的是被inflate的Layout，所以第二次inflate会报错。

此外可以通过设置`android:inflatedId="@+id/root_id"`属性，把layout对应布局的根元素设置为这个id。

## 5.Hierarchy Viewer
该工具位于sdk/tools目录下，进入该目录，在命令行中输入hierarchyviewer.bat，就可以启动程序了。如图：
![](http://oeiu2t0ur.bkt.clouddn.com/5522.png)

然后选择要调试的进程，然后点击上面的load view Hierarchy按钮
![](http://oeiu2t0ur.bkt.clouddn.com/20160430135018944.png)

我们使用多层的线性布局，这个代码是多余的，通常情况下，我们关心ID为content的分支，这是setvontenview所设置的内容
![](http://oeiu2t0ur.bkt.clouddn.com/20160430135028447.png)
这里我们就可以看到我们的线性布局
当点击一个view的时候，可以显示改view的绘制情况，不过第一次显示的时候，各种显示时间都是NA，需要点击菜单中的“profile node”重新计算
此时就可以知道每个view所绘制的时长，并且在系统的下方给出三个不同颜色的小圆点，用来表示绘制的效率，绿黄红，好，中，差
通过Hierarchy工具就可以很快的找到视图树上多余的布局了，从而有目的去优化布局了

# 二.内存优化

## 1.什么是内存
由于Android的沙箱机制,每个应用所分配的内存大小是有限度的,内存太低就会触发`LMK-Low Memory Killer`机制。那么到底什么是内存呢?通常情况下我们所说的内存是指手机的RAM,它包括以下几个部分
> 
• 寄存器(Registers)
速度最快的存储场所,因为寄存器位于处理器内部.在程序中无法控制
• 栈（Stack）
存放基本类型的数据和对象的引用.但对像本身不存放在栈中,而是存放在堆中
• 堆内存（Heap）
堆内存用来存放由new创建的对象和数组.在堆中分配的内存，由java虚拟机的自动垃圾回收（GC）管理
• 静态存储区域(static Field)
静态存储区域就是指在固定的位置存放应用程序运行时一直存在的数据，java在内存中专门划分了一个静态存储区域来管理一些特殊的数据变量如静态的数据变量
• 常量池(Constant Pool)
JVM虚拟机必须为每个被装载的类型维护一个常量池，常量池就是该类型所用到常量的一个有序集合，包括直接常量(基本类型，String)和对其他类型. 字段和方法的符号引用

在这些概念中最容易搞错的就是堆和栈的区分。当定义一个变量，Java虚拟机就会在栈中为该变量分配内存空间 这部分内存空间会马上被用作新的空间进行分配，如果使用new的方式创建一个变量, 那么就会在堆中为这个对象分配内存空间，即使该时象的作用域结束, 这部分内存也不会立即被回收.而是等待系统Gc进行回收，堆的大小随着手机的不断发展而不断变大，可以使用如下所示的代码来获得堆的大小，所谓的内存分析,正是分析Heap中的内存状态。
```java
ActivityManager manager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
int heapSize = manager.getLargeMemoryClass();
```	
## 2.获取Android系统内存信息
### Process Stats
Process Stats是KK上新增的一个系统内存监视服务. 可以通过’Setting-Developer options-Process Stats来开启该功能（我的在开发者选项—进程统计分析）
![](http://oeiu2t0ur.bkt.clouddn.com/20160430135044947.png)

同样也可以使用Dumpsys命令来获取这些信息
`agb shell dumpsys procstats`

### Meminfo
Meminof也是系统的一个非常重要的内存监视工具，可以通过setting-apps-running打开，如图
![](http://oeiu2t0ur.bkt.clouddn.com/20160430135056432.png)

同样，我们也可以通过adb命令来实现
`agb shell dumpsys meminfo`

### 内存回收
java虽然不用手动管理系统资源，但是java的gc是系统自动进行的，开发者无法控制的，即使调用system.gc方法,也只是建议系统进行GC,但是系统是否采纳你的建设.那就不一定了。JVM虚拟机虽然能够自动控制GC,但是再强大的算法，也难免会存在部分对象忘记回收的现象发生, 这就是造成内存泄漏的原因。

### 内存优化实例
分别从Bitmap和代码两个角度来对内存进行优化。

#### Bitmap
Bitmap是造成内存占用过高甚至是OOM的最大威胁,下面给出一些使用Bitmap的技巧
> 
• 使用适当分辨率和大小的图片：例如在图片列表界面可以使用的缩略图thumbnails,而在显示详细图片的时候在显示原图
• 及时回收： 一旦用完Bitmap后,一定要及时使用bitmap.recycle()方法来释放资源，自Android3.0后，由于bitmap被放置到了堆内存由gc管理.就不需要释放了（？）
• 使用图片缓存： 通过内存缓存(LrcCache)和硬盘缓存(DiskLrcCache) 可以更高的使用bitmap

#### 代码
任何Java类,都将占用大约500字节的内存空间，创建一个类的实例会消耗大约15子节的内存。从代码的实现方式上,也可以对内存进行优化 这里同样总结了一些小的技巧
> 
• 对常量使用static修饰符
• 使用静态方法,静态方法会比普通方法提高15%左右的访问速度
• 减少不必要的成员变量 ,这点在Android Lint工具上已经集成检测了.如果一个变量可以定义为局部变量，则会建议你不要定义为成成变量.
• 减少不必要的对象 使用基础类型会比使用对象更加节省资源, 同时更应该避免频繁创建短作用域的变量
• 尽量不要使枚举、少使用迭代器。
• 对Cursor,Receiver，Sensor,Fiie等对象,要非常注意对它们的创建.回收与注册解注册。
• 避免使用IOC框架.IOC通常使用注解.反射来进行实现,虽然现在java对反射的效率已经进行了很好的优化.但大量使用反射依然会带来性能的下降.
• 使用RenderScript，openGL来进行非常复杂的绘图操作.
• 使用surfaceView来替View进行大量.频繁的绘图操作.
• 尽量使用视图缓存，而不是每次都执行inflaler()方法解析视图

# 三.Lint工具
Android Lint工具是Android studio中继承的一个代码提示工具，他可以给你的布局，代码提供非常强大的帮助
![](http://oeiu2t0ur.bkt.clouddn.com/20160430135121964.png)

# 四.使用Android Studio的Memory Monitor工具
Memory Monitor工具是Android studio上的一个内存监视工具，他可以很好的帮助我们进行内存实时分析，通过点击Android studio右下角的Memory Monitor标签就可以进行查看了

# 五.使用TraceView工具优化APP性能
TraceView是一个Android下的可视化性能调查工具，它用来分析TraceView的日志

# 六.使用MAT工具分析APP内存状态

# 七.使用Dumpsys命令分析系统状态

使用Dumpsys命令可以列出Android系统相关的信息和服务状态，Dumpsys命令的功能非常强大,可使用的参数配置也非常多，关于Dumpsys的官方信息可以从如所示的网址来获取。
[https://source.android.com/devices/input/dumpsys.html](https://source.android.com/devices/input/dumpsys.html)

Dumpsys所支持的命令非常多，这里就不列举了

使用Dumpsys命令时.只需要输人`“adb Shell dumpsys +参数` 就可以，
比如：`adb shell dumpsys activity`就可以获取到acticity栈的信息

常用的Dumpsys参数：
![](http://oeiu2t0ur.bkt.clouddn.com/20160430144107935.png)