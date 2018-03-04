---
layout:     post
title:      "ButterKnife框架原理浅析"
date:       2016-10-21 23:46:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 源码解析
---


[`ButterKnife`](http://jakewharton.github.io/butterknife/)是`Jake Wharton`写的开源依赖注入框架。与传统的注解库使用反不同，
它用了`Java Annotation Processing`技术，在编译期间生成辅助代码来达到View注入的目的,就是在Java代码编译成Java字节码的时候就已经处理了`@Bind、@OnClick`这些注解。

<!--more-->

> Annotation Processing Tool(APT)(注解处理器)是Java1.5引入的工具，它提供了在程序编译期间扫描和处理注解的能力。它的原理就是在编译期间读取Java代码，解析注解，然后动态生成Java代码。下图是Java编译代码的流程，可以看到，我们的注解处理器的工作在Annotation Processing阶段，最终通过注解处理器生成的代码会和源代码一起被编译成Java字节码。不过比较遗憾的是你不能修改已经存在的Java文件，比如在已经存在的类中添加新的方法，所以通过Java Annotation Tool只能通过辅助类的方式来实现View的依赖注入，这样会略微增加项目的方法数和类数，不过只要控制好，不会对项目有太大的影响
![](http://oeiu2t0ur.bkt.clouddn.com/213213.png)

# ButterKnife工作流程
当你编译你的Android工程时，ButterKnife工程中`ButterKnifeProcessor`类的`process()`方法会执行以下操作：

- 开始它会扫描Java代码中所有的`ButterKnife`注解`@Bind、@OnClick、@OnItemClicked`等
- 当它发现一个类中含有任何一个注解时，`ButterKnifeProcessor`会帮你生成一个Java类，名字类似`<className>$$ViewBinder`，这个新生成的类实现了`ViewBinder<T>`接口
- 这个`ViewBinder`类中包含了所有对应的代码，比如`@Bind`注解对应`findViewById()`, `@OnClick`对应了`view.setOnClickListener()`等等
- 最后当Activity启动`ButterKnife.bind(this)`执行时，`ButterKnife`会去加载对应的`ViewBinder`类调用它们的`bind()`方法

## 栗子
```java
class ExampleActivity extends Activity {
     @Bind(R.id.user) EditText username;
     @Bind(R.id.pass) EditText password;

    @Override public void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.simple_activity);
         ButterKnife.bind(this);
         // TODO Use fields…
     }

     @OnClick(R.id.submit) void submit() {
     // TODO call server…
     }
}
```

编译成功后，生成了下面的代码：
```java
public class ExampleActivity$$ViewBinder<T extends 
        io.bxbxbai.samples.ui.ExampleActivity> implements ViewBinder<T> {

     @Override public void bind(final Finder finder, final T target, Object source) {
          View view;
          view = finder.findRequiredView(source, 21313618, “field ‘user’”);
          target.username = finder.castView(view, 21313618, “field ‘user’”);
          view = finder.findRequiredView(source, 21313618, “field ‘pass’”);
          target.password = finder.castView(view, 21313618, “field ‘pass’”);
          view = finder.findRequiredView(source, 21313618, “field ‘submit’ and method ‘submit’”);
          view.setOnClickListener(
            new butterknife.internal.DebouncingOnClickListener() {
               @Override public void doClick(android.view.View p0) {
      target.submit();
           }
        });
      }

     @Override public void reset(T target) {
           target.username = null;
           target.password = null;
     }
}
```
示意图：
![](http://oeiu2t0ur.bkt.clouddn.com/butterknife_example.png)

## ButterKnife.bind 执行阶段
最后，执行`bind`方法时，我们会调用`ButterKnife.bind(this)`：

- `ButterKnife`会调用`findViewBinderForClass(targetClass)`加载`ExampleActivity$$ViewBinder.java`类
- 然后调用`ViewBinder`的`bind`方法，动态注入`ExampleActivity`类中所有的`View`属性
- 如果Activity中有`@OnClick`注解的方法，`ButterKnife`会在`ViewBinder`类中给View设置`onClickListener`，并且将`@OnClick`注解的方法传入其中

在上面的过程中可以看到，为什么你用`@Bind、@OnClick`等注解标注的属性或方法必须是`public`或`protected`的，因为`ButterKnife`是通过`ExampleActivity.this.editText`来注入View的

为什么要这样呢？有些注入框架比如`roboguice`你是可以把View设置成private的，答案就是性能。如果你把View设置成private，那么框架必须通过反射来注入View，不管现在手机的CPU处理器变得多快，如果有些操作会影响性能，那么是肯定要避免的，这就是ButterKnife与其他注入框架的不同

## 注意
通过ButterKnife来注入View时，ButterKnife有`bind(Object, View) `和 `bind(View)`两个方法，有什么区别呢？

如果你自定义了一个View，比如`public class BadgeLayout extends Fragment`，那么你可以通过`ButterKnife.bind(BadgeLayout)`来注入View的

如果你在一个ViewHolder中`inflate`了一个xml布局文件，得到一个View对象，并且这个View是`LinearLayout或FrameLayout`等系统自带View，那么不是不能用`ButterKnife.bind(View)`来注入View的，因为ButterKnife认为这些类的包名以`com.android`开头的类是没有注解功能的,所以这种情况你需要使用`ButterKnife.bind(ViewHolder，View)`来注入View。
这表示你是把`@Bind、@OnClick`等注解写到了这个ViewHolder类中，ViewHolder中的View呢需要从后面那个View中去找.



# 参考
[ButterKnife框架原理](http://bxbxbai.github.io/2016/03/12/how-butterknife-works/?utm_source=tuicool&utm_medium=referral)  
[浅析ButterKnife](http://mp.weixin.qq.com/s?__biz=MzI1NjEwMTM4OA==&mid=2651232205&idx=1&sn=6c24e6eef2b18f253284b9dd92ec7efb&chksm=f1d9eaaec6ae63b82fd84f72c66d3759c693f164ff578da5dde45d367f168aea0038bc3cc8e8&scene=0#wechat_redirect)
