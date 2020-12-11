# Handler机制
Handler 主要是用于异步消息处理，类似于辅助类，他封装了消息的投递 处理的接口，通常用来处理耗时较长的操作。它有四个重要的部分：

- Handler：消息发送和处理
- Message：被发送和处理的消息
- MessageQueue：存放消息的消息队列
- Looper：循环的从MessageQueue中取消息给Handler处理


## Handler
Handler需要与线程中的Looper绑定，主线程中的Looper已经创建过且执行力loop（）方法；非主线程创建Looper，则需要调用Looper.prepare（）、looper.loop()。

## Looper
Looper 它的内部包含了一个消息队列,也就是Messagequeue 所有的handler发送的消息都会进入这个消息队列。  

Looper的loop方法 是一个死循环  它不断的从MessageQueue中来获取消息 如果有消息就处理消息 没有消息他就会进入阻塞状态（让cpu休眠，不会ANR）。

一个线程只有一个Looper（ThreadLocal），实现原理：是用当前ThreadLocal对象sThreadLocal作为key，sThreadLocal是所有线程共享的，一个线程只有一个ThreadLocalMap，而一个ThreadLocalMap在key = this时只能获取到一个value。

## 同步屏障
通常我们使用Handler想消息队列中添加的Message都是同步的，如果我们想要添加一个异步的Message，有以下两种方式：

- 1、Handler的构造方法有个async参数，设为true。
- 2、在创建Message对象时，直接调用Message的setAsynchronous()方法。

同步屏障就是为了确保异步消息的优先级，设置了屏障后，只能处理其后的异步消息，同步消息会被挡住，除非撤销屏障。

同步屏障是通过MessageQueue.postSyncBarrier方法开启的。  
同步屏障的移除是在MessageQueue.removeSyncBarrier()方法。

## 内存泄漏
如果用户在网络请求过程中关闭了Activity，正常情况下，Activity不再被使用，在onDestory()方法中执行GC检查时就应该被回收掉。  

但由于这时线程尚未执行完，而该线程持有Handler的引用，这个Handler又持有Activity的引用，就导致该Activity无法被回收，造成内存泄漏，直到网络请求结束。

解决：静态内部类，静态内部类不会被当前这个类所引用，再使用弱引用，gc触发时将handler中的activity回收。




