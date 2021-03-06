---
title: ANR
date: 2017-1-03 22:18:22
categories: Android
tags: ANR
---

# ANR
ANR全称Application Not Responding，意思就是程序未响应。  
Android系统对于一些事件需要在一定的时间范围内完成，如果超过预定时间能未能得到有效响应或者响应时间过长，都会造成ANR。

<!--more-->

### 出现场景

- InputDispatching Timeout: 输入事件分发超时5s，包括按键和触摸事件。
- Service Timeout:比如前台服务在20s内未执行完成；后台服务为200s
- BroadcastQueue Timeout：比如前台广播在10s内未执行完成;后台广播为60s
- ContentProvider Timeout：内容提供者执行超时
- Activity: onCreate(), onResume(), onDestroy(),onKeyDown(), onClick()等，超时时间5s


### 定位分析
ANR产生时, 可以通过Logcat或者位于/data/anr/下traces.txt文件分析定位

#### Logcat
```
E/ActivityManager﹕ ANR in com.qihoo.browser (com.qihoo.browser/org.chromium.
chrome.browser.ChromeTabbedActivity)
Reason: keyDispatchingTimedOut
PID: 8762
//CPU前一分钟、五分钟、十五分钟的CPU平均负载, 
//CPU平均负载可以理解为一段时间内正在使用和等待使用CPU的活动进程的平均数量。
Load: 5.16 / 9.69 / 30.66      
//请注意ago，表示ANR发生之前的一段时间内的CPU使用率，并不是某一时刻的值
CPU usage from 34388ms to -1ms ago:
4.1% 32614/com.qihoo.browser: 2.5% user + 1.6% kernel / faults: 465 minor 1 major
...
+0% 3684/ksoftirqd/1: 0% user + 0% kernel
9.7% TOTAL: 4.8% user + 4.2% kernel + 0.3% iowait + 0.3% softirq
//这里是later，表示ANR发生之后
CPU usage from 1656ms to 2187ms later:
8.7% 743/system_server: 0% user + 8.7% kernel / faults: 4 minor
7% 943/InputDispatcher: 0% user + 7% kernel
1.7% 1199/Binder_6: 0% user + 1.7% kernel
5.2% 379/adbd: 0% user + 5.2% kernel / faults: 27 minor
3.5% 379/adbd: 0% user + 3.5% kernel
1.7% 1009/RX_Thread: 0% user + 1.7% kernel
1.7% 1768/mpdecision: 0% user + 1.7% kernel
1.7% 1784/mpdecision: 0% user + 1.7% kernel
1.2% 6883/kworker/u:37: 0% user + 1.2% kernel
1.3% 29165/kworker/0:2: 0% user + 1.3% kernel
2.3% TOTAL: 0.1% user + 0.3% kernel + 1.8% iowait
```
从Logcat中大致可以得到这些信息：

- 导致ANR的包名（com.qihoo.browser），类名（ChromeTabbedActivity），进程PID（8762）
- 原因：keyDispatchingTimedOut
- 系统中活跃进程的CPU占用率
比如这样一句话`99%TOTAL: 5.9% user + 4.1% kernel + 88% iowait`
表示CPU占用几乎满负荷了，其中绝大数是被iowait即I/O操作占用了。我们就可以大致得出是io操作导致的ANR。

#### traces.txt
首先对于没有root的手机，/data/anr/traces.txt通过普通的文件访问应该是没有权限的，不过通过adb还是可以获取的。  
`$adb pull data/anr/traces.txt ~/Desktop`

每次发生ANR， 这个文件都会被清空，写入新的内容. 如果想查看以前发生ANR的信息, 可以去查看DB文件。例如下面这段信息：

```
----- pid 30307 at 2015-05-30 14:51:14 -----
Cmd line: com.example.androidyue.bitmapdemo

JNI: CheckJNI is off; workarounds are off; pins=0; globals=272

DALVIK THREADS:
(mutexes: tll=0 tsl=0 tscl=0 ghl=0)

"main" prio=5 tid=1 TIMED_WAIT
  | group="main" sCount=1 dsCount=0 obj=0x416eaf18 self=0x416d8650
  | sysTid=30307 nice=0 sched=0/0 cgrp=apps handle=1074565528
  | state=S schedstat=( 0 0 0 ) utm=5 stm=4 core=3
  at java.lang.VMThread.sleep(Native Method)
  at java.lang.Thread.sleep(Thread.java:1044)
  at java.lang.Thread.sleep(Thread.java:1026)
  at com.example.androidyue.bitmapdemo.MainActivity$1.run(MainActivity.java:27)
  at android.app.Activity.runOnUiThread(Activity.java:4794)
  at com.example.androidyue.bitmapdemo.MainActivity.onResume(MainActivity.java:33)
  at android.app.Instrumentation.callActivityOnResume(Instrumentation.java:1282)
  at android.app.Activity.performResume(Activity.java:5405)
```

从上往下大致可以得出这些信息：

- 进程号（pid 30307）、ANR发生的时间点（2015-05-30 14:51:14）和进程名称（com.example.androidyue.bitmapdemo）
- （DALVIK THREADS）下面是各个线程的函数堆栈信息
- （"main" prio=5 tid=1 TIMED_WAIT）这段话依次表示：线程名、线程优先级、线程创建时的序号、①线程当前状态
- （group="main" sCount=1 dsCount=0 obj=0x416eaf18 self=0x416d8650）这段话依次表示：线程组名称、线程被挂起次数、线程被调试挂起的次数、线程的Java对象地址、线程的Native对象地址
- （sysTid=30307 nice=0 sched=0/0 cgrp=apps handle=1074565528）依次表示Linux系统中内核线程ID、线程调度优先级、线程调度策略和优先级、线程调度组、线程处理函数地址


#### DropBox
traces.txt只保留最后一次发生ANR时的信息, android 2.2开始增加了DropBox功能, 保留历史上发生的所有ANR的log.
“/data/system/dropbox”是DropBox指定的文件存放位置。日志保存的最长时间， 默认是3天.

### background ANR
有些时候程序实际上已经发生了ANR，只是没有进行对话框弹出而已。这种ANR就是background ANR，即后台程序的ANR。例如下面这段代码：
```java
public class NetworkReceiver extends BroadcastReceiver{
    private static final String LOGTAG = "NetworkReceiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.i(LOGTAG, "onReceive intent=" + intent);
        try {
            Thread.sleep(60000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Log.i(LOGTAG, "onReceive end");
    }
}
```
可以在Android开发者选项—>高级—>显示所有”应用程序无响应“勾选，即可对后台ANR也进行弹窗显示。

### ANR避免和检测

#### StrictMode
StrictMode意思为严格模式，是Android SDK提供的一个用来检测程序中违例情况的开发者工具。最常用的场景就是检测主线程中本地磁盘和网络读写等耗时的操作。

严格模式主要检测两大问题，一个是线程策略，即TreadPolicy，另一个是VM策略，即VmPolicy。

- 线程策略ThreadPolicy
> 
自定义的耗时调用 使用detectCustomSlowCalls()开启  
磁盘读取操作 使用detectDiskReads()开启  
磁盘写入操作 使用detectDiskWrites()开启  
网络操作 使用detectNetwork()开启  

- 虚拟机策略VmPolicy
> 
Activity泄露 使用detectActivityLeaks()开启  
未关闭的Closable对象泄露 使用detectLeakedClosableObjects()开启  
泄露的Sqlite对象 使用detectLeakedSqlLiteObjects()开启  
检测实例数量 使用setClassInstanceLimit()开启  

如何启用？
只需在在Application的onCreate方法中，添加如下代码：
```java
if (IS_DEBUG && Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD) {
    StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder().detectAll().penaltyLog().build());
  StrictMode.setVmPolicy(new VmPolicy.Builder().detectAll().penaltyLog().build());
}
```
上面的代码启用全部的ThreadPolicy和VmPolicy违例检测，你也可以按需开启某些策略
```java
if (DEVELOPER_MODE) {  
 StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()  
         .detectDiskReads()  
         .detectDiskWrites()  
         .detectNetwork()   
         .penaltyLog()  
         .build());  
 StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()  
         .detectLeakedSqlLiteObjects()  
         .detectLeakedClosableObjects()  
         .penaltyLog()  
         .penaltyDeath()  
         .build());  
} 
```

如何查看结果？  
通过logcat过滤StrictMode就能得到违例的具体stacktrace信息。

#### BlockCanary
可以使用[BlockCanary(AndroidPerformanceMonitor)](https://github.com/markzhai/AndroidPerformanceMonitor)来监控应用主线程的卡顿问题。

有一些ANR问题很难调查清楚，因为整个系统不稳定的因素很多，例如Linux Kernel本身的bug引起的内存碎片过多、硬件损坏等。这类比较底层的原因引起的ANR问题往往无从查起，并且这根本不是应用程序的问题。

### 参考
[Keeping Your App Responsive](https://developer.android.com/training/articles/perf-anr.html?hl=zh-cn)  
[Android 系统稳定性 - ANR（二）](http://rayleeya.iteye.com/blog/1955657)  
[理解Android ANR的触发情景](http://gityuan.com/2016/07/02/android-anr/)  
[Android性能调优利器StrictMode](http://droidyue.com/blog/2015/09/26/android-tuning-tool-strictmode/index.html)