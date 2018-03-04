---
layout:     post
title:      "Android-Art第十一章-Android的线程和线程池"
date:       2016-06-26 21:33:10
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



## 11.1 主线程和子线程
主线程是指进程所拥有的线程，在Java中默认情况下一个进程只有一个线程，也就是主线程，其他线程都是子线程，也叫工作线程。Android中的主线程（也叫UI线程）主要处理和界面相关的事情，而子线程则往往用于执行耗时操作。

<!--more-->

从Android 3.0开始，系统要求网络访问必须在子线程中进行，否则网络访问将会失败并抛出`NetworkOnMainThreadException`这个异常，这样做是为了避免主线程由于被耗时操作所阻塞从而出现ANR现象。

## 11.2 Android中的线程形态
除了传统的`Thread`外还包含`AsyncTask`、`HandlerThread`和`IntentService`。它们具有不同的表现形式，有各自的优缺点。

### 11.2.1 AsyncTask
`AsyncTask`是一种轻量级的异步任务类，封装了线程池和`Handler`，方便开发者在子线程中更新UI。但它并不适合特别耗时的后台任务，对于特别耗时的后台任务建议用线程池。
`AsyncTask`是一个抽象泛型类（`AsyncTask<Params，Progress，Result>`），它提供了`Params、Progress、Result`三个泛型参数，如果`task`确实不需要传递具体的参数，那么都可以设置为`Void`。
下面是它的四个核心方法，其中`doInBackground`不是在主线程执行的。
`onPreExecute、doInBackground、onProgressUpdate、onPostResult`
#### 使用
一个典型的例子：
```java
private class DownloadFileTask extends AsyncTask<URL,Integer,Long>{

    @Override
    protected void onPreExecute() {
        //准备工作
    }

    @Override
    protected Long doInBackground(URL... urls) {
        int count = urls.length;
        long totalSize = 0;
        for (int i = 0; i < count; i++) {
            totalSize +=1;
            publishProgress(100*i/count);
            if (isCancelled()){
                break;
            }
        }
        return totalSize;
    }

    @Override
    protected void onProgressUpdate(Integer... progress) {
        showProgress(progress[0]);//
    }

    @Override
    protected void onPostExecute(Long result) {
        showDialog("Downloaded "+result+"count");
    }
}
```
执行：
	`new DownloadFileTask().execute(url1,url2,url3);`
使用AsyncTask注意点：
> 
	1. AsyncTask的类必须在主线程中加载，即第一次用AsyncTask必须在主线程，这个过程在Android 4.1及以上版本中已经被系统自动完成。
	2. AsyncTask对象必须在主线程中创建
	3. execute方法必须在UI线程中调用
	4. 不能在程序中直接调用onPreExecute()、onPostExecute()、doInBackground()和onProgressUpdate()方法
	5. 一个AsyncTask对象只能执行一次，即只能调用一次execute方法，否则会报运行时异常
	6. 在Android 1.6之前，AsyncTask是串行执行任务的，Android 1.6的时候AsyncTask开始采用线程池并行处理任务，但是从Android 3.0开始，为了避免AsyncTask带来的并发错误，AsyncTask又采用一个线程来串行执行任务。尽管如此，在Android 3.0以及后续版本中，我们可以使用AsyncTask的executeOnExecutor方法来并行执行任务。但是这个方法是Android 3.0新添加的方法，并不能在低版本上使用。

#### AsyncTask的原理
`AsyncTask`中有两个线程池：`SerialExecutor`和`THREAD_POOL_EXECUTO`R。前者是用于任务的排队，默认是串行的线程池；后者用于真正执行任务。`AsyncTask`中还有一个`Handler`，即`InternalHandler`，用于将执行环境从线程池切换到主线程。`AsyncTask`内部就是通过`InternalHandler`来发送任务执行的进度以及执行结束等消息。
`AsyncTask`排队执行过程：系统先把参数`Params`封装为`FutureTask`对象，它相当于`Runnable`；接着将`FutureTask`交给`SerialExecutor`的`execute`方法，它先把`FutureTask`插入到任务队列`tasks`中，如果这个时候没有正在活动的`AsyncTask`任务，那么就会执行下一个`AsyncTask`任务，同时当一个`AsyncTask`任务执行完毕之后，`AsyncTask`会继续执行其他任务直到所有任务都被执行为止。

### 11.2.2 HandlerThread
`HandlerThread`继承了`Thread`，它是一种可以使用`Handler`的`Thread`，它的实现就是在`run`方法中通过`Looper.prepare()`来创建消息队列，并通过`Looper.loop()`来开启消息循环，这样在实际的使用中就允许在`HandlerThread`中创建`Handler`了，外界可以通过`Handler`的消息方式通知`HandlerThread`执行一个具体的任务。`HandlerThread`的`run`方法是一个无限循环，因此当明确不需要再使用`HandlerThread`的时候，可以通过它的`quit`或者`quitSafely`方法来终止线程的执行。`HandlerThread`的最主要的应用场景就是用在`IntentService`中。

### 11.2.3 IntentService
`IntentService`是一个继承自`Service`的抽象类，要使用它就要创建它的子类。`IntentService`是服务，不容易被杀死，所以适合执行一些高优先级的后台任务。（别忘了在配置文件中注册！）
`IntentService`的`onCreate`方法中会创建`HandlerThread`，并使用`HandlerThread`的`Looper`来构造一个`Handler`对象`ServiceHandler`，这样通过`ServiceHandler`对象发送的消息最终都会在`HandlerThread`中执行。
```java
public void onCreate(){
    super.onCreate();
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();
    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}
```
`IntentService`会将`Intent`封装到`Message`中，这里的`intent`和`startService（intent）`中的内容完全一致，然后通过`ServiceHandler`发送出去，在`ServiceHandler`的`handleMessage`方法中会调用`IntentService`的抽象方法`onHandleIntent`，所以`IntentService`的子类都要是实现这个方法。
```java
public void onStart(Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}
 
private final class ServiceHandler extends Handler {
    public ServiceHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        onHandleIntent((Intent)msg.obj);
        stopSelf(msg.arg1);
    }
}
```
一个例子：
```java
public class MyIntentService extends IntentService {

    public MyIntentService(String name) {
        super(name);
    }
    
    @Override
    protected void onHandleIntent(Intent intent) {
        String url = intent.getStringExtra("url");
        loadFromNet(url);//执行后台任务
    }

    private void loadFromNet(String url) {
        //...
    }
}
```
执行：
```java
Intent intent = new Intent(this, MyIntentService.class);
intent.putExtra("url","url1");
startService(intent);
intent.putExtra("url","url2");
startService(intent);
intent.putExtra("url","url3");
startService(intent);
```
 这三个后台任务会按顺序执行。当最后一个任务执行完后会自动执行onDestory()，停止服务。

## 11.3 Android中的线程池
使用线程池的好处：
> 
a. 重用线程，避免线程的创建和销毁带来的性能开销；
b. 能有效控制线程池的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象；
c. 能够对线程进行简单的管理，并提供定时执行以及指定间隔循环执行等功能。

`Executor`只是一个接口，真正的线程池是`ThreadPoolExecutor`。`ThreadPoolExecutor`提供了一系列参数来配置线程池，通过不同的参数可以创建不同的线程池，Android的线程池都是通过`Executors`提供的工厂方法得到的。

### 11.3.1 ThreadPoolExecutor
构造方法通过一系列参数来配置线程池，常用的构造如下：
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
 
```
参数解释：

- corePoolSize：核心线程数，默认情况下，核心线程会在线程中一直存活；但是如果将ThreadPoolExecutor的allCoreThreadTimeOut属性设置为true，当等待时间超过keepAliveTime所指定的时长后，核心线程会被终止
- maximumPoolSize：最大线程数，当活动线程数达到这个数值后，后续的任务将会被阻塞；
- keepAliveTime：非核心线程闲置时的超时时长，超过这个时长，闲置的非核心线程就会被回收；
- unit：用于指定keepAliveTime参数的时间单位，有TimeUnit.MILLISECONDS、TimeUnit.SECONDS、TimeUnit.MINUTES等；
- workQueue：任务队列，通过线程池的execute方法提交的Runnable对象会存储在这个参数中；
- threadFactory：线程工厂，为线程池提供创建新线程的功能。它是一个接口，它只有一个方法Thread newThread(Runnable r)；
- RejectedExecutionHandler：当线程池无法执行新任务时，可能是由于任务队列已满或者是无法成功执行任务，这个时候就会调用这个Handler的rejectedExecution方法来通知调用者，默认情况下，rejectedExecution会直接抛出一个rejectedExecutionException。

ThreadPoolExecutor执行任务的规则：

a. 如果线程池中的线程数未达到核心线程的数量，那么会直接启动一个核心线程来执行任务；
b. 如果线程池中的线程数量已经达到或者超过核心线程的数量，那么任务会被插入到任务队列中排队等待执行；
c. 如果在步骤2中无法将任务插入到的任务队列中，可能是任务队列已满，这个时候如果线程数量没有达到规定的最大值，那么会立刻启动非核心线程来执行这个任务；
d. 如果步骤3中线程数量已经达到线程池规定的最大值，那么就拒绝执行此任务，ThreadPoolExecutor会调用RejectedExecutionHandler的rejectedExecution方法来通知调用者。

`AsyncTask`的`THREAD_POOL_EXECUTOR`线程池的配置：

a. corePoolSize=CPU核心数+1；
b. maximumPoolSize=2倍的CPU核心数+1；
c. 核心线程无超时机制，非核心线程在闲置时间的超时时间为1s；
d. 任务队列的容量为128。

### 11.3.2 线程池分类
`Android`中常见的4类具有不同功能特性的线程池：

#### 1. FixedThreadPool
通过`Executors.newFixedThreadPool(5)`来创建。
特点：线程数量固定的线程池，线程即使处于空闲状态也不会被回收，除非线程池关闭；当所有线程都处于活动状态时，新任务会处于等待状态，知道有线程空出来；它只有核心线程，意味着能更快的响应外界请求；没有超时机制；任务队列没有大小限制。
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                              0L, TimeUnit.MILLISECONDS,
                              new LinkedBlockingQueue<Runnable>());
}
```
	
#### 2. CachedThreadPool
通过`Executors.newCachedThreadPool()`来创建。
特点：线程数量不固定的线程池，数量相当于任意大；它只有非核心线程；都有超时机制，时长60秒，超过60秒闲置线程就会被回收。
```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
```

#### 3. ScheduledThreadPool
通过`Executors.newScheduledThreadPool(5)`来创建
特点：核心线程数量固定，非核心线程数量没有限制的线程池，主要用于执行定时任务和具有固定周期的任务；
```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

#### 4. SingleThreadPool
通过`Executors.newSingleThreadExecutor()`来创建
特点：只有一个核心线程的线程池，确保了所有的任务都在同一个线程中按顺序执行。意义在于统一外界所有的任务到一个线程中，使得这些任务之间不需处理线程同步问题。
```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

		 





