---
layout:     post
title:      "Android-apk签名"
date:       2016-04-03 19:51:30
author:     "afayp"
catalog:    true
tags:
    - Android
---



# Android签名机制

Android系统在安装APK的时候，首先会检验APK的签名，如果发现签名文件不存在或者校验签名失败，则会拒绝安装，所以应用程序在发布之前一定要进行签名。

<!--more-->

# 好处

## 保护应用
应用签名之后虽然无法阻止APK被人修改，但修改后再签名就和原先的签名不一致，这样可以避免有些人用相同包名的APK来替换已有的应用

## 应用程序升级

如果想无缝升级一个应用，Android系统要求应用程序的新版本与老版本具有相同的签名与包名。若包名相同而签名不同，系统会拒绝安装新版应用。

## 应用程序模块化
Android系统可以允许同一个证书签名的多个应用程序在一个进程里运行，系统实际把他们作为一个单个的应用程序。此时就可以把我们的应用程序以模块的方式进行部署，而用户可以独立的升级其中的一个模块。


## 代码或数据共享
Android提供了基于签名的权限机制，一个应用程序可以为另一个以相同证书签名的应用程序公开自己的功能与数据，同时其它具有不同签名的应用程序不可访问相应的功能与数据。

在AndroidManifest.xml文件中，提供了一个基于签名的Permission标签。通过设置，我们可以实现对不同App之间的访问和共享，如下：  
```java
<permission android:protectionLevel="normal" />
```

其中protectionLevel标签有4种值：normal(缺省值),dangerous, signature,signatureOrSystem。
> normal是低风险的，所有的App不能访问和共享此App。  
> dangerous是高风险的，所有的App都能访问和共享此App。  
> signature是指具有相同签名的App可以访问和共享此App。


# 签名原理

对一个APK文件签名之后，APK文件根目录下会增加META-INF目录，该目录下增加三个文件：

> MANIFEST.MF  
> NETEASE.RSA  
> NETEASE.SF  

Android系统就是根据这三个文件的内容对APK文件进行签名检验的。  
这三个文件的具体内容参考<http://www.36nu.com/post/183.html>


# 生成签名

Android签名有两种方式：DEBUG和RELEASE。  
在开发测试期间使用DEBUG方式，BUILD时，会自动使用工具KeyTools创建KEY包括别名和密码。每次编译时，都会使用DEBUG的KEY进行签名。
在第一次安装Android开发环境的时候，SDK工具已经创建了缺省的keystore/key和账号、密码：
```java
Keystore name – "debug.keystore"
Keystore password – "android"
Key alias – "androiddebugkey"
Key password – "android"
CN – "CN=Android Debug,O=Android,C=US"
```

keystore其实就是一个文件，存放以上信息的文件，由于使用了加密难以看懂。DEBUG模式的签名只有365天有效期,过了有效期，编译会出错。但不用担心，只要将debug.keystore文件删除后，下次BUILD会自动生成的keystore和key的。debug.keystore文件一般在/home/username/.android目录下。

##　使用Keytool生成key文件

创建key，需要用到keytool(位于JAVA_HOME\jre\bin目录下),在Shell中输入：
```java
keytool -genkey -alias android.keystore -keyalg RSA -validity 36500 -keystore android.keystore
```
命令行参数解释：

- genkey:产生秘钥
- alias android.keystore 别名 android.keystore
- keyalg RSA 使用RSA算法对签名加密
- validity 36500 有效期限
- keystore android.keystore  存储的文件名

## 使用jarsigner签名

jarsigner在目录JAVA_HOME\bin下，在Shell中输入：
```java
jarsigner -verbose -keystore android.keystore -signedjar android_signed.apk android.apk android.keystore
```

命令行参数解释：

- verbose 输出签名的详细信息
- keystore  android.keystore 密钥库位置
- signedjar android_signed.apk android.apk android.keystore 正式签名，三个参数中依次为签名后产生的文件android_signed，要签名的文件android.apk和密钥库android.keystore


## 使用Gradle完成签名

在Module的build.gradle文件的android配置代码块添加如下内容：
```java
android{
    signingConfigs {
        debug {
            storeFile file("/home/lippi/.android/debug.keystore")
        }
        relealse {
            //这样写就得把demo.jk文件放在项目目录
            storeFile file("android.keystore")
            storePassword "android"
            keyAlias "lippi"
            keyPassword "password"
        }
    }
   buildTypes {
        debug {
            // 显示Log
            buildConfigField "boolean", "LOG_DEBUG", "true"

            versionNameSuffix "-debug"
            minifyEnabled false
            zipAlignEnabled false
            shrinkResources false
            signingConfig signingConfigs.debug
        }

        release {
            // 不显示Log
            buildConfigField "boolean", "LOG_DEBUG", "false"
            //混淆
            minifyEnabled true
            //Zipalign优化
            zipAlignEnabled true

            // 移除无用的resource文件
            shrinkResources true
            //前一部分代表系统默认的android程序的混淆文件，该文件已经包含了基本的混淆声明
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard.cfg'
            //签名
            signingConfig signingConfigs.relealse
        }
    }
}
```
然后执行gradle 命令：
```java
gradle assembleRelease
```

编译并发布。 在build/outputs/apk/ 下能看到未签名的apk 和 已经签名的apk。如果未用签名文件，使用debug mode的debug签名，那就会生成一个debug签名的apk。

签名密码放在Gradle文件中不安全,可以改成下面这样的格式这样在执行命令时，就会被要求输入密码

```java
signingConfigs { 
    myConfig { 
        storeFile file("android.keystore")  
        storePassword System.console().readLine("\ninput Keystore password: ")  
        keyAlias "lippi"  
        keyPassword System.console().readLine("\n input Key password: ")  
    } 
}
```

##　使用Android Studio自带的签名工具

菜单Build > Generate Signed APK,具体看<http://chenfeicqq.iteye.com/blog/1889160>

# 参考链接
<http://www.ezlippi.com//blog/2016/03/android-apk-signing.html>  
<http://www.36nu.com/post/183.html>