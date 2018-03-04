---
layout:     post
title:      "Android群英传读书笔记-第九章Android系统信息和安全机制"
date:       2016-07-17 23:10:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



本章主要讲如何查看Android系统的软硬件信息，包括：
> 
• Android系统信息的获取
• PackageManager的使用
• ActivityManager的使用
• Android安全机制

<!--more>

# 一. Android系统信息的获取
要获取系统的配置信息，通产可以从以下两个方面获取：
• android.os.Build
• SystemProperty
	
## 1.android.os.Build
android.os.Build类里面的信息非常的丰富，它包含了系统编译时的大量设备，配置信息，我们列举一些常用的：
> 
• Build.BOARD——主板
• Build.BRAND——Android系统定制商
• Build.SUPPORTED_ABIS——CPU指令集
• Build.DEVICE——设备参数
• Build.DISPLAY——显示屏参数
• Build.FINGERPRINT——唯一编号
• Build.SERIAL——硬件序列号
• Build.ID——修订版本列表
• Build.MANUFACTURER——硬件制造商
• Build.MODEL——版本
• Build.HARDWARE——硬件名
• Build.PRODUCT——手机产品名
• Build.TAGS——描述Build的标签
• Build.TYPE——Builder类型
• Build.VERSION.CODENAME——当前开发代号
• Build.VERSION.INCREMENTAL——源码控制版本号
• Build.VERSION.RELEASE——版本字符串
• Build.VERSION.SDK_INT——版本号
• Build.HOST——host值
• Build.USER——User名
• Build.TIME——编译时间

上面的一些参数没有注释，他们来自系统的RO值中，这些值都是手机生产商配置的只读的参数值，根据厂家的配置不同而不同，接下来我们再来看一下另一个存储设备软硬件信息的类——SystemProperty
## 2.SystemProperty
SystemProperty类包含了许多系统配置属性值和参数，很多信息和上面通过android.os.Build获取的值是相同的，我们列举一些常用的
> 
• os.version——OS版本
• os.name——OS名称
• os.arch——OS架构
• user.home——home属性
• user.name——name属性
• user.dir——Dir属性
• user.timezone——时区
• path.separator——路径分隔符
• line.separator——行分隔符
• file.separator——文件分隔符
• java.vendor.url——Java vender Url属性
• java.class.path——Java Class属性
• java.class.version——Java Class版本
• java.vendor——Java Vender属性
• java.version——Java版本
• java.home——Java Home属性

## 3.Android系统信息获取
```java
String brand = Build.BRAND;
String osversion = System.getProperty("os.version");
。。。都类似
```
这些信息的来源又是哪儿呢？在system/build.prop中，包含了很多的RO值，打开命令窗，通过cat build.prop可以看到。
除了上述的两个方法，android系统还在另外一个非常重要的目录来存储系统信息——/proc目录，在adb shell中进入/proc目录，通过ll命令查看文件信息，这里的信息比Build获得信息更加丰富。

# 二.Android APK应用程序信息获取之PackageManager
ADB命令中有两个非常强大的命令PM和AM，PM主宰着应用的包管理，AM主宰着应用的活动管理。
PackageManager:
![](http://oeiu2t0ur.bkt.clouddn.com/20160428234822274.png)

最里面的框就代表整个Activity的信息，系统提供了ActivityInfo类来进行封装；
最外面的框代表整个Mainifest文件中节点的信心，系统提供了PackageInfo来进行封装；
系统提供了PackageManager来负责管理所有已安装的App，PackageManager可以获得AndroidManifest中不同节点的封装信息，下面是一些常用的封装信息
> 
• AcitivityInfo 
ActivityInfo封装在了Mainfest文件中的< activity ></ activity >和<receiver></receiver>之间的所有信息，包括name、icon、label、launchMode等。
• serviceInfo
ServiceInfo与ActivityInfo类似，封装了< service></ service>之间的所有信息。
• ApplicationInfo
它封装了< application>之间的信息，特别的是，ApplicationInfo包含了很多Flag，FLAG_SYSTEM表示为系统应用，FLAG_EXTERNAL_STORAGE表示为安装在SDcard上的应用，通过这些flag可以很方便的判断应用的类型。
• PackageInfo
PackageInfo包含了所有的Activity和Service信息。
• ResolveInfo
ResolveInfo包含了< intent>信息的上级信息，所以它可以返回ActivityInfo、ServiceInfo等包含了< intent>的信息，经常用来帮助找到那些包含特定intent条件的信息，如带分享功能、播放功能的应用。 

有了这些封装的信息后，还需要有特定的方法来获取它们，下面就是PackageManager中封装的用来获取这些信息的方法：
> 
• getPackageManager()——通过这个方法可以返回一个PackageManager对象。
• getApplicationInfo()——以ApplicationInfo的形式返回指定包名的ApplicationInfo。
• getApplicationIcon()——返回指定包名的Icon。
• getInstalledApplications()——以ApplicationInfo的形式返回安装的应用。 
• getInstalledPackages()——以PackageInfo的形式返回安装的应用。
• queryIntentActivities()——返回指定Intent的ResolveInfo对象、Activity集合。
• queryIntentServices()——返回指定Intent的ResolveInfo对象、Service集合。
• resolveActivity()——返回指定Intent的Activity。
• resolveService()——返回指定Intent的Service。

看一个例子，通过PackageManager筛选不同类型的App。
判断App的类型，就是根据ApplicationInfo中的FLAG_SYSTEM来进行判断
> 
• 如果当前应用的flags & ApplicationInfo.FLAG_SYSTEM != 0则为系统应用
• 如果flags & ApplicationInfo.FLAG_SYSTEM <= 0 则为第三方应用
• 特殊的当系统应用升级后也会成为第三方应用，此时 flags & ApplicationInfo.FLAG_UPDATED_SYSTEM_APP != 0;
• 如果flags & ApplicationInfo.FLAG_EXTERNAL_STORAGE != 0 则为安装在SDCard上的应用。

例子就不写了，看书

# 三.Android APK应用程序信息获取值ActivityManager

PackageManager侧重于获取应用的包信息，ActivityManager侧重于获取运行的应用程序信息
同packagemanager一样，ActivityManager也封装了不少的Bean对象，我们选几个重点来说一下
> 
• ActivityManager.MemoryInfo——MemoryInfo有几个非常重要的字段：availMem（系统可用内存），totalMem（总内存），threshold（低内存的阈值，即区分是否低内存的临界值），lowMemory（是否处于低内存）。
• Debug.MemoryInfo——这个MemoryInfo用于统计进程下的内存信息。
• RunningAppProcessInfo——运行进程的信息，存储的字段有：processName（进程名），pid（进程pid），uid（进程uid），pkgList（该进程下的所有包）
• RunningServiceInfo——运行的服务信息，在它里面同样包含了一些服务进程信息，同时还有一些其他信息。activeSince（第一次被激活的时间、方式），foreground（服务是否在后台执行）。

下面这个方法可以获取正在运行的程序：
```java
	private List<ActivityManager.RunningAppProcessInfo> getRunningProcossInfo(){
	        mAMProcessInfo = new ArrayList<>();
	        List<ActivityManager.RunningAppProcessInfo>appRunningList = activityManager.getRunningAppProcesses();
	 
	        for (int i = 0;i<appRunningList.size();i++){
	            ActivityManager.RunningAppProcessInfo info = appRunningList.get(i);
	            int pid = info.pid;
	            int uid = info.uid;
	            String procossName = info.processName;
	            int[]memoryPid = new int[]{pid};
	            Debug.MemoryInfo[] memoryInfos = activityManager.getProcessMemoryInfo(memoryPid);
	 
	            int memorySize = memoryInfos[0].getTotalPss();
	 
	            AMProcessInfo processInfo = new AMProcessInfo();
	            processInfo.setPid(""+pid);
	            processInfo.setUid(""+uid);
	            processInfo.setMemorySize(""+memorySize);
	            processInfo.setProcessName(procossName);
	            appRunningList.add(processInfo);
	        }
	        return  appRunningList;
	    }
```	

# 四.解析Packages.xml获取系统信息
Android开机启动时，在系统初始化到时候，packagemanager的底层实现类PackageManagerService会去扫描系统的一些特定目录，并且解析其中的Apk文件，同时Android把他获取到的应用信息保存到xml中，做成一个应用的花名册，就是data/system/apckages.xml，我们可以用adb pull命令把他导出来。里面内容很多，我们只需知道各种标签的含义就可以了。
> 
• < permissions>标签
permissions标签定义了现在系统所有的权限，分两类，系统定义（package属性为Android）的和app定义的（package属性为apk的包名）。如
```java
<item name="android.permission.BIND_TV_INPUT" package="android" protection="18">
```
> 
• < package>标签
package代表的是一个apk的属性,其中各节点的信息含义大致为
	• name:APK的包名
	• codePath:APK安装路径，主要在system/app和data/app两种，前者放系统级别的，后者放系统安装的
	• userid:用户ID
	• version:版本
	• < perms>标签
对应apk的AndroidManifest文件中的<user_permission>标签，记录apk的权限信息

# 五.Android安全机制
在Android系统中建立了五道防线来保护Android的安全

1. 第一道防线 代码安全机制——代码混淆proguard 
	由于java语言的特殊性，即使是编译成apk的应用程序也存在反编译的风险，而proguard则是在代码从上对app的第一道程序，他混淆关键代码，替换命名，让破坏者阅读难，同样也可以压缩代码，优化编译后的字节。
2. 第二道防线（下面的理解应该只适用于6.0以前吧？）
	应用接入权限控制——清单文件权限声明，权限检查机制， 
	任何app在使用Android受限资源的时候，都需要显示向系统生命权限，只有当一个应用app具有相应的权限，才能申请相应的资源，通过权限机制的检查并且使用并且使用系统的Binder对象完成对系统服务的调用，但是这道防线也有先天的不足，如以下几项：
		• 被授予的权限无法停止
		• 在应用声明app使用权限的时候，用户无法针对部分权限进行限制
		• 权限的判断机制与用户的安全理念相关
	Android系统通常按照以下顺序来检查操作者的权限. 
		a. 首先,判断permission名称.如果为空则直接返回PERMISSION_DENIED 
		b. 其次。判断Uid,如果为0则为root权限，不做权限控制，如果为systyemsystem service的uid则为系统服务.不做权限控制:如果Uid与参数中的请求uid不同则返回PERMISSION_DENIED 
		c. 最后,通过调用packagemanageservice.checkUidPermission()方法来判断该uid是否具有相应的权限，该方法会去xml的权限列表和系统级的权限进行查找 
	通过上面的步骤Android就确定了使用者是否具有某项使用权限
3. 第三道防线
	应用签名机制一数字证书。 
	Android中所有的app都会有个数字证书,这就是app的签名.数字证书用于保护app的作者和其app的信任关系，只有拥有相同数字签名的app，才会在升级时被认为是同一app，而且Android系统不会安装没有签名的App
4. 第四道防线
	Linux内核层安全机制一一Uid 访问权限控制。 
	Animid本质是基于Linux内核开发的，所以Android同样继承了Linux的安全特性，比如Linux文件系统的权限控制是由user,group,other与读，写，执行的不同组合来实现的，同样,Android也实现了这套机制”通常情况下.只有system，root用户才有权限访问到系统文件，而一般用户无法访问。
5. 第五道防线
	Android虚拟机沙箱机制——沙箱隔流 
	Android的App运行在虚拟机中 因此才有沙箱机制，可以让应用之间相互隔离，通常情况下.不同的应用之间不能互相访问.每个App都单独的运行在虚似机中，与其他应用完全隔离.在实现安全机制的基础上，也让应用之间能够互不影响,即时一个应用崩溃，，也不会导致其他应用异常。
	
## Android Apk反编译
APk文件,说到底也是一个压缩文件，那么可以通过解压缩, 获得里面的文件内容（直接改后缀名，解压），解压缩出来的文件：
![](http://oeiu2t0ur.bkt.clouddn.com/3232.png)

当然里面的文件基本都无法打开，或者打开也是乱码，正确姿势：
> 
apktools:反编译 
下载地址：http://ibotpeaches.github.io/Apktool/install/ 
dex2jar 这个工具用于将dex文件转换成jar文件 
下载地址：http://sourceforge.net/projects/dex2jar/files/ 
jd-gui 这个工具用于将jar文件转换成java代码 
下载地址：http://jd.benow.ca/

这三个工具分别负责反编译不同的部分

- apktool
首先我们来反编译apk的xml文件，使用的是apktool，我们进入所在的目录执行反编译命令
![](http://oeiu2t0ur.bkt.clouddn.com/20160428234701633.png)
格式非常的简单，参数d是指decode，并写入要反编译的目录，执行后，就会生成一个对应apk名字的文件夹，这个时候你进去看xml的代码就不会有错误了
![](http://oeiu2t0ur.bkt.clouddn.com/20160428214218256.png)

这个工具可以方便我们汉化们重新打包的命令是b，选择文件夹即可
下面我们来解决源码

- Dex2jar,jd-gui
现在需要这两个工具了，我们回到apk的文件夹，里面有一个非常重要的文件classes.dex,这个就是源代码了，我们把它复制到Dex2jar的目录下
![](http://oeiu2t0ur.bkt.clouddn.com/20160428235209681.png)

并且执行如下命令:
![](http://oeiu2t0ur.bkt.clouddn.com/20160428235342229.png)

这里就生成了一个jar文件，然后我们就可以用最后的jd-gui去查看了
![](http://oeiu2t0ur.bkt.clouddn.com/20160428235519479.png)

##　Android APK 加密
由于java字节的特殊性，他很容易反编译，为了能够保护好代码，我们通常会使用一些措施，比如说混淆，而在Android studio中，可以很方便的使用ProGuard，在Gradle Scripts目录下
```java
 buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```
这里的minifyEnabled属性就是控制是否启动ProGuard，这个属性以前叫做runProgyard,在Android studio1.1的时候改成minifyEnabled，将他设置成true，就可以混淆了，他位于《SDK目录下的tools/proguard/proguard-android.txt目录下，大部分的情况下使用使用这个默认的混淆就好了，后面亦不过分是项目中自定义的混淆，可以在项目的app文件夹下找到这个文件，在这根文件里可以定义引用的第三方依赖库和混淆规则，配置好ProGuard之后，用AS到处apk即可，
