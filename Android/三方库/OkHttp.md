# OkHttp

## Dispatcher
- 记录同步任务、异步任务及等待执行的异步任务。
- 线程池管理异步任务。
- 发起/取消网络请求API：execute、enqueue、cancel。

使用三个双端队列，管理任务。

```java
# Dispatcher
//异步任务等待队列
private val readyAsyncCalls = ArrayDeque<AsyncCall>()
//异步任务队列
private val runningAsyncCalls = ArrayDeque<AsyncCall>()
//同步任务队列
private val runningSyncCalls = ArrayDeque<RealCall>()

```
Note：为什么不使用LinkedList？因为LinkedList内存不连续，查找起来慢。

放入运行时队列条件：当前请求数量小于64,单个host支持的最大并发量5.

放入时机：当有新的请求入队和当前请求完成后会调用finish，并从readyAsyncCalls中添加新的任务到运行时队列，最后交到线程池中去执行。

### 线程池

```java
executorServiceOrNull = ThreadPoolExecutor(0, Int.MAX_VALUE, 60, TimeUnit.SECONDS,
        SynchronousQueue(), threadFactory("OkHttp Dispatcher", false))
  }
```
核心线程数0，非核心线无穷大，阻塞队列SynchronousQueue(当添加一个元素时，必须等待一个消费线程取出它，否则一直阻塞)。  
实际上这个线程池就是newCacheThreadPool，短时间内可创建无限多的线程，保障最大吞吐量。  


## 拦截器

```kotlin
#RealCall
fun getResponseWithInterceptorChain(): Response {
    //创建拦截器数组
    val interceptors = mutableListOf<Interceptor>()
    //添加应用拦截器
    interceptors += client.interceptors
    //添加重试和重定向拦截器
    interceptors += RetryAndFollowUpInterceptor(client)
    //添加桥接拦截器
    interceptors += BridgeInterceptor(client.cookieJar)
    //添加缓存拦截器
    interceptors += CacheInterceptor(client.cache)
    //添加连接拦截器
    interceptors += ConnectInterceptor
    if (!forWebSocket) {
      //添加网络拦截器
      interceptors += client.networkInterceptors
    }
    //添加请求拦截器
    interceptors += CallServerInterceptor(forWebSocket)

    //创建责任链
    val chain = RealInterceptorChain(interceptors, transmitter, null, 0, originalRequest, this,
        client.connectTimeoutMillis, client.readTimeoutMillis, client.writeTimeoutMillis)
    ...
    try {
      //启动责任链
      val response = chain.proceed(originalRequest)
      ...
      return response
    } catch (e: IOException) {
      ...
    }
  }


```
一级一级的向下调用，最终会执行到CallServerInterceptor的intercept方法，此方法会将网络响应的结果封装成一个Response对象并return。之后沿着责任链一级一级的回溯，最终就回到getResponseWithInterceptorChain方法的返回。

### 拦截器分类

#### 1.应用拦截器   

拿到的是原始请求，可以添加一些自定义header、通用参数、参数加密、网关接入等等。  

#### 2.RetryAndFollowUpInterceptor  

处理错误重试和重定向。
内部是个死循环，抛出IO或Route异常时就会重新执行，执行重试操作。
获取到Response后会根据响应码处理30X重定向（拿到响应头中的Location 的url，然后重新创建Request，发起请求）。

#### 3.BridgeInterceptor

主要工作是为请求添加cookie、添加固定的header，比如Host、Connection、Content-Length、Content-Type、User-Agent等等，然后保存响应结果的cookie，如果响应使用gzip压缩过，则还需要进行解压。

#### 4.CacheInterceptor

缓存拦截器，如果命中缓存则不会发起网络请求。  
OKHttp默认只支持get请求的缓存,使用request URL作为缓存的key.  
实现原理是Http的Etag & If-None-Match：服务器生成Etag返回给客户端，客户端发起第二次请求时发送一个If-None-Match，而它的值就是Etag的值。然后，服务器会比对这个客服端发送过来的Etag是否与服务器的相同，如果相同，就将If-None-Match的值设为false，返回状态为304，客户端继续使用本地缓存，不解析服务器返回的数据；如果不相同，就将If-None-Match的值设为true，返回状态为200，客户端重新解析服务器返回的数据。

#### 5.ConnectInterceptor

连接拦截器，内部会维护一个连接池，负责连接复用、创建连接（三次握手等等）、释放连接以及创建连接上的socket流。
默认最多五个闲置连接，闲置连接最大存活时间5min。  
第一次向连接池中put socket时，便会开启一个清理任务，该任务会将存活时间超过5min的闲置连接remove，或者当前闲置连接数超过5个时，会remove闲置最久的那个连接。

#### 6.networkInterceptors（网络拦截器）  

用户自定义拦截器，通常用于监控网络层的数据传输。

#### 7.CallServerInterceptor

请求拦截器，在前置准备工作完成后，真正发起了网络请求。

### addInterceptor与addNetworkInterceptor的区别

首先，应用拦截器在RetryAndFollowUpInterceptor和CacheInterceptor之前，所以一旦发生错误重试或者网络重定向，网络拦截器可能执行多次，因为相当于进行了二次请求，但是应用拦截器永远只会触发一次。另外如果在CacheInterceptor中命中了缓存就不需要走网络请求了，因此会存在短路网络拦截器的情况。

最后，从使用场景看，应用拦截器因为只会调用一次，通常用于统计客户端的网络请求发起情况；而网络拦截器一次调用代表了一定会发起一次网络通信，因此通常可用于统计网络链路上传输的数据。


## HTTP2

### 二进制

HTTP/1 和 HTTP/2 的主要区别之一，就是 HTTP/2 是一个二进制、基于数据报的协议，而 HTTP/1 是完全基于文本的，基于文本的协议方便人类阅读，但是机器解析起来比较困难。

### 多路复用

HTTP/1 是一种同步的、独占的请求—响应协议，客户端发送 HTTP/1 消息，然后服务器返回 HTTP/1 响应，为了能更快地收发更多数据，HTTP/1 的解决办法就是打开多个连接，并且使用资源合并，以减少请求数，但是这种解决办法会引入其他问题和带来性能开销。

而 HTTP/2 允许在单个连接上同时执行多个请求，每个 HTTP 请求或响应使用不同的流，以流的方式多路复用。
