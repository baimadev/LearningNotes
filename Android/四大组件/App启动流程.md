## APP启动流程

- 1.点击App图标，Launcher进程向system_server的AMS发起startActivity的请求；
- 2.system_server进程收到请求后，向Zygote进程发起创建App进程的请求；
- 3.Zygote进程fork出新的子进程，即App进程；
- 4.App进程创建即初始化ActivityThread，然后再创建ApplicationThread，Looper，Handler对象，并开启主线程的消息循环Looper.loop();

**ApplicationThread**:是一个Binder对象，AMS和ActivityThread通信的桥梁。

**mH**：ApplicationThread 和ActivityThread之间的联系是通过Handler，ActivityThread的变量final H mH = new H(); 私有内部类H继承自Handler，是主线程的handler，处理一些消息事务。

- 5.然后向system_server的AMS发起attachApplication(applicationThread)请求,用于初始化Application和Activity。初始化时调用了两个关键函数：

(1)、bindApplication()方法通知主线程Handler创建Application对象、绑定context、执行onCreate()。

(2)、attachApplicationLocked()通过主线程Handler通知创建Activity对象，然后再调用onCreate()。

## init进程

Android手机开机Linux内核启动后，启动init进程。  
init进程会启动Zygote进程，启动ServiceManager。


## Zygote进程

孵化进程，启动后会首先孵化system_server进程，它的main函数主要负责：

- 创建Binder线程池
- 初始化Looper
- 创建了SystemServiceManager对象，它会启动Android中的各种服务，包括AMS、PMS、WMS
- 启动桌面进程
- 开启looper循环，SystemServer进程一直运行，保障其他应用程序正常运行

除了init进程fork出第一个进程Zygote进程，其他进程都是Zygote的子进程。  

Note：Zygote进程与AMS之间是通过Socket通信的。

## AMS

AMS是一个系统服务，四大组件都由它管理。

- startActivity 最终调用了AMS的 startActivity 系列方法，实现了Activity的启动；Activity的生命周期回调，也在AMS中完成；
- startService,bindService 最终调用到AMS的startService和bindService方法；
- 动态广播的注册和接收在 AMS 中完成（静态广播在 PMS 中完成）
- getContentResolver 最终从 AMS 的 getContentProvider 获取到ContentProvider

### ProcessRecord

其中维护了所有进程运行时的信息(ProcessRecord)，在attachApplication(mAppThread)时，将ProcessRecord.thread赋值成应用进程的IApplicationThread。这样一来，在AMS中就能通过该IApplicationThread实例发起向应用进程的调用。

### ActivityRecord

AMS通过ActivityRecord来维护Activity运行时的状态信息，需要将Activity绑定到AMS中的ActivityRecord能开始Activity的生命周期。  
系统进程ActivityRecord和应用进程的Activity之间存在映射关系（Token，通过token找到对应的Activity）。

## Instrumentation
 Instrumentation可以理解为应用进程的管家，ActivityThread要创建或暂停某个Activity时，都需要通过Instrumentation来进行具体的操作。

- AMS通过Binder通知ApplicationThrea
- ApplicationThread通过Handler通知ActivityThread
- ActivityThread通过Instrumentation做具体的操作
