---
layout:     post
title:      "安卓注解-Support Annotation Library"
date:       2016-05-15 20:14:13
author:     "afayp"
catalog:    true
tags:
    - Android
---



# 安卓注解-Support Annotation Library


Android support library从19.1版本开始引入了一个新的注解库，它包含很多有用的元注解，用来帮助开发者在编译期间发现可能存在的Bug。

<!--more>

### 如何使用
Annotation Library默认不会包含在项目中，要手动引入，只需在gradle中加入一下依赖即可。
```java
dependencies {
    compile 'com.android.support:support-annotations:23.1.1'
}
```

注意：对于`Android application`和`Android library`这两个类型的module(你应用了com.android.application或者com.android.library插件的)来说，上面这样就可以使用注解了。
但如果你想只在`Java module`使用这些注解，那你就得明确的包含SDK仓库了，因为support libraries不能从jcenter获得(Android Gradle插件会自动的包含这些依赖，但是Java插件却没有。)
```java
repositories {
   jcenter()
   maven { url '<your-SDK-path>/extras/android/m2repository' }
}
```
当你用Android Studio和IntelliJ的时候，如果给标注了这些注解的方法传递错误类型的参数，那么IDE就会实时标记出来。

一共有8种类型，分别是Nullness注解、资源类型注解、线程注解、变量限制注解、权限注解、结果检查注解、CallSuper注解、枚举注解(IntDef和StringDef)。下面分类介绍常用的几种注解：

### Nullness注解
> 
@Nullable注解能被用来标注给定的参数或者返回值可以为null。  
@NonNull注解能被用来标注给定的参数或者返回值不能为null。

### 资源类型注解
Android的资源值通常都是使用整型传递。这意味着一个需要传入Layout资源的函数，如果传入String资源并不会在编译时报错；因为他们都是int类型，编译器很难区分。只有在运行时才会发现问题。  
资源类型注解可以在这种情况下提供类型检查。  

此类注解以Res结尾，常用的有：
> 
AnimatorRes ：animator资源类型  
AnimRes：anim资源类型  
AnyRes：任意资源类型  
ArrayRes：array资源类型  
AttrRes：attr资源类型  
BoolRes：boolean资源类型  
ColorRes：color资源类型  
DimenRes：dimen资源类型。  
DrawableRes：drawable资源类型  
FractionRes：fraction资源类型  
IdRes：id资源类型  
IntegerRes：integer资源类型  
InterpolatorRes：interpolator资源类型  
LayoutRes：layout资源类型  
MenuRes：menu资源类型  
PluralsRes：plurals资源类型  
RawRes：raw资源类型  
StringRes：string资源类型  
StyleableRes：styleable资源类型  
StyleRes：style资源类型  
TransitionRes：transition资源类型  
XmlRes：xml资源类型  

### 线程注解
可以给某个方法指定运行的线程，如果没有在制定的线程中执行也是编译不过的。
> @UiThread UI线程  
@MainThread 主线程  
@WorkerThread 子线程  
@BinderThread  绑定线程 

通常一个应用只有一个主线程，也就是UI线程，所以一般@UiThread和@MainThread可以互换。不过我们一般用@MainThread来注解生命周期相关函数，用@UiThread来注解视图相关函数。

### 类型定义注解
整型除了可以作为资源的引用之外，也可以用作“枚举”类型使用。不用枚举的原因是因为性能考虑,可以参考[Android性能优化典范 - 第3季](http://hukai.me/android-performance-patterns-season-3/)

@IntDef和@typedef作用非常类似，你可以创建另外一个注解，然后用@IntDef指定一个你期望的整型常量值列表，最后你就可以用这个定义好的注解修饰你的API了。

```java 
@IntDef({NAVIGATION_MODE_STANDARD, NAVIGATION_MODE_LIST, NAVIGATION_MODE_TABS})
@Retention(RetentionPolicy.SOURCE)
public @interface NavigationMode {}

public static final int NAVIGATION_MODE_STANDARD = 0;
public static final int NAVIGATION_MODE_LIST = 1;
public static final int NAVIGATION_MODE_TABS = 2;

@NavigationMode
public abstract int getNavigationMode();

public abstract void setNavigationMode(@NavigationMode int mode);
```
首先创建了一个新的注解(NavigationMode)并且用@IntDef标注它,通过@IntDef为返回值或者参数指定了可用的常量值。我们还添加了@Retention(RetentionPolicy.SOURCE)告诉编译器这个新定义的注解不需要被记录在生成的.class文件中

使用这个注解后，如果你传递的参数或者返回值不在指定的常量值中的话，IDE将会给出警告。


### 值范围注解
实际开发过程中，我们有时可能需要设置一个取值范围，这时我们可以使用取值范围注解来约束。
有三种：
>  @Size, @IntRange, @FloatRange

如果你的参数是一个int或者long类型，你可以使用@IntRange注解约束其值在一个特定的范围内：
```public void setAlpha(@IntRange(from=0,to=255) int alpha) { … }```
如果你的参数是一个float或者double类型，并且一定要在某个范围内，你可以使用@FloatRange注解：
```public void setAlpha(@FloatRange(from=0.0, to=1.0) float alpha) {...}```

对于数据、集合以及字符串，你可以用@Size注解参数来限定集合的大小(当参数是字符串的时候，可以限定字符串的长度)。

举几个例子

集合不能为空: @Size(min=1)
字符串最大只能有23个字符: @Size(max=23)
数组只能有2个元素: @Size(2)
数组的大小必须是2的倍数: @Size(multiple=2)

### 权限注解
有时我们的方法调用需要调用者拥有指定的权限，这时我们可以使用@RequiresPermission注解
```
@RequiresPermission(Manifest.permission.SET_WALLPAPER)
public abstract void setWallpaper(Bitmap bitmap) throws IOException;
```

如果你至少需要权限集合中的一个，你可以使用anyOf属性
```
@RequiresPermission(anyOf = {
    Manifest.permission.ACCESS_COARSE_LOCATION,
    Manifest.permission.ACCESS_FINE_LOCATION})
public abstract Location getLastKnownLocation(String provider);
```

如果你同时需要多个权限，你可以用allOf属性
```
@RequiresPermission(allOf = {
    Manifest.permission.READ_HISTORY_BOOKMARKS, 
    Manifest.permission.WRITE_HISTORY_BOOKMARKS})
public static final void updateVisitedHistory(ContentResolver cr, String url, boolean real) ;
```

对于intents的权限，可以直接在定义的intent常量字符串字段上标注权限需求(他们通常都已经被@SdkConstant注解标注过了)
```
@RequiresPermission(android.Manifest.permission.BLUETOOTH)
public static final String ACTION_REQUEST_DISCOVERABLE =
            "android.bluetooth.adapter.action.REQUEST_DISCOVERABLE";
```

对于content providers的权限，你可能需要单独的标注读和写的权限访问，所以可以用@Read或者@Write标注每一个权限需求
```
@RequiresPermission.Read(@RequiresPermission(READ_HISTORY_BOOKMARKS))
@RequiresPermission.Write(@RequiresPermission(WRITE_HISTORY_BOOKMARKS))
public static final Uri BOOKMARKS_URI = Uri.parse("content://browser/bookmarks");
```

### 方法重写 @CallSuper
如果你的API允许使用者重写你的方法，但是呢，你又需要你自己的方法(父方法)在重写的时候也被调用，这时候你可以使用@CallSuper标注

例如：Activity的onCreate函数
```
@CallSuper
protected void onCreate(@Nullable Bundle savedInstanceState) 
```

### @VisibleForTesting

你可以把这个注解标注到类、方法或者字段上，以便你在测试的时候可以使用他们。

### @Keep

我们还在注解库里添加了@Keep注解，但是Gradle插件还支持（尽管已经在进行中）。被这个注解标注的类和方法在混淆的时候将不会被混淆。



上面介绍了大部分常用的注解，更多注解可查看[官方文档](https://developer.android.com/studio/write/annotations.html)
