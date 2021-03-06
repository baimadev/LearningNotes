# 前言
基于事件流的链式调用。  
任何操作都可以看成一条流。例如基本的网络请求操作：用户点击 -> 请求网络 -> 切换线程 -> 处理数据 -> view展示  
优点：避免了回调地狱、切换线程方便、流式处理逻辑更加清楚、有很多的强大的操作符使用  



#RxJava

 [distinct](#1)  
 [distinctUntilChanged](#12)  
 [firstOrError](#2)   
 [just and fromCallable](#3)  
 [scan](#4)  
 [WithLatesFrom](#5)  
 [filter](#6)  
 [blockingGet](#7)  
 [andThen](#8)  
 [switchMap](#9)  

## subscribeOn ObserveOn
Rxjava 提供了subscribeOn()方法来用于每个observable对象的操作符在哪个线程上运行。一般用来指定创建事件流的线程，也会影响其他操作符的执行线程。

Rxjava 提供了ObserveOn()方法来用于每Subscriber(Observer)对象的操作符在哪个线程上运行。

线程切换的时候subscribeOn()只被执行一次。如果出现多次，那么以第一次出现是用的那个线程为准。observeOn()改变调用它之后代码的线程。   


<h3 id="1"></h3>

## distinct

   `public final Observable<T> distinctUntilChanged()`  
     去掉重复的，只保留一个。
   
   ![img](https://github.com/baimadev/LearningNotes/blob/master/img/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1e7f4b05-0a29-4e95-b78e-6c5819f5ad5c.png?raw=true)
   
 
    
<h3 id="12"></h3>

## distinctUntilChanged
只有当我们观察的数据状态发生改变的时候才会释放数据，需要注意的是，它只会对前后两次释放的数据进行比较。  
![](https://github.com/baimadev/LearningNotes/blob/master/img/distinctUntilChanged.png?raw=true)
   
<h3 id="2"></h3>

## firstOrError

`public final Single<T> firstOrError()`  
只发送第一个或者NoSuchElementException。
![as](https://github.com/baimadev/LearningNotes/blob/master/img/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_8e0c024b-991e-483c-9631-6dc9417105d7.png?raw=true)
   
   
<h3 id="3"></h3>

## just and fromCallable

当just的参数为一个方法时，即使没有订阅也会先执行；fromCallable的参数是方法时，没有订阅便不会执行。

```kotlin

    val s1 = Single.just(just1())

    val s2 = Single.fromCallable { just2() }
    
    fun just1() {print("just1")}

    fun just2() {print("just2")}
	输出结果：
	just1
```

这样写just也不会执行。

```kotlin
 val s1 = Single.just{
        print("j1")
    }

```

<h3 id="4"></h3>

## scan

`  public final <R> Observable<R> scan(final R initialValue, BiFunction<R, ? super T, R> accumulator)` 
![](https://github.com/baimadev/LearningNotes/blob/master/img/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_002b792b-06d7-483c-be7a-109ca39342c6.png?raw=true)  
可以拿到上次流的数据。


<h3 id="5"></h3>

## WithLatesFrom

![](https://github.com/baimadev/LearningNotes/blob/master/img/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_12a675f3-3cd1-4496-a3d2-10f23acdb523.png?raw=true)  
可以拿到两个流最近发射的数据。

<h3 id="6"></h3>

## filter
![](https://github.com/baimadev/LearningNotes/blob/master/img/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_2c8c1a8d-30a5-4c16-948e-56cdf3995f7b.png?raw=true)  
为true则发送，false过滤掉。

<h3 id="7"></h3>

## blockingGet
以阻塞的方式等待获得success value或者an exception，不用再订阅。
![](https://github.com/baimadev/LearningNotes/blob/master/img/blockingGet.png?raw=true)

<h3 id="8"></h3>

## andThen
订阅此Completable，如果上游没有错误，则将输入的参数singleSource发送到下游；上游有错误，则将错误事件发送到下游，并且跳过Single的订阅。  
![](https://github.com/baimadev/LearningNotes/blob/master/img/andThen.png?raw=true)

<h3 id="9"></h3>

## switchMap
switch()和flatMap()很像，除了一点:当源Observable发射一个新的数据项时，如果旧数据项订阅还未完成，就取消旧订阅数据和停止监视那个数据项产生的Observable,开始监视新的数据项.  
![](https://github.com/baimadev/LearningNotes/blob/master/img/switchMap.png?raw=true)



