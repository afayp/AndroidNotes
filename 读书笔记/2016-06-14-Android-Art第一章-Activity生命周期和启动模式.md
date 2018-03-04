---
layout:     post
title:      "Android-Art第一章-Activity生命周期和启动模式"
date:       2016-06-14 18:59:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



# 生命周期分析
## 典型情况下
什么是典型情况下?activity受用户操作所导致的正常的生命周期方法的调度

一张图概括：

<!--more-->

![](http://images2015.cnblogs.com/blog/424424/201601/424424-20160111134839507-1925938998.png)

再来一张：
![](http://hukai.me/android-training-course-in-chinese/basics/activity-lifecycle/basic-lifecycle.png)

一些要注意的点：  

1. 当用户再次回到原来的Activity时，回调如下：onRestart -> onStart -> onResume。
2. 当用户打开新的Activity或者切换到桌面的时候（按Home键），回调如下：onPause -> onStop，但是如果新Activity采用了透明主题，那么onStop方法不会被回调。
3. 按back键：onPause——onStop——onDestory
4. 从Activity A进入到Activity B，回调顺序是onPause(A) -> onCreate(B) -> onStart(B) -> onResume(B) -> onStop(A)，（旧的Activity先pause，新的Activity再oncreate）所以不能在onPause方法中做重量级的操作（非要做的话也尽量在onStop中做~）,可以在onPause中停止动画；保存用户期待保存的数据如邮件草稿；释放系统资源如camera，broadcast ，receiver；尽量不要有io操作，否则导致打开下一个activity缓慢。在onStop中执行比如往数据库中读写的操作
5. onStart和onStop对应，它们是从Activity是否可见这个角度来回调的；onPause和onResume方法对应，它们是从Activity是否位于前台这个角度来回调的。这两组方法在实际使用中没有什么实质区别
6. 	上面第二张图，只有三个状态是静态的，这三个状态（resumed，paused,stopped）下activity可以存在一段比较长的时间,其它状态 (Created与Started)都是短暂的，系统快速的执行那些回调函数并通过执行下一阶段的回调函数移动到下一个状态。在系统调用onCreate(), 之后会迅速调用onStart(), 之后再迅速执行onResume()
7. 	入口activity：
```java
 	<intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
```
8. 进入Paused，不一定要Stopped; 进入Stopped, 不一定要Destory.注定要destroy的activity不代表不要保存数据，比如保存某个flag，这个flag会决定下次启动app显示的内容。
9. 重写生命周期方法时，一般都讲super.onXxx()放最前面
10. 我们在onStop里面做了哪些清除的操作，就该在onStart里面重新把那些清除掉的资源重新创建出来。
11. 我们会在onStop方法里面做释放资源的操作，那么onDestory方法则是我们最后去清除那些可能导致内存泄漏的地方。因此需要确保那些线程都被destroyed并且所有的操作都被停止。
12. 返回键、home键、电源键Activity生命周期执行顺序：
	1. 返回键当Activity处于运行状态时点击返回键，相应的代码执行顺序onPause()–>onStop()–>onDestory()；如果当前activity正同用户交互时点击返回键，相当于销毁了当前Activity。
	2. home键处于运行状态的Activity当用户点击home键时，相应的代码执行顺序onPause()–>onStop(),如果这时还没被当前系统回收销毁，用户重新进入应用，相应的代码执行顺序onRestart()–>onStart()–>onResume()；
	3. 电源键电源键同home键的执行顺序一致，但是有一点需要注意，就是如果点击电源键之后手机锁屏了，这时候回来重新点击电源键虽然用户没有开启锁屏，但是这时候已经执行了onResume()方法。。


## 异常情况下
异常情况分两种：资源相关的系统配置发生改变和系统内存不足导致activity被杀死，两种都属于异常情况

1. 系统在activity异常终止的时候会调用onSaveInstanceState存储数据（正常情况下不会被调用），它的调用时机是在onStop之前，它和onPause方法没有既定的时序关系，可能在它之前，也可能在它之后。(也有说是在onPause之前)
2. 在activity异常终止的时候，系统会默认为我们保存当前Activity的视图结构，并且在Activity重启后为我们恢复这些数据，比如文本框中用户输入的数据、listview滚动的位置等，这些view相关的状态系统都会默认为我们恢复（不用自己去实现了）。具体针对某一个view系统能为我们恢复哪些数据可以查看view的源码中的onSaveInstanceState和onRestoreInstanceState方法
3. 当Activity被重新创建的时候，onRestoreInstanceState会被回调，它的调用时机是onStart之后。系统会把onSaveInstanceState方法所保存的Bundle对象作为参数同时传递给onCreate和onRestoreInstanceState。
4. onCreate和onRestoreInstanceState中都可以拿到之前保存的Bundle对象，二者的区别：
	1. onRestoreInstanceState一旦被调用，Bundle对象一定是有值的，我们不用额外的判断是否为空
	2. onCreate如果正常启动的话，其Bundle参数为null，所以必须额外判断
	3. 官方建议onRestoreInstanceState中去恢复数据
5. 当系统内存不足时会按下述优先级去杀死目标Activity所在的进程。如果一个进程没有四大组件在执行，那么这个进程很容易会被杀死。所以一般把后台工作放入Service中从而保证有一定的优先级
6. Activity优先级：
	1. 前台Activity——正在交互的
	2. 非前台Activity——比如弹出了一个对话框，Activity可见但是无法交互
	3. 后台Activity——不可见，执行了onStop
7. 系统配置改变后，如何设置不重新创建Activity？添加  
```java  
android:configChanges="orientation|keyboardHidden|screenSize"
```
8. 常用的属性：（更多见P14）  
	local：设备的本地位置发生了变化，一般指切换了系统语言；  
	keyboardHidden：键盘的可访问性发生了变化，比如用户调出了键盘；  
	orientation：屏幕方向发生了变化，比如旋转了手机屏幕。  
	screenSize：屏幕尺寸发生改变，比如旋转屏幕。当minSdkVersion和targetSdkVersion均低于13时，才不会导致Activity重启，否则会。（所以一般情况都会滴）
9. 配置了android:configChanges="xxx"属性之后，Activity就不会在对应变化发生时重新创建，而是调用Activity的 onConfigurationChanged 方法
10. 如果<activity>配置了android:screenOrientation属性，则会使android:configChanges="orientation"失效。

# 启动模式

## LanunchMode

1. 当任务栈中没有任何Activity的时候，系统就会回收这个任务栈。
2. 四种模式（android:launchMode="这里指定模式"）：  
	1. standrad：标准模式。每启动一个Activity都会重新创建一个实例，不管这个实例是否存在。在ActivityA中启动ActivityB（B是standrad模式），那么B就会运行在A所在的栈中。
		- 从非Activity类型的Context(例如ApplicationContext、Service等)中以standard模式启动新的Activity是不行的，因为这类context并没有任务栈，所以需要为待启动Activity指定FLAG_ACTIVITY_NEW_TASK标志位。其实也就是以singleTask模式启动
	2. singleTop：栈顶复用模式。如果栈顶有，调用其onNewIntent
	3. singleTask：栈内复用模式。默认有FLAG_ACTIVITY_CLEAR_TOP的效果。过程描述：系统会先寻找A想要的任务栈（见后面）。如果不存在就重新创建一个任务栈，然后把A的实例放到栈内；如果存在，如果这个栈内有A实例，就把它调到栈顶，并调用的onNewIntent方法，原先在A前面的实例全部出栈。如果不存在A实例，就创建A实例放入栈顶。另外，如果这个这个栈在后台则会被切换到前台。
	4. singleInstance：单实例模式。此种模式启动的Activity只能单独的位于一个任务栈中。

3. 任务栈分为前台任务栈和后台任务栈，后台任务栈中的Activity位于暂停状态，用户可以通过切换将后台任务栈再次调到前台。  

4. TaskAffinity，任务相关性，这个参数标识了一个Activity所需要的任务栈的名字。默认情况下，所有Activity所需的任务栈的名字为包名，不过我们可以为每个Activity单独制定这个属性。（直接在activity中指定taskAffinity属性）askAffinity属性主要和singleTask启动模式（FLAG_ACTIVITY_NEW_TASK）或者allowTaskReparenting属性配对使用，在其他情况下没有意义。
	1. 当TaskAffinity和singleTask启动模式配对使用的时候，它是具有该模式的Activity的目前任务栈的名字，待启动的Activity会运行在名字和TaskAffinity相同的任务栈中；
	2. 当TaskAffinity和allowTaskReparenting结合的时候，当一个应用A启动了应用B的某个Activity C后，如果Activity C的allowTaskReparenting属性设置为true的话，那么当应用B被启动后，系统会发现Activity C所需的任务栈存在了，就将Activity C从A的任务栈中转移到B的任务栈中。
5. 指定启动模式有两种方法：一种是通过AndroidMenifest指定属性android:launchMode，也可以使用代码intent.addFlags()。区别：
	1. 第二种优先级更高，两种同时存在时以后者为准。
	1. 限定范围不同，前者无法直接为Activity设置FLAG_ACTIVITY_CLEAR_TOP标识，而后者无法为Activity指定singleInstance模式。
6. adb shell dumpsys activity命令可以查看Running Activities；Recent Tasks等信息

## Flags
(intent.addFlags(...))
几种常用的：
1. FLAG_ACTIVITY_NEW_TASK,等同与singleTask
2. FLAG_ACTIVITY_SINGLE_TOP,singleTop
3. FLAG_ACTIVITY_CLEAR_TOP，singleTask默认有这种效果
4. FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS：具有这个标记的Activity不会出现在历史Activity列表中，当某些情况下我们不希望用户通过历史列表回到我们的Activity的时候这个标记比较有用，它等同于属性设置android:excludeFromRecents="true"。



# Activity启动方式
分显示和隐式两种。  
显示一般用于启动同一个应用下的Activity，隐式一般用在启动不同应用下的Activity。  
显示启动通过指定目标Activity的类名启动，隐式启动通过指的定Action动作启动指定的Activity。


## 显示启动
显示又有四种方式(用处不大。。)：

```java
// 启动方式一
 Intent intent = new Intent(this, SecondActivity.class);
 startActivity(intent);
 
// 启动方式二
 Intent intent = new Intent();
 intent.setClass(this, SecondActivity.class);
 startActivity(intent);
 
// 启动方式三
 ComponentName component = new ComponentName(this,
 SecondActivity.class);
 Intent intent = new Intent();
 intent.setComponent(component);
 startActivity(intent);
 
// 启动方式四
 Intent intent = new Intent();
 // 第二个参数必须是全类名 包名+类名
  intent.setClassName(this, "com.sunny.activity.SecondActivity");
// intent.setClassName("com.sunny.activity","com.sunny.activity.SecondActivity");
 startActivity(intent);
```
四种写法事实上是殊途同归，最终代码都转化为了方式三


## 隐式启动IntentFilter匹配规则
启动activity分两种：显示（要知道类名，包名）和隐式。一个Intent不应该即是显示又是隐式，如果这样的话，显示为主。隐式调用需要intent能够匹配目标activity在IntentFilter中所设置的过滤信息。IntentFilter中的过滤信息有action、category、data，为了匹配过滤列表，需要同时匹配过滤列表中的action、category、data信息，否则匹配失败。  
一个过滤列表中的action、category、data可以有多个，所有的action、category、data分别构成不同类别，同一类别的信息共同约束当前类别的匹配过程。只有一个Intent同时匹配action类别、category类别和data类别才算完全匹配，只有完全匹配才能成功启动目标Activity。此外，一个Activity中可以有多个intent-filter，一个Intent只要能匹配任何一组intenf-filter即可成功启动对应的Activity。（好复杂。。）
```java
<intent-filter>
    <action android:name="com.ryg.charpter_1.c" />
    <action android:name="com.ryg.charpter_1.d" />

    <category android:name="com.ryg.category.c" />
    <category android:name="com.ryg.category.d" />
    <category android:name="android.intent.category.DEFAULT" />

    <data android:mimeType="text/plain" />
</intent-filter>
```
### action匹配规则
action是一个字符串。系统定义了一些action，我们也可以自动以自己的action。Intent中的action存在（必须有）并且能够和过滤规则中的任何一个action相同即可匹配成功，另外，action匹配区分大小写。

### category匹配规则
category是一个字符串。系统定义了一些category，我们也可以自动以自己的category。
1. Intent中如果有category：那么所有的category都必须是过滤规则中已经定义了的category
2. Intent中没有category：如果没有，那么就是默认的category，即android.intent.category.DEFAULT，因为系统在调用startActivity或者startAcitvityForResult的时候会默认加上"android.intent.category.DEFAULT"，所以为了Activity能够接收隐式调用，配置多个category的时候必须加上默认的category。

### data匹配规则
1. data的语法：
```java
<data android:scheme="string"
android:host="string"
android:port="string"
android:path="string"
android:pathPattern="string"
android:pathPrefix="string"
android:mimeType="string" />
```    
主要由mimeType和URI组成，其中mimeType代表媒体类型，而URI的结构也复杂，如下：  
```java
<scheme>://<host>:<port>/[<path>]|[<pathPrefix>]|[pathPattern]

例如:content://com.example.project:200/folder/subfolder/etc
	 http://www.baidu.com:80/search/info
```

每个数据的含义：
	
	- Scheme：URI模式，比如http,file,content等，如果没有Scheme，那么整个URI就废了
	- Host：URI的主机名，比如：www.baidu.com，如果没有host，也废了
	- Port：URI的端口号
	- path：表示完整的路径信息
	- pathPrefix：也表示完整的路径信息，但里面可以包含通配符*，*表示任意个自负。如果要表示*要写成\\*。
	- pathPrefix：表示路径前缀信息
2. 规则：Intent中必须含有data数据，并且data数据能够完全匹配过滤规则中的某一个data。这里的完全匹配是指IntentFilter中的data部分（可能只指定了部分数据）也出现在了Intent的data中。分情况：

如果IntentFilter中是不完全定义的data,如下只指定了type
```java
<intent-filter>
    <data android:mimeType="image/*"/>
    ...
</intent-filter>
```
如上的情况，Intent的mimeType属性必须指定为"image/*"才能匹配。
要匹配上面的可以这么写intent：
```java
intent.setDataAndType(Uri.parse("file://abc"),"image/png")
```
另外，如果过滤规则中的mimeType指定为image/*或者text/*等这种类型的话，那么即使过滤规则中没有指定URI，URI有默认的scheme是content和file！如果过滤规则中指定了scheme的话那就不是默认的scheme了。
```java
//URI有默认值
<intent-filter>
    <data android:mimeType="image/*"/>
    ...
</intent-filter>
//URI默认值被覆盖
<intent-filter>
    <data android:mimeType="image/*" android:scheme="http" .../>
    ...
</intent-filter>
```
如果要为Intent指定完整的data，必须要调用setDataAndType方法！不能先调用setData然后调用setType，因为这两个方法会彼此清除对方的值。  
data的下面两种写法作用是一样的：
```java
<intent-filter>
    <data android:scheme="file" android:host="www.github.com"/>
</intent-filter>

<intent-filter>
    <data android:scheme="file"/>
    <data android:host="www.github.com"/>
</intent-filter>
```

IntentFilter的匹配规则对于Service和Broadcastreceiver也是一样的，不过建议尽量用显示调用方式来启动Service

### 如何判断是否有Activity能够匹配我们的隐式Intent？

1. PackageManager的resolveActivity方法或者Intent的resolveActivity方法：如果找不到就会返回null。通过返回值就可以判断，例如：
	```java
	ResolveInfo resolveInfo = getPackageManager().resolveActivity(intent, PackageManager.MATCH_DEFAULT_ONLY);
	if (resolveInfo != null) {
	    startActivity(intent);
	}
	```
2. PackageManager的queryIntentActivities方法：它返回所有成功匹配的Activity信息
3. MATCH_DEFAULT_ONLY 参数表示只匹配含有android.intent.category.DEFAULT的activity。如果不用这个标记位，那么可能匹配到category中没有DEFAULT的那些Activity，从而导致startActivity失败，因为不含DEFAULT的category的Activity不能隐式启动。反正用这个标志位就行了。
4. 针对Service和BroadcastReceiver等组件，PackageManager同样提供了类似的方法去获取成功匹配的组件信息，例如queryIntentServices、queryBroadcastReceivers等方法
5. 入口Activity：
```java
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```
# Activity请求回传值

一般通过startActivityForResult，然后在onActivityResult中接受。

```java
//RequestActivity部分代码：
Intent intent = new Intent(this, ResultActivity.class);
//启动一个Activity，并且携带本次请求的请求码Constant.REQUEST_CODE
startActivityForResult(intent, Constant.REQUEST_CODE);

//接收目标Activity的回传值
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	if (resultCode == RESULT_OK) {
		switch (requestCode) {
		case Constant.REQUEST_CODE:
			String number = data.getStringExtra("phone");
			et_number.setText(number);
			break;
		}
	}
}

//ResultActivity部分核心源码
Intent data = new Intent();
data.putExtra("phone", list.get(position));
setResult(Activity.RESULT_OK, data);
finish();

```

# Activity的一些技巧

## 锁定屏幕方向

只需在注册时设置android:screenOrientation属性为landscape或者portrait

## 全屏的Activity

全屏Activity就是状态栏都不显示，最常用的就是视频播放页面，可以在其 onCreate()方法中添加如下代码实现：

```java
//必须放在setContentView的前面
//去除标题栏
requestWindowFeature(Window.FEATURE_NO_TITLE);
setContentView(R.layout.activity_fullscreen);
//设置全屏
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
```

# task
task是一个具有栈结构的容器，可以放置多个Activity实例。启动一个应用，系统就会为之创建一个task，来放置根Activity；默认情况下，一个Activity启动另一个Activity时，两个Activity是放置在同一个task中的，后者被压入前者所在的task栈，当用户按下后退键，后者从task被弹出，前者又显示在幕前，特别是启动其他应用中的Activity时，两个Activity对用户来说就好像是属于同一个应用；系统task和task之间是互相独立的，当我们运行一个应用时，按下Home键回到主屏，启动另一个应用，这个过程中，之前的task被转移到后台，新的task被转移到前台，其根Activity也会显示到幕前，过了一会之后，在此按下Home键回到主屏，再选择之前的应用，之前的task会被转移到前台，系统仍然保留着task内的所有Activity实例，而那个新的task会被转移到后台，如果这时用户再做后退等动作，就是针对该task内部进行操作了。

## affinity
affinity对于Activity来说就好像它的身份证一样，可以告诉所在的task，自己属于这个task中的一员；拥有相同affinity的多个Activity理论同属于一个task，而task自身的affinity决定于根Activity的affinity值。  

### affinity在什么场合应用呢？  
1. 根据affinity重新为Activity选择宿主task（与allowTaskReparenting属性配合工作）；
2. 启动一个Activity过程中Intent使用了FLAG_ACTIVITY_NEW_TASK标记，根据affinity查找或创建一个新的具有对应affinity的task。

默认情况下，一个应用内的所有Activity都具有相同的affinity，都是从Application继承而来，而Application默认的affinity是<manifest>中的包名，我们可以为<application>设置taskAffinity属性值，这样可以应用到<application>下的所有<activity>，也可以单独为某个Activity设置taskAffinity。  
	android:taskAffinity="android.task.browser"

### affinity与Intent几种常见的flags的关系

##### FLAG_ACTIVITY_NEW_TASK
当Intent对象包含这个标记时，系统会寻找或创建一个新的task来放置目标Activity，寻找时依据目标Activity的taskAffinity属性进行匹配，如果找到一个task的taskAffinity与之相同，就将目标Activity压入此task中，如果查找无果，则创建一个新的task，并将该task的taskAffinity设置为目标Activity的taskActivity，将目标Activity放置于此task。注意，如果同一个应用中Activity的taskAffinity都使用默认值或都设置相同值时，应用内的Activity之间的跳转使用这个标记是没有意义的，因为当前应用task就是目标Activity最好的宿主（即不会创建一个新的task）。

<http://blog.csdn.net/liuhe688/article/details/6761337>