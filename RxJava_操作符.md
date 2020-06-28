
#RxJava

[distinctUntilChanged](#1)  
 [firstOrError](#2)   
 [just and fromCallable](#3)  
 [scan](#4)  
 [WithLatesFrom](#5)  
 [filter](#6)  
 [blockingGet](#7)
 [andThen](#8)

<h3 id="1"></h3>

## distinctUntilChanged

   `public final Observable<T> distinctUntilChanged()`  
     去掉重复的，只保留一个。
   
   ![img](https://github.com/baimadev/LearningNotes/blob/master/img/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1e7f4b05-0a29-4e95-b78e-6c5819f5ad5c.png?raw=true)
   
 
   
   
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


