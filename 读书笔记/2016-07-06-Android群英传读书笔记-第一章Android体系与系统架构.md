---
layout:     post
title:      "Android群英传读书笔记-第一章Android体系与系统架构"
date:       2016-7-6 20:59:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---

# 1.1 Google的生态圈
Android是一个以Linux为基础的开源移动设备操作系统，主要用于智能手机和平板电脑，由Google成立的Open Handset Alliance（OHA，开放手持设备联盟）持续领导与开发中。
Android系统最初由安迪·鲁宾（Andy Rubin）等人开发制作 ，最初开发这个系统的目的是创建一个数码相机的先进操作系统；但是后来发现市场需求不够大，加上智能手机市场快速成长，于是Android被改造为一款面向智能手机的操作系统。於2005年8月被美国科技企业Google收购。2007年11月，Google与84家制造商、开发商及电信营运商成立开放手持设备联盟来共同研发改良Android系统，随後，Google以Apache免费开放原始码许可证的授权方式，发布了Android的原码，让生产商推出搭载Android的智能手机，Android後来更逐渐拓展到平板电脑及其他领域上。我们来看看Android的成长史：
<!--more-->

![](http://oeiu2t0ur.bkt.clouddn.com/20160223181655862.jpg)

# 1.2 Android系统架构
![](http://oeiu2t0ur.bkt.clouddn.com/20160223181826154.png)

大致可分为四层：

- Linux底层
- 库和运行时
- Framework层
- 应用层

## 1.2.1 Linux
> 
	主要是一些驱动什么的，看图片（红色部分）就可以看出，什么相机驱动，蓝牙驱动什么的，这些事Android最底层最核心的部分，我们打开关于手机就可以看到内核版本，这就是我们使用的Linux内核版本
	
## 1.2.2 Dalvik和ART
> 
	这两种都是运行环境的虚拟机，Dalvik是以前的，ART是Android 4.4（好像是）的时候发布的，因为Dalvik是应用运行的时候进行编译，而ART是全部编译完了再运行，效率要高很多

## 1.2.3 Framework
> 各种manager。。。

## 1.2.4 Standard libraries
> 包含了一些android中的标准库，比如SQLite，OpenGL ES

## 1.2.5 Application
	。。。
# 1.3 Context
	见Context知识点
	
# 1.4.1 Android系统源代码

## 1.4.1 Android系统源代码目录
查看源代码的网站：[http://androidxref.com/](http://androidxref.com/) 
这里要注意，这个目录结构也只有AOSP的Android项目才是，其他厂商，比如MTK就不是。
Android作为手机操作系统，我们需要把他编译后才能使用，这里我们就不能使用AS，Eclipse之类的开发IDE了，Android和其他语言一样引入Mackfile机制。一个像Android这样大的工程，源码肯定是有很多的，而且种类更是繁多，这些文件都是由一个叫做Mackfile的文件来管理的，他有自己的规则来归类这些信息，比如编译规则，打包规则，所以Mackfile就像一个shell脚本，不仅可以使用自己的语法，而且可以调用操作系统的命令。Android源码中，每个最小功能单位的目录下都会有一个Makefile文件，这样每级向上，通过一个个Makefile文件就把整个源代码有条不紊的联系在一起了。
## 1.4.2 Android系统目录
主要看/system和/data这两个目录。
• system/app：里放着的使我们系统的应用
• system/bin：放着Linux自带的一些组件
• system/build.prop：清单文件，记录着各种各样的信息
• system/fonts： 字体
• system/framework：系统的核心文件，框架层
• system/lib：存放一些共享库
• system/media：铃声之类的声音文件
• system/usr ：保存用户的数据
• data/app：用户安装的app
• data/data：应用数据
• data/system： 各项手机信息
• data/misc 保存着wifi vpn等信息 ，已经连接的wifi密码也是在这里看到的
## 1.4.3 Android APP文件目录
新建一个工程长什么样。。不赘述
