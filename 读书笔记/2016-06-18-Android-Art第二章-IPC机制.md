---
layout:     post
title:      "Android-Art第二章-IPC机制"
date:       2016-06-18 23:39:32
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



# 2.1简介
`IPC`是`Inter-Process Communication`的缩写。含义为进程间通信或跨进程通信，是指两个进程之间进行数据交换的过程。

<!--more-->

进程与线程区别？
> - 按照操作系统的描述，线程是CPU调度的最小单元，同时线程是一种有限的系统资源。
- 进程一般指一个执行单元，在PC和移动设备上指一个程序或者一个应用。一个进程可以包含多个线程，因此进程和线程是包含与被包含的关系。

任何一个操作系统都需要有相应的IPC机制，Linux上可以通过命名通道、共享内存、信号量等来进行进程间通信。Android系统不仅可以使用了`Binder`机制来实现IPC，还可以使用`Socket`实现任意两个终端之间的通信。

IPC使用场景
> 
- 第一种情况是一个应用因为某些原因自身需要采用多线程模式来实现（可能是为了获得更多的内存空间）
- 另一种情况是当前应用需要向其他应用获取数据，如使用`ContentProvider`去查询数据


# 2.2 Android中的多进程模式

## 2.2.1 开启多进程
通过给四大组件指定`android:process`属性，我们可以开启多线程模式,默认进程的进程名是包名`packageName`，进程名以`:`开头的进程属于当前应用的私有进程，其他应用的组件不可以和它跑在同一个进程中，而进程名不以`:`开头的进程属于全局进程，其他应用通过`ShareUID`方法可以和它跑在同一个进程中。
```java
android:process=":xyz" //进程名是 packageName:xyz,私有进程
android:process="aaa.bbb.ccc" //进程名是 aaa.bbb.ccc，全局进程
```

Android系统会为每个应用分配一个唯一的UID，具有相同UID的应用才能共享数据（`android:sharedUserId`一样）。两个应用通过`ShareUID`跑在同一个进程中是有要求的，需要这两个应用有相同的ShareUID并且签名相同才可以。 在这种情况下，它们可以相互访问对方的私有数据，比如data目录、组件信息等，不管它们是否跑在同一个进程中。如果它们跑在同一个进程中，还可以共享内存数据，它们看起来就像是一个应用的两个部分。

除了可以在DDMS中查看进程信息，还可以用shell来查看： `adb shell ps` 或者 `adb shell ps | 包名`


## 多进程的运行机制

- Android为每一个应用分配了一个独立的虚拟机，或者说为每个进程都分配了一个独立的虚拟机，不同的虚拟机在不同的内存分配上有不同的地址空间，这就导致在不同的虚拟机中访问同一个类的对象会产生多份副本。

- 所有运行在不同进程中的四大组件，只要它们之间需要通过内存来共享数据，都会共享失败。

使用多进程会造成以下几个问题

- 静态成员和单例模式完全失效
- 线程同步机制完全失效
因为不管是锁对象还是锁全局类都无法保证线程同步，因为不同进程锁的不是同一个对象
- SharedPreference的可靠性下降
SharedPreferences不支持两个进程同时去执行写操作，否则会导致一定几率的数据丢失，这时因为SharedPreferences底层是通过读写XML文件来实现的，并发写显然是可能出问题的，甚至并发读写都有可能发生问题
- Application会多次创建
运行在同一个进程中的组件是属于同一个虚拟机和同一个Application的。同理，运行在不同进程中的组件是属于两个不同的虚拟机和Application的。

# 2.3 IPC基础概念介绍

## 2.3.1 Serializable接口
`Serializable`接口是`Java`中为对象提供标准的序列化和反序列化操作的接口，而`Parcelable`接口是Android提供的序列化方式的接口。

实现序列化只需实现`Serializable`接口和声明一个`serialVersionUId`。serialVersionUId是一串long型数字，主要是用来辅助序列化和反序列化的，原则上序列化后的数据中的`serialVersionUId`只有和当前类的`serialVersionUId`相同才能够正常地被反序列化。
`private static final long serialVersionUID = 8711368828010083044L`

```java
//序列化过程
Person person = new Person();
person.setId(1);
person.setName("nima");
//ObjectOutputStream 对象输出流，将Person对象存储到E盘的Person.txt文件中，完成对Person对象的序列化操作
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("person.txt")));
	    oos.writeObject(person);
	    oos.close();


//反序列化过程
ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("person.txt")));
Person p = (Person) ois.readObject();
ois.close();
return p;

```
`serialVersionUId`的详细工作机制：序列化的时候系统会把当前类的`serialVersionUId`写入序列化的文件中，当反序列化的时候系统会去检测文件中的`serialVersionUId`，看它是否和当前类的`serialVersionUId`一致，如果一致就说明序列化的类的版本和当前类的版本是相同的，这个时候可以成功反序列化；否则说明版本不一致无法正常反序列化（会报`InvalidClassException`）。

一般来说，我们应该手动指定`serialVersionUId`的值。

- 如果不指定，如果当前类某个成员变量增加了，系统就会重新计算当前类的`serialVersionUId`，就会导致当前类的`serialVersionUId`和序列化存储的数据中的`serialVersionUId`不一致，程序就会crash。
- 如果指定了，即使我们增加或者删除了某个成员变量，程序也能反序列化成功，能够最大限度的恢复数据。
- 当然如果类结构发生了非常规性变化，比如说修改了类名，修改了成员变量类型，尽管`serialVersionUId`验证通过，反序列化还是会失败。

另外要注意：
> 
1. 静态成员变量属于类不属于对象，所以不参与序列化过程；
2. 声明为transient的成员变量不参与序列化过程。

## 2.3.2Parcelable接口
`Parcelable`接口内部包装了可序列化的数据，可以在`Binder`中自由传输，`Parcelable`主要用在**内存序列化**上，可以直接序列化的有`Intent`、`Bundle`、`Bitmap`以及`List`和`Map`等等,前提是他们里面的每个元素都是可序列化的。

```java
	public class Book implements Parcelable {
    public int bookId;
    public String bookName;
    public Book() {
    }
	public Book(int bookId, String bookName) {
        this.bookId = bookId;
        this.bookName = bookName;
    }
	//“内容描述”，如果含有文件描述符返回1，否则返回0，几乎所有情况下都是返回0
    public int describeContents() {
        return 0;
    }
	//实现序列化操作，flags标识只有0和1，1表示标识当前对象需要作为返回值返回，不能立即释放资源，几乎所有情况都为0
    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(bookId);
        out.writeString(bookName);
    }
	//实现反序列化操作
    public static final Parcelable.Creator<Book> CREATOR = new Parcelable.Creator<Book>() {
        //从序列化后的对象中创建原始对象
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }
        public Book[] newArray(int size) {//创建指定长度的原始对象数组
            return new Book[size];
        }
    };
	private Book(Parcel in) {
        bookId = in.readInt();
        bookName = in.readString();
    }
	}
```

Parcelable和Serializable如何选择？

- Serializable使用简单，但开销很大。当序列化到存储设备或者序列化后的对象用于网络传输时用Serializable。
- Parcelable效率高，更适用于Android平台，主要用在内存序列化上。

## 2.3.3 Binder
`Binder`是什么？

- `Binder`是Android中的一个类，它实现了`IBinder`接口；
-  从`IPC`角度看，`Binder`是Android中一种跨进程通信的方式；
-  `Binder`还可以理解为虚拟的物理设备，它的设备驱动是`/dev/binder`，在Linux中没有；
-  从`Framework`层角度看，`Binder`是`ServiceManager`连接各种`Manager（ActivityManager、WindowManager）`和相应的`ManagerService`的桥梁；
-  从Android应用层来说，`Binder`是客户端和服务端进行通信的媒介，当`bindService`的时候，服务端会返回一个包含了服务端业务调用的`Binder`对象，通过这个`Binder`对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于`AIDL`的服务。

好难，以后再看~

# 2.4 Android中的IPC方式

## 2.4.1 Bundle
`Activity`,`Service`,`Receiver`都支持在`Intent`中传递`Bundle`数据，`Bundle`实现了`Parcelable`接口。

我们在一个进程中启动另一个进程的`Activity`、`Servic`或者`Receiver`时，可以在`Bundle`中附上要传递的信息，并通过`Intent`发送出去。当然，传输的数据必须能够被序列化，也就是`Bundle`支持的数据类型。

一种特殊的使用场景：A进程要进行一个计算，计算完成后启动B进程，并把计算结果传递给进程B，但是这个计算结构不支持序列化，无法通过Intent传输。可以考虑这样：通过Intent启动进程B的一个Service（如IntentService），让Service在后台计算，计算完毕后再启动进程B中的其他组件C，因为B和C运行在同一进程，C就可以直接获取结果了，就避开了跨进程。

## 2.4.2 文件共享
两个进程通过读写同一个文件来交换数据，Linux对文件并发读写没有限制，甚至可以两个进程同时对同一个文件写操作。所以该种方式适合对数据同步不高的进程间通信，不适合高并发场景。

`SharedPreference`比较特殊，通过键值对来存储数据，底层采用XML文件来存储，本质也是文件的一种。但是系统对它的读写有一定的缓存策略，在内存中会有一份`SharedPreference`文件的缓存。多进程模式下，读写会变得很不可靠，所以一般不建议在多进程通信中使用。

## 2.4.3 Messenger

`Messenger`是一种轻量级的IPC方案，它的底层实现就是AIDL。Messenger是以串行的方式处理请求的，即服务端只能一个个处理，不存在并发执行的情形。

## 2.4.4 AIDL
大致流程：首先建一个`Service`和一个`AIDL`接口，接着创建一个类继承自AIDL接口中的Stub类并实现Stub类中的抽象方法，在Service的onBind方法中返回这个类的对象，然后客户端就可以绑定服务端Service，建立连接后就可以访问远程服务端的方法了。

1. AIDL支持的数据类型：基本数据类型、String和CharSequence、ArrayList、HashMap、Parcelable以及AIDL；

2. 某些类即使和AIDL文件在同一个包中也要显式import进来；

3. AIDL中除了基本数据类，其他类型的参数都要标上方向：in、out或者inout；

4. AIDL接口中支持方法，不支持声明静态变量；

5. 为了方便AIDL的开发，建议把所有和AIDL相关的类和文件全部放入同一个包中，这样做的好处是，当客户端是另一个应用的时候，可以直接把整个包复制到客户端工程中。

6. RemoteCallbackList是系统专门提供的用于删除跨进程Listener的接口。RemoteCallbackList是一个泛型，支持管理任意的AIDL接口，因为所有的AIDL接口都继承自IInterface接口。

## 2.4.5 ContentProvider

1. ContentProvider主要以表格的形式来组织数据，并且可以包含多个表；

2. ContentProvider还支持文件数据，比如图片、视频等，系统提供的MediaStore就是文件类型的ContentProvider；

3. ContentProvider对底层的数据存储方式没有任何要求，可以是SQLite、文件，甚至是内存中的一个对象都行；

4. 要观察ContentProvider中的数据变化情况，可以通过ContentResolver的registerContentObserver方法来注册观察者；

## 2.4.6 Socket
套接字，分为流式套接字和用户数据报套接字两种，分别对应于网络的传输控制层中TCP和UDP协议。
具体使用见书上P103

# Binder连接池
当项目规模很大的时候，创建很多个`Service`是不对的做法，因为`service`是系统资源，太多的`service`会使得应用看起来很重，所以最好是将所有的`AIDL`放在同一个`Service`中去管理。整个工作机制是：每个业务模块创建自己的`AIDL`接口并实现此接口，这个时候不同业务模块之间是不能有耦合的，所有实现细节我们要单独开来，然后向服务端提供自己的唯一标识和其对应的`Binder`对象；对于服务端来说，只需要一个`Service`，服务端提供一个`queryBinder`接口，这个接口能够根据业务模块的特征来返回相应的`Binder`对象给它们，不同的业务模块拿到所需的`Binder`对象后就可以进行远程方法调用了。

`Binder`连接池的主要作用就是将每个业务模块的`Binder`请求统一转发到远程`Service`去执行，从而避免了重复创建`Service`的过程。
(2)作者实现的`Binder`连接池[BinderPool](https://github.com/singwhatiwanna/android-art-res/blob/master/Chapter_2/src/com/ryg/chapter_2/binderpool/BinderPool.java)的实现源码，建议在AIDL开发工作中引入BinderPool机制。

# 选用合适的IPC机制

![](http://oeiu2t0ur.bkt.clouddn.com/2015-12-10_56692ec1c55ed.png)