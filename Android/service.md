---
title: service知识点
date: 2016-06-07 21:36:39
categories: Android
tags: service
---

# Service生命周期


Service生命周期可以从两种启动Service的模式开始讲起，分别是context.startService()和context.bindService()。

<!--more-->

## startService
startService的启动模式下的生命周期：当我们首次使用startService启动一个服务时，系统会实例化一个Service实例，依次调用其onCreate和onStartCommand方法，然后进入运行状态，此后，如果再使用startService启动服务时，不再创建新的服务对象，系统会自动找到刚才创建的Service实例，调用其onStart方法；如果我们想要停掉一个服务，可使用stopService方法，此时onDestroy方法会被调用，需要注意的是，不管前面使用了多个次startService，只需一次stopService，即可停掉服务。


## bindService
bindService启动模式下的生命周期：在这种模式下，当调用者首次使用bindService绑定一个服务时，系统会实例化一个Service实例，并一次调用其onCreate方法和onBind方法(onstart不执行)，然后调用者就可以和服务进行交互了，此后，如果再次使用bindService绑定服务，系统不会创建新的Service实例，也不会再调用onBind方法；如果我们需要解除与这个服务的绑定，可使用unbindService方法，此时onUnbind方法和onDestroy方法会被调用。


## 两种模式比较
两种模式有以下几点不同之处：startService模式下调用者与服务无必然联系，即使调用者结束了自己的生命周期，只要没有使用stopService方法停止这个服务，服务仍会运行；通常情况下，bindService模式下服务是与调用者生死与共的，在绑定结束之后，一旦调用者被销毁，服务也就立即终止，就像江湖上的一句话：不求同生，但愿同死。
在Android2.0时系统引进了onStartCommand方法取代onStart方法，为了兼容以前的程序，在onStartCommand方法中其实调用了onStart方法，不过我们最好是重写onStartCommand方法。
以上两种模式的流程如下图所示：

![](http://img.my.csdn.net/uploads/201110/25/0_13195414898sMm.gif)

## start+bind
如果一个Service又被启动又被绑定，则该Service将会一直在后台运行。并且不管如何调用，onCreate始终只会调用一次，对应startService调用多少次，Service的onStart便会调用多少次。调用unbindService将不会停止Service，而必须调用 stopService 或 Service的 stopSelf 来停止服务。

## 注意点：
1. 有bindService，一定要在某处unbindService （尽管 Activity 被 finish 的时候绑定会自　　　　　　动解除，并且Service会自动停止）
2. 使用 startService 启动服务之后，一定要使用 stopService停止服务，不管你是否使用bindService
3. 同时使用 startService 与 bindService 要注意到，Service 的终止，需要unbindService与stopService同时调用，才能终止 Service，不管 startService 与 bindService 的调用顺序，如果先调用 unbindService 此时服务不会自动终止，再调用 stopService 之后服务才会停止，如果先调用 stopService 此时服务也不会终止，而再调用 unbindService 或者 之前调用 bindService 的 Context 不存在了（如Activity 被 finish 的时候）之后服务才会自动停止；
4. 当在旋转手机屏幕的时候，当手机屏幕在“横”“竖”变换时，此时如果你的 Activity 如果会自动旋转的话，旋转其实是 Activity 的重新创建，因此旋转之前的使用 bindService 建立的连接便会断开（Context 不存在了），对应服务的生命周期与上述相同。
5. Service默认也是运行在主线程中的，因此如果处理cpu密集工作，还是建议在service开启一个新的子线程的。


## 代码

### startService：
```java
新建startService类extend Service 重写需要的方法。  
在AndroidManifest中注册
startService(new Intent(this, MyService.class));  
stopService(new Intent(this,MyService.class));
```

### bindService:
新建service类
```java
public class MyService extends Service {

    private DownloadBinder downloadBinder;
    /**
     * 新建一个内部类继承Binder，这个类提供一些activity需要的方法,不能是private
     */
    public class DownloadBinder extends Binder{
        public void startDownload(){
            //开始下载
        }
        public void stopDownload(){
            //停止下载
        }
    }

    @Override
    public void onCreate() {
        super.onCreate();
        downloadBinder = new DownloadBinder();//new出这个类的实例
    }
    /**
     * onBind 是 Service 的虚方法，因此我们不得不实现它。
     * 返回null表示客户端不能建立到此服务的连接
     */

    @Override
    public IBinder onBind(Intent intent) {
        //要想实现绑定操作，必须返回一个实现了IBinder接口类型的实例,有了它我们才能与服务进行交互。
        //这个实例会在activity中获取到，这样就可以调用这个类的方法了
        return downloadBinder;
    }
}
```
activity中：
```java
public class MainActivity extends AppCompatActivity  {

    private MyService.DownloadBinder downloadBinder;

    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (MyService.DownloadBinder) service;
            downloadBinder.startDownload();//调用方法
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            downloadBinder.stopDownload();
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //bindService绑定
        //有两个flag，BIND_DEBUG_UNBIND 与 BIND_AUTO_CREATE
        //前者用于调试（详细内容可以查看javadoc 上面描述的很清楚），后者默认使用
        bindService(new Intent(this,MyService.class),serviceConnection,BIND_AUTO_CREATE);

        //unbindService解绑，多次解绑会出问题
        unbindService(serviceConnection);
    }
}
```

# 前台服务和后台服务

- 前台服务：会在通知一栏显示 ONGOING 的 Notification，当服务被终止的时候，通知一栏的 Notification 也会消失，这样对于用户有一定的通知作用。常见的如音乐播放服务。  
- 后台服务：默认的服务即为后台服务，即不会在通知一栏显示 ONGOING 的 Notification。	当服务被终止的时候，用户是看不到效果的。某些不需要运行或终止提示的服务，如天气更新，日期同步，邮件同步等。

## 使后台服务变成前台服务
在service类的onCreate中：
```java
@Override
public void onCreate() {
    super.onCreate();
    Notification notification = new Notification(R.mipmap.ic_launcher,"notification!",System.currentTimeMillis());
    Intent notificationIntent = new Intent(this,MainActivity.class);
    PendingIntent pendingIntent = PendingIntent.getActivity(this,0,notificationIntent,0);
    notification.setLatestEventInfo(this, "Foreground Service",
            "Foreground Service Started.", contentIntent);
    startForeground(1,notification);
}
```

# 本地服务和远程服务
- 本地服务（Local）：服务依附在主进程上而不是独立的进程，这样在一定程度上节约了资源，另外Local服务因为是在同一进程因此不需要IPC，也不需要AIDL。相应bindService会方便很多。（就是上面的bindService）。进程被Kill后，服务便会终止。常见的应用如：HTC的音乐播放服务，天天动听音乐播放服务。
- 远程服务（Remote）：服务为独立的进程，对应进程名格式为所在包名加上你指定的android:process字符串。由于是独立的进程，因此在Activity所在进程被Kill的时候，该服务依然在运行，不受其他进程影响，有利于为多个进程提供服务具有较高的灵活性。	 该服务是独立的进程，会占用一定资源，并且使用AIDL进行IPC稍微麻烦一点。	 一些提供系统服务的Service，这种Service是常驻的。

# IntentService
继承的这种服务在运行结束后会自动停止
```java
public class MyIntentService extends IntentService {
   
    public MyIntentService(String name) {
        super(name);//无参构造，内部调用父类有参构造
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        //在子线程中执行，不必担心ANR，并且会自动自动停止，调用onDestory()方法
       
    }
}
//在activity中启动这个服务：
Intent intentService = new Intent(this,MyIntentService.class);
startService(intentService);


```

# Service 与 Thread 的区别
你可能会问，为什么要用 Service，而不用 Thread 呢，因为用 Thread 是很方便的，比起 Service 也方便多了，下面我详细的来解释一下

- Thread：Thread 是程序执行的最小单元，它是分配CPU的基本单位。可以用 Thread 来执行一些异步的操作。

- Service：Service 是android的一种机制，当它运行的时候如果是Local Service，那么对应的 Service 是运行在主进程的 main 线程上的。如：onCreate，onStart 这些函数在被系统调用的时候都是在主进程的 main 线程上运行的。如果是Remote Service，那么对应的 Service 则是运行在独立进程的 main 线程上。因此请不要把 Service 理解成线程，它跟线程半毛钱的关系都没有！
- 那么我们为什么要用 Service 呢？其实这跟 android 的系统机制有关，我们先拿 Thread 来说。Thread 的运行是独立于 Activity 的，也就是说当一个 Activity 被 finish 之后，如果你没有主动停止 Thread 或者 Thread 里的 run 方法没有执行完毕的话，Thread 也会一直执行。因此这里会出现一个问题：当 Activity 被 finish 之后，你不再持有该 Thread 的引用。另一方面，你没有办法在不同的 Activity 中对同一 Thread 进行控制。
- 举个例子：如果你的 Thread 需要不停地隔一段时间就要连接服务器做某种同步的话，该 Thread 需要在 Activity 没有start的时候也在运行。这个时候当你 start 一个 Activity 就没有办法在该 Activity 里面控制之前创建的 Thread。因此你便需要创建并启动一个 Service ，在 Service 里面创建、运行并控制该 Thread，这样便解决了该问题（因为在任何 Activity 中都可以控制同一 Service，只要知道类名，而系统也只会创建一个对应 Service 的实例）。
- 因此你可以把 Service 想象成一种消息服务，而你可以在任何有 Context 的地方调用 Context.startService、Context.stopService、Context.bindService，Context.unbindService，来控制它，你也可以在 Service 里注册 BroadcastReceiver，在其他地方通过发送 broadcast 来控制它，当然这些都是 Thread 做不到的。


# 实际使用选择
在什么情况下使用 startService 或 bindService 或 同时使用startService 和 bindService？
  

如果你只是想要启动一个后台服务长期进行某项任务那么使用 startService 便可以了。如果你想要与正在运行的 Service 取得联系，那么有两种方法，一种是使用 broadcast ，另外是使用 bindService ，前者的缺点是如果交流较为频繁，容易造成性能上的问题，并且 BroadcastReceiver 本身执行代码的时间是很短的（也许执行到一半，后面的代码便不会执行），而后者则没有这些问题，因此我们肯定选择使用 bindService（这个时候你便同时在使用 startService 和 bindService 了，这在 Activity 中更新 Service 的某些运行状态是相当有用的）。另外如果你的服务只是公开一个远程接口，供连接上的客服端（android 的 Service 是C/S架构）远程调用执行方法。这个时候你可以不让服务一开始就运行，而只用 bindService ，这样在第一次 bindService 的时候才会创建服务的实例运行它，这会节约很多系统资源，特别是如果你的服务是Remote Service，那么该效果会越明显（当然在 Service 创建的时候会花去一定时间，你应当注意到这点）。

# 在 AndroidManifest.xml 里 Service 元素的常见选项

android:name　　-------------　　服务类名

android:label　　--------------　　服务的名字，如果此项不设置，那么默认显示的服务名则为类名

android:icon　　--------------　　服务的图标

android:permission　　-------　　申明此服务的权限，这意味着只有提供了该权限的应用才能控制或连接此服务

android:process　　----------　　表示该服务是否运行在另外一个进程，如果设置了此项，那么将会在包名后面加上这段字符串表示另一进程的名字

android:enabled　　----------　　如果此项设置为 true，那么 Service 将会默认被系统启动，不设置默认此项为 false

android:exported　　---------　　表示该服务是否能够被其他应用程序所控制或连接，不设置默认此项为 false

# 参考链接
<http://www.cnblogs.com/newcj/archive/2011/05/30/2061370.html>