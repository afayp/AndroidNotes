---
layout:     post
title:      "Android中的MVP设计模式"
date:       2016-06-22 15:00:22
author:     "afayp"
catalog:    true
tags:
    - 设计模式
---



# 架构的目的
通常一个APP是由多种数据模型（Model）和多种视图（View）组成，如果我们直接使用Model-View设计模型，那这将使得我们的程序代码变得复杂、耦合度高、不利于单元测试和代码重构。 
 
<!--more-->

当然设计不能违背目的，对于不同量级的工程，具体架构的实现方式必然是不同的，切忌犯为了设计而设计，为了架构而架构的毛病。

所有的架构都是为了解耦，提高可维护性，如果违背了这个初衷，为了使用而使用，最终肯定是得不偿失的。  

而解耦的常用方法有两种：分层与模块化。模块化很好理解，就是按照功能划分模块；分层按照我的理解，比如项目中一般都有一些基础组件，业务组件，这就是一种分层的思想吧。

# 什么是MVP设计模式

简单来说就是View负责显示，Presenter负责逻辑处理，Model提供数据。具体：  

- View：用户交互层，代表UI组件部分，与用户进行交互。在Android中它可能是Activity、Fragment、View、Dialog。  
	- 一般每个view都有对应的View interface，里面定义了view与用户交互要实现的方法。p层通过这个View interface与view层交互，可以降低耦合，方便进行单元测试。
- Model:数据访问层，在Model层中实现数据的获取或者更新等操作，例如对数据库查询，网络获取数据等等（有时也实现一个Model interface用来降低耦合)  
- Presenter是View和Model的中间纽带，负责UI层的交互和后台数据处理任务。p层持有view层和model层的引用，用来调用对应的方法更新UI或者更新Model。
	- 因为P层持有view（activity）的引用，所以要在activity销毁时移除p层对activity的引用（置为null）。建议定义一个顶级的p层，是一个接口，有相关的方法，具体的p层都要实现这些方法。如下
	```java
	//顶层p,这里用了泛型，因为每个p关联的v都不一样
	public interface IPresenter<V> {
	    void attachView(V view);//当p拿到v引用时调用
	    void detachView();//activity销毁时调用，移除activity的引用
	}
	```


## 关于View interface的必要性:  
回想一下你在开发Android应用时是如何对代码逻辑进行单元测试的？是否每次都要将应用部署到Android模拟器或真机上，然后通过模拟用户操作进行测试？然而由于Android平台的特性，每次部署都耗费了大量的时间，这直接导致开发效率的降低。而在MVP模式中，处理复杂逻辑的Presenter是通过interface与View(Activity)进行交互的，这说明我们可以通过自定义类实现这个interface来模拟Activity的行为对Presenter进行单元测试，省去了大量的部署及测试的时间。

具体到Android App中，一般可以将App根据程序的结构进行纵向划分，根据MVP可以将App分别为模型层(M)，UI层(V)和逻辑层(P)。

UI层一般包括Activity，Fragment，Adapter等直接和UI相关的类，UI层的Activity在启动之后实例化相应的Presenter，App的控制权后移，由UI转移到Presenter，两者之间的通信通过BroadCast、Handler或者接口完成，只传递事件和结果。

举个简单的例子，UI层通知逻辑层（Presenter）用户点击了一个Button，逻辑层（Presenter）自己决定应该用什么行为进行响应，该找哪个模型（Model）去做这件事，最后逻辑层（Presenter）将完成的结果更新到UI层。

# MVP流程：  
![](http://7q5c2h.com1.z0.glb.clouddn.com/androidmvpsample2.png)  
说明：  
步骤1：UI实现View方法，引用Presenter  
步骤2：Presenter调用Model，走Model具体逻辑  
步骤3：Model逻辑实现，回调Presenter方法  
步骤4：Presenter回调View，即回到UI，回调View方法  

# MVP的优点
1、模型与视图完全分离，我们可以修改视图而不影响模型；

2、可以更高效地使用模型，因为所有的交互都发生在一个地方——Presenter内部；

3、我们可以将一个Presenter用于多个视图，而不需要改变Presenter的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化频繁；

4、如果我们把逻辑放在Presenter中，那么我们就可以脱离用户接口来测试这些逻辑（单元测试）。

# 实践

[https://github.com/afayp/Architecture](https://github.com/afayp/Architecture)

# MVP优点

- 以前MVC架构的android项目，我们会把业务逻辑全部写到Activity中，随着业务逻辑的复杂，导致Activity越来越庞大。而利用MVP的设计模型可以把业务逻辑代码从Fragment或者Activity中移出来，在Presenter中持有View（Activity或者Fragment）的引用，然后在Presenter调用View暴露的接口对视图进行操作。有利于把视图操作和业务逻辑分开来。MVP能够让Activity成为真正的View而不是View和Control的合体，Activity只做UI相关的事。这样一来各层相对独立，通过接口通信，也方便测试和开发写假数据。

# MVP缺点
- 代码量增加，文件数量大大增加(这个可以通过as的template解决)
- 每次通信都需要繁琐的通过接口传递信息

# 实际使用感受
- 对于逻辑简单的页面可以不使用Presenter,直接在Activity或Fragment中处理逻辑，灵活一点。
- 理论上来说Presenter因为没有和具体ui绑定，是可以重用的，比如多个Activity重用一个Presenter。但这就要求presenter必须只含有一些共有的逻辑，但实际项目中你要求两个页面包含相同的业务逻辑，基本不可能。如果强制将不同页面的逻辑放在同一个Prsenter中,来达到重用的目的,那么每个Activity会被迫实现许多并不需要的方法,得不偿失。
- 对MVP来说，目的是为了代码解耦，提高可维护性，但是会牺牲代码的精简程度。

# 参考资源
[http://blog.csdn.net/lmj623565791/article/details/46596109](http://blog.csdn.net/lmj623565791/article/details/46596109)  

[http://wuxiaolong.me/2015/09/23/AndroidMVPSample/#comments](http://wuxiaolong.me/2015/09/23/AndroidMVPSample/#comments)

[https://www.sdk.cn/news/2501](https://www.sdk.cn/news/2501)

[http://blog.waynell.com/2015/05/29/mvp-on-android/](http://blog.waynell.com/2015/05/29/mvp-on-android/)

[https://github.com/xitu/gold-miner/blob/master/TODO/android-basic-project-architecture-for-mvp.md](https://github.com/xitu/gold-miner/blob/master/TODO/android-basic-project-architecture-for-mvp.md)

[Android应用架构](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1214/3772.html)

[https://gold.xitu.io/post/58b25e588d6d810057ed3659]()


