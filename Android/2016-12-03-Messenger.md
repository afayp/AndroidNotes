---
layout:     post
title:      "Messenger解析"
date:       2016-12-03 21:23:23
author:     "afayp"
catalog:    true
tags:
    - Android
---




# Messenger

Messenger译为信使，通过它可以在不同进程中传递Messenger对象，在Messenger中放入需要传递的数据，就可以实现进程间的数据传递了。

<!--more-->

## 实现
Messager的使用可分以下几步：

- 服务端实现一个Handler，由其接受来自客户端的每个调用的回调
- 使用实现的Handler创建Messenger对象
- 通过Messenger得到一个IBinder对象，并将其通过onBind()返回给客户端
- 客户端使用 IBinder 将 Messenger（引用服务的 Handler）实例化，然后使用后者将 Message 对象发送给服务
- 服务端在其 Handler 中（具体地讲，是在 handleMessage() 方法中）接收每个 Message

看下代码：

### 服务端
在服务端创建一个service来处理来自客户端的请求，在service中创建一个handler，并通过它来创建一个Messenger对象，然后在service的onBind中返回Messenger对象的底层Binder即可。

```
//服务端
public class MessengerServiceDemo extends Service {

    static final int MSG_SAY_HELLO = 1;

    class ServiceHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_SAY_HELLO:
                    //当收到客户端的message时
                    Toast.makeText(getApplicationContext(), msg.getData().getString("msg"), Toast.LENGTH_SHORT).show();
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

    final Messenger mMessenger = new Messenger(new ServiceHandler());

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Toast.makeText(getApplicationContext(), "binding", Toast.LENGTH_SHORT).show();
        //返回给客户端一个IBinder实例
        return mMessenger.getBinder();
    }
}
```

然后注册：
```
<service
    android:name=".ActivityMessenger"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="com.lypeer.messenger"></action>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</service>
```

### 客户端

客户端进程中，首先绑定服务端的service，绑定成功后用服务端返回的IBinder对象创建一个Messenger，通过这个Messenger就可以像服务端发消息了，发送的消息类型为Messenge对象。
```
//客户端
public class ActivityMessenger extends Activity {

    static final int MSG_SAY_HELLO = 1;

    Messenger mService = null;
    boolean mBound;

    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className, IBinder service) {
            //接收onBind()传回来的IBinder，并用它构造Messenger
            mService = new Messenger(service);
            mBound = true;
        }

        public void onServiceDisconnected(ComponentName className) {
            mService = null;
            mBound = false;
        }
    };

    //调用此方法时会发送信息给服务端
    public void sayHello(View v) {
        if (!mBound) return;
        //发送一条信息给服务端
        Message msg = Message.obtain(null, MSG_SAY_HELLO);
		Bundle data = new Bundle();
		data.putString("msg","hello,this is client");
		msg.setData(data);
        try {
            mService.send(msg);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onStart() {
        super.onStart();
        //绑定服务端的服务，此处的action是service在Manifests文件里面声明的
        Intent intent = new Intent();
        intent.setAction("com.lypeer.messenger");
        //不要忘记了包名，不写会报错
        intent.setPackage("com.lypeer.ipcserver");
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStop() {
        super.onStop();
        // Unbind from the service
        if (mBound) {
            unbindService(mConnection);
            mBound = false;
        }
    }
}
```


## Messenger与AIDL的比较
Messenger本质上是AIDL，系统为我们做了封装，方便上层调用。另外Messenger以串行的方式处理客户端传来的消息，如果有大量的消息同时发送到服务端，用Messenger就不合适了。另外Messenger的作用主要是为了传递消息，如果需要跨进程调用服务端的方法，Messenger就无法办到，更推荐使用AIDL。
