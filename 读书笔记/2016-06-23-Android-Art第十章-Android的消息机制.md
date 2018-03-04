---
layout:     post
title:      "Android-Art第十章-Android的消息机制"
date:       2016-06-23 20:38:09
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



# 简介
`Handler`是`Android`消息机制的上层接口，所以我们在开发中只需和`Handler`交互即可。
`Android`的消息机制主要指`Handler`的运行机制，`Handler`的运行需要底层的`MessageQueue`和`Looper`的支撑。`MessageQueue`虽然叫消息队列，但它的内部存储并不是队列结构，而是以单链表的数据结构存储消息列表，但是以队列的形式对外提供插入和删除消息操作。`MessageQueue`只是消息的存储单元，它不能处理消息。`Looper`则是以无限循环的形式去查找是否有新消息，如果有的话就去处理消息，否则就一直等待着。

<!--more-->

# 10.1 Android消息机制概述
`Handler`的主要作用是将一个任务切换到某个指定的线程中去执行。
为什么要提供这个功能呢？因为Android规定访问UI只能在主线程中进行，如果在子线程中访问UI程序就会抛异常（CalledFromWrongThreadException）。这个验证是通过`ViewRootImpl`的`checkThread`方法来完成的。

那为什么不允许在子线程中访问UI呢？因为Android的UI线程是不是线程安全的，在多线程中并发访问会导致UI控件处于不可预估状态。那为什么不用锁呢？一是因为锁机制会降低UI访问效率，二是因为锁机制会阻塞某些线程的执行。所以最简单高效的方法就是用单线程模型来处理UI。

`Handler`创建时会采用当前线程的`Looper`来构建内部的消息循环机制（`Handler mHandler = new Handler()`），如果当前线程没有`Looper`，就会报错。当然你也可以用主线程的`Looper`来创建`Handler`，这样创建的`Handler`就与UI线程绑定，可以在里面更新UI。
```java
private Handler mHandler = new Handler(Looper.getMainLooper()){
    @Override
    public void handleMessage(Message msg) {
        ImageView imageView = (ImageView) msg.obj;
        imageView.setBackgroundColor(Color.BLUE);
    }
};
```

通俗一点讲就是用线程A的`Looper`创建了`mHandler`，在线程B中拿到`mHanlder`，调用`mHandler.sendMessage`方法，这个消息就会传到线程A中，在线程A中处理。这样业务逻辑就从线程B切换到线程A中了，而线程A往往是主线程。

`Handler`可以用`post`方法将一个`Runnable`投递到消息队列中，也可以用`send`方法发送一个消息投递到消息队列中，其实`post`最终还是调用了`send`方法。`send`方法会调用`MessageQueue`的`enqueueMessage`方法将这个消息放入消息队列中，`Looper`发现有新消息到来时，就会处理这个消息，最终消息中的`Runnable`或者`Handler`的`handleMessage`方法就会被调用。

# 10.2 Android的消息机制分析
## 10.2.1 ThreadLocal的工作原理
`ThreadLocal`是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到数据。一般来说，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，可以考虑使用`ThreadLocal`。 对于`Handler`来说，它需要获取当前线程的`Looper`，而`Looper`的作用域就是线程并且不同线程具有不同的`Looper`，这个时候通过`ThreadLocal`就可以实现`Looper`在线程中的存取了。一个例子：
```java
	private ThreadLocal<Boolean> mBooleanThreadLoacl = new ThreadLocal<>();
	//主线程
	mBooleanThreadLoacl.set(true);
	Log.d("TAG","[Thread#main]mBooleanThreadLocal: " +mBooleanThreadLoacl.get());//结果为true
	//新线程
	new Thread("Thread#1"){
	    @Override
	    public void run() {
	        mBooleanThreadLoacl.set(false);
	        Log.d("TAG","[Thread#1]mBooleanThreadLocal: " +mBooleanThreadLoacl.get());//结果为false
	    }
	};
```
可以看出在不同线程中访问同一个`ThreadLocal`对象，`get`到的值却是不一样的。

原理：不同线程访问同一个`ThreadLocal`的`get`方法时，`ThreadLocal`内部会从各自的线程中取出一个数组，然后再从数组中根据当前`ThreadLocal`的索引去查找出对应的`value`值，不同线程中的数组是不同的，这就是为什么通过`ThreadLocal`可以在不同线程中维护一套数据的副本并且彼此互不干扰。
内部实现，set方法：
```java
public void set(T value) {
    Thread currentThread = Thread.currentThread();
    Values values = values(currentThread);
    if (values == null) {
        values = initializeValues(currentThread);
    }
    values.put(this, value);
}
```
首先通过`values`方法获取当前线程的`ThreadLocal`数据，`Values`是`Thread`类内部专门用来存储线程的`ThreadLocal`数据的，它内部有一个数组`private Object[] table，ThreadLocal`的值就存在这个`table`数组中。如果`values`的值为`null`，那么就需要对其进行初始化然后再将`ThreadLocal`的值进行存储。`ThreadLocal`数据的存储规则：`ThreadLocal`的值在`table`数组中的存储位置总是`ThreadLocal的索引+1的位置`。对应的get方法就是这个table数组中取出数据。

## 10.2.2 MessageQueue的工作原理
`MessageQueue`主要包含两个主要操作`enqueueMessage`和`next`，前者是插入消息，后者是取出一条消息并移除。`MessageQueue`的内部其实是通过单链表来维护消息列表的，单链表在插入和删除上比较有优势。
`next`方法是一个无限循环的方法，如果消息队列中没有消息，那么`next`方法会一直阻塞在这里。当有新消息到来时，`next`方法会返回这条消息并将它从链表中移除。

## 10.2.3 Looper的工作原理
首先看一下它的构造，做了两件事：创建一个`MessageQueue`；将当前线程对象保存起来。
```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```
为一个线程创建`Looper`的方法，代码如下：
```java
new Thread("test"){
@Override
public void run() {
    Looper.prepare();//创建looper
    Handler handler = new Handler();//可以创建handler了
    Looper.loop();//开始looper循环
}
}.start();
```

除了`prepare`方法，`Looper`还有`prepareMainLooper`方法，主要是给主线程也就是`ActivityThread`创建`Looper`使用的，本质也是调用了`prepare`方法。

`Looper`提供了`getMainLooper`，在任何地方都可以获取到主线程的`Looper`。

`Looper`的`quit`和`quitSafely`方法的区别是：前者会直接退出`Looper`，后者只是设定一个退出标记，然后把消息队列中的已有消息处理完毕后才安全地退出。`Looper`退出之后，通过`Handler`发送的消息就会失败，这个时候`Handler`的`send`方法会返回`false`。

在子线程中，如果手动为其创建了`Looper`，那么在所有的事情完成以后应该调用`quit`方法来终止消息循环，否则这个子线程就会一直处于等待的状态，而如果退出`Looper`以后，这个线程就会立刻终止，因此建议不需要的时候终止`Looper`。

`Looper`的`loop`方法是个死循环，唯一跳出循环的方式是`MessageQueue`的`next`方法返回`null`，`Looper`调用`quit`后会调用`MessageQueue`的`quit`，此时`MessageQueue`就会被标记为退出状态，`MessageQueue`的`next`返回`null`，`loop`方法也就停止了。

`loop`方法会调用`MessageQueue`的`next`方法来获取新消息，而`next`是一个阻塞操作，当没有消息时，`next`方法会一直阻塞着在那里，这也导致了`loop`方法一直阻塞在那里。如果`MessageQueue`的`next`方法返回了新消息，`Looper`就会处理这条消息：`msg.target.dispatchMessage(msg)`，其中的`msg.target`就是发送这条消息的`Handler`对象。`handler`的`dispatchMessage`方法会在创建`handler`时使用的`Looper`中执行，这样就切换了线程。

## 10.2.4 Handler的工作原理
`Handler`的工作主要包含：消息的发送和接收之后的处理。

发送有多种方式：`sendMessage(Message msg)`；`sendMessageDelayed(Message msg，long delayMillis)`;`sendMessageAtTime(Messgae msg ,long uptimeMills)`
最终都会调用`enqueueMessage`方法向`MessageQueue`中插入这条消息`。MessageQueue`的`next`方法就会返回消息给`Looper`，`Looper`最后交给`Handler`处理，即`dispatchMessage`方法会被调用，实现如下：
```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);//当message是runnable的情况，也就是Handler的post方法传递的参数，这种情况下直接执行runnable的run方法
    } else {
        if (mCallback != null) {//如果创建Handler的时候是给Handler设置了Callback接口的实现，那么此时调用该实现的handleMessage方法
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);//如果是派生Handler的子类，就要重写handleMessage方法，那么此时就是调用子类实现的handleMessage方法
    }
}
	private static void handleCallback(Message message) {
        message.callback.run();
}
	/**
 * Subclasses must implement this to receive messages.
 */
public void handleMessage(Message msg) {
}
```
`Handler`还有一个特殊的构造方法，它可以通过特定的`Looper`来创建Handler。
```java
public Handler(Looper looper){
  this(looper, null, false);
}
```	
# 10.3 主线程的消息循环
`Android`的主线程就是`ActivityThread`，主线程的入口方法就是`main`，其中调用了`Looper.prepareMainLooper()`来创建主线程的`Looper`以及`MessageQueue`，并通过`Looper.loop()`方法来开启主线程的消息循环。主线程内有一个`Handler`，即`ActivityThread.H`，它定义了一组消息类型，主要包含了四大组件的启动和停止等过程，例如`LAUNCH_ACTIVITY`等。
`ActivityThread`通过`ApplicationThread`和`AMS`进行进程间通信，`AMS`以进程间通信的方式完成`ActivityThread`的请求后会回调`ApplicationThread`中的`Binder`方法，然后`ApplicationThread`会向H发送消息，H收到消息后会将`ApplicationThread`中的逻辑切换到`ActivityThread`中去执行，即切换到主线程中去执行，这个过程就是主线程的消息循环模型。





	





