---

layout: post
title:  "Android进程间通信之使用Messenger"
date:   2017-04-28 1:05:00
catalog:  true
tags:

   - Messenger
   
       
   
---

## Messenger定义

Messenger,信使，可使用它进行进程间的通信，而Messenger对Service的请求采用队列的方式，因此它不支持多线程通信。

 

看看官方文档对于Messenger的解释：

> Reference to a Handler, which others can use to send messages to it. This allows for the implementation of message-based communication across processes, by creating a Messenger pointing to a Handler in one process,and handing that Messenger to another process.

客户端和服务端可相互持有对方的Messenger来进行通信，下面我们来看看具体的实现。

在eclipse下创建两个android工程，分别为客户端和服务端

## 客户端

创建客户端的Messenger，使用Messenger的构造方法指向一个handler实例，此handler用于处理服务端发过来的消息。

而客户端通过onServiceConnected获得服务端的Messenger，使用此Messenger给服务端发送消息，客户端的Messenger通过Message的replyTo传递给服务端。

```
   public class MainActivity extends Activity {

    private static final String TAG = "--DEBUG--";

    // 用于启动service的ACTION
    private static final String START_SERVER_ACTION = "com.young.server.START_SERVICE";
    private static final int WHAT_ON_TO_SERVICE = 1;
    private static final int WHAT_ON_TO_CLIENT = 2;

    private Button mBindBtn;
    private boolean isBindService = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mBindBtn = (Button) findViewById(R.id.bind_service);
        mBindBtn.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                bindService(new Intent(START_SERVER_ACTION), conn, Context.BIND_AUTO_CREATE);
            }
        });
    }

    // client端Handler，用于处理server端发来的消息
    private Handler mClientHandler = new Handler(new Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            switch (msg.what) {
            case WHAT_ON_TO_CLIENT:
                Log.v(TAG, "客户端收到服务端发来的消息！");
                break;

            default:
                break;
            }
            return false;
        }
    });

    // client端Messenger
    private Messenger mClientMessenger = new Messenger(mClientHandler);

    private ServiceConnection conn = new ServiceConnection() {

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.v(TAG, "服务已断开");

            isBindService = false;
            mClientMessenger = null;
        }

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.v(TAG, "服务已链接");

            isBindService = true;
            // 获得server端信使Messenger实例
            Messenger serverMessenger = new Messenger(service);
            // 向server端发送的消息
            Message toServerMessage = Message.obtain(null, WHAT_ON_TO_SERVICE);
            // 通过replyTo把client端的信使传递给service
            toServerMessage.replyTo = mClientMessenger;
            try {
                serverMessenger.send(toServerMessage);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    };

    protected void onStop() {
        if (isBindService)
            unbindService(conn);
        super.onStop();
    };
    }
```

## 服务端

服务端Service的实现，服务端接收到客户端的消息以后，通过Message的replyTo取出客户端的Messenger，使用此Messenger给客户端发送消息，这就实现了进程之间的双向通信。

服务端通过Messenger的getBinder方法将IBinder对象返给客户端，用于共享服务端的Messenger。

```
  public class RemoteService extends Service {
    private static final String TAG = "--DEBUG--";

    private static final int WHAT_ON_TO_SERVICE = 1;
    private static final int WHAT_ON_TO_CLIENT = 2;

    // server端handler，用来处理client发来的消息
    private Handler mServerHandler = new Handler(new Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            switch (msg.what) {
            case WHAT_ON_TO_SERVICE:
                Log.v(TAG, "收到客户端发来的消息");
                // server端获得client端的信使Messenger
                Messenger clientMessenger = msg.replyTo;
                Message toClientMsg = Message.obtain(null, WHAT_ON_TO_CLIENT);
                try {
                    // 使用客户端Messenger向客户端发送消息
                    clientMessenger.send(toClientMsg);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
                break;

            default:
                break;
            }
            return false;
        }
    });

    // server端信使Messenger
    private Messenger mServerMessenger = new Messenger(mServerHandler);

    @Override
    public IBinder onBind(Intent intent) {
        return mServerMessenger.getBinder();
    }
```

再来看看服务端service的声明，因为要在其他进程中启动service，所以设置android:exported为true，此外还为service加入启动了权限。

```
<permission android:protectionLevel="normal" android:name="young.permission.START_SERVICE"></permission>
    
<service
    android:name="com.young.server.RemoteService"
    android:permission="young.permission.START_SERVICE"
    android:exported="true" >
    <intent-filter>
        <action android:name="com.young.server.START_SERVICE" />
    </intent-filter>
</service>
```
最后要在客户端添加相应的启动Service权限。

```
<uses-permission android:name="young.permission.START_SERVICE" />

```

程序运行后的结果，可以看到客户端和服务端都收到了对方发来的消息。

```
11-12 12:58:37.197: V/--DEBUG--(21322): 服务已链接
11-12 12:58:37.197: V/--DEBUG--(21268): 收到客户端发来的消息
11-12 12:58:37.197: V/--DEBUG--(21322): 客户端收到服务端发来的消息！
```