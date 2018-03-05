---
layout:     post
title:      "Style和Theme"
date:       2016-03-10 19:06:47
author:     "afayp"
catalog:    true
tags:
    - Android
---


# Style

## 概念


style是给View或Window指定外观和格式的属性集合。style能够指定如高、边距、字体颜色、字体尺寸、背景颜色等属性。它被定义在一个与布局xml文件分开的xml,资源文件中。通过使用style可以更加简洁的表示组件的一些样式。

<!--more-->

不使用样式属性：
```java
<TextView
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:layout_marginBottom="10dp"
	android:text="Text | Default" />
```

使用样式：
```java
<TextView
	style="@style/CodeFont"
	android:text="Text | CodeFont" />
```

## 创建style

如果要创建一个样式，就要把一个xml文件保存在项目目录res/values目录中，xml文件的名字可以随意（不过一般都在style中，但是如果定义的样式太多，如果都放在styles.xml文件里，那这文件也太臃肿了这时可以将样式分类拆分成多个文件，如style_holo），但是这个xml文件的根节点必须是<resources>。    
具体的每种样式定义则是通过resource下的子标签style来完成，name属性唯一的标识这个样式。每个style通过添加多个item来设置样式不同的属性，item也必须有name属性，<item>元素的值能够是一个字符串、十六进制的颜色、另一个资源类型的引用、或者依赖样式属性的其他值。示例：

```java
<resources>
    <style name="CodeFont" parent="@android:style/Widget.TextView">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_marginBottom">10dp</item>
    </style>
</resources>
```

resources元素的每个子元素在编译时都要被转换成一个应用程序资源对象，通过style元素的name属性值来引用。这个示例的样式，在xml布局中使用@style/CodeFont来引用。

在xml中定义的一个样式style，它可以使用在一个View也可以作用于某个Activity或者整个应用Application，所以他们的样式style定义完全相同。


另外，样式是可以继承的，可通过style标签的parent属性声明要继承的样式，也可通过点前缀 (.) 继承，点前面为父样式名称，后面为子样式名称。点前缀方式只适用于自定义的样式，若要继承Android内置的样式，则只能通过parent属性声明。如果需要，可以覆写item来替换原来的item属性值。


## 使用样式：
- 引用系统已有style，可以直接使用@android:style/系统样式名，如：parent="@android:style/Widget.TextView"

- 引用非系统自定义style，使用@style/自定义样式名，如：parent="@style/CodeFont"

- 引用主题属性attrs中的属性值，在布局文件引用：style="?android:attr/progressBarStyleHorizontal"

- 引用自定义主题属性attrs中属性值，有两种形式 如：android:background="?attr/settingbackground"或者android:background="?settingbackground"（其中settingbackground为自定义的一个主题属性：<attr name="settingbackground" format="reference|color"/>；）

在style中每一个条目item的属性定义中，除了Android本身内置的属性添加命名空间android:xxx之外，其余的第三方包或者support包以及自定attrs中主题属性统统不需要添加命名空间来重置style。

## 样式引用范围

当把一个样式应用于布局中单一的View对象时，这个样式定义的属性只会用于这个View对象。如果样式被用于一个ViewGroup对象，那么其中的View子对象不会继承这个样式属性，因为样式只会用于直接引用该样式的元素。


# 主题

主题是应用在整个Activity或应用程序Application的样式，而不是一个独立的View对象。当一个样式被用作主题时，Activity或应用程序中的每个View对象都会使用它所支持的每个样式属性。例如，把相同的CodeFont样式用作一个Activity的主题，那么这个Activity内的所有文本都会使白色字体。


通过android:theme属性为Activity或者Application设置主题（引用的其实也是style*，但一般称为主题。主题一般以XxxTheme命名）
```java
<application android:theme="@style/CustomTheme"> 
<activity android:theme="@android:style/Theme.Dialog"> 
```


Android系统提供了多套主题，查看Android的frameworks/base/core/res/res/values目录，就会看到有以下几个文件(目前为止)：

- themes.xml：低版本的主题，目标API level一般为10或以下
- themes_holo.xml：从API level 11添加的主题
- themes_device_defaults.xml：从API level 14添加的主题
- themes_material.xml：从API level 21添加的主题
- themes_micro.xml：应该是用于Android Wear的主题
- themes_leanback.xml： 还不清楚什么用


不过在实际应用中，因为大部分都采用兼容包的，一般都会采用兼容包提供的一套主题：Theme.AppCompat。AppCompat主题默认会根据不同版本的系统自动匹配相应的主题，比如在Android 5.0系统，它会继承Material主题。不过这也会导致一个问题，不同版本的系统使用不同主题，就会出现不同的体验。因此，为了统一用户体验，最好还是自定义主题。


如果要使用一个主题，但需要调整，那么可以把这个主题作为定制主题的父主题。例如，可以修改传统的亮度主题，并添加自己想要的颜色：
```java
<color name="custom_theme_color">#b0b0ff</color>
<style name="CustomTheme" parent="android:Theme.Light">
	<item name="android:windowBackground">@color/custom_theme_color</item>
	<item name="android:android:colorBackground">@color/custom_theme_color</item>
</style> 
```

# 版本兼容


有些属性可能只在高版本上才能使用，低版本上使用会报错。  
比如android:windowTranslucentStatus属性只在Android4.4以上的版本才可以使用，这时候我们要在res/values-19目录下新建一个xml文件style来放入一个可选主题声明。

values下的style.xml文件中：
```java
<style name="AppTheme" parent="AppBaseTheme">
	<写一些低版本高版本兼容的属性item>
</style>
```
values-19下的style.xml文件中：

```java
<style name="AppTheme" parent=" AppBaseTheme ">
	<一些通用兼容属性，下面是高版本才能用的属性>
	<item name="android:windowTranslucentStatus">true</item>
</style>
```

当Android版本在4.4及其以上时就会自动使用android:windowTranslucentStatus属性了。


# xml资源文件中@、？、@+三者的含义和区别

##　@代表引用资源

### 引用自定义资源，格式：@type/name

android：textColor="@color/red"

### 引用系统资源，格式：@android:type/name
android:textColor="@android:color/red"

## ？代表引用主题属性
系统属性：style="?android:attr/progressBarStyleHorizontal"
自定义属性：android:background="?attr/settingbackground"
		或者android:background="?settingbackground"

##　@+代表在创建引用资源，格式：@+type/name
最常见的设置id：
android:id="@+id/newId"




# 参考链接
<http://www.sunnyang.com/347.html>