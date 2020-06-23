
#RxJava

[distinctUntilChanged](#1)  
 [firstOrError](#2)   
 [just and fromCallable](#3)  
 [scan](#4)  
 [WithLatesFrom](#5)  
 [filter](#6)

<h3 id="1"></h3>

## distinctUntilChanged

   `public final Observable<T> distinctUntilChanged()`  
     去掉重复的，只保留一个。
   
   ![img](img/企业微信截图_1e7f4b05-0a29-4e95-b78e-6c5819f5ad5c.png)
   
 
   
   
<h3 id="2"></h3>

## firstOrError

`public final Single<T> firstOrError()`  
只发送第一个或者NoSuchElementException。
![as](img/企业微信截图_8e0c024b-991e-483c-9631-6dc9417105d7.png)
   
   
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
![](img/企业微信截图_002b792b-06d7-483c-be7a-109ca39342c6.png)  
可以拿到上次流的数据。


<h3 id="5"></h3>
## WithLatesFrom


![](img/企业微信截图_12a675f3-3cd1-4496-a3d2-10f23acdb523.png)  
可以拿到两个流最近发射的数据。

<h3 id="6"></h3>
## filter
