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

当有新的请求入队和当前请求完成后，且runningAsyncCalls有空位时，将任务加进去，并从readyAsyncCalls中删除，最后交到线程池中去执行。

## 线程池

```java
executorServiceOrNull = ThreadPoolExecutor(0, Int.MAX_VALUE, 60, TimeUnit.SECONDS,
        SynchronousQueue(), threadFactory("OkHttp Dispatcher", false))
  }
```
核心线程数0，非核心线无穷大，阻塞队列SynchronousQueue(当添加一个元素时，必须等待一个消费线程取出它，否则一直阻塞)，实际上这个线程池就是newCacheThreadPool，短时间内可创建无限多的线程。  

但OkHttp设置了默认的最大并发请求量 maxRequests = 64（runningAsyncCalls最大容量） 和单个host支持的最大并发量 maxRequestsPerHost = 5。

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

#### 3.BridgeInterceptor 

主要工作是为请求添加cookie、添加固定的header，比如Host、Content-Length、Content-Type、User-Agent等等，然后保存响应结果的cookie，如果响应使用gzip压缩过，则还需要进行解压。

#### 4.CacheInterceptor

缓存拦截器，如果命中缓存则不会发起网络请求。  
OKHttp默认只支持get请求的缓存,使用request URL作为缓存的key.  
实现原理是Http的Etag & If-None-Match：服务器生成Etag返回给客户端，客户端发起第二次请求时发送一个If-None-Match，而它的值就是Etag的值。然后，服务器会比对这个客服端发送过来的Etag是否与服务器的相同，如果相同，就将If-None-Match的值设为false，返回状态为304，客户端继续使用本地缓存，不解析服务器返回的数据；如果不相同，就将If-None-Match的值设为true，返回状态为200，客户端重新解析服务器返回的数据

#### 5.ConnectInterceptor

连接拦截器，内部会维护一个连接池，负责连接复用、创建连接（三次握手等等）、释放连接以及创建连接上的socket流。

#### 6.networkInterceptors（网络拦截器）  

用户自定义拦截器，通常用于监控网络层的数据传输。

#### 7.CallServerInterceptor

请求拦截器，在前置准备工作完成后，真正发起了网络请求。

### addInterceptor与addNetworkInterceptor的区别

首先，应用拦截器在RetryAndFollowUpInterceptor和CacheInterceptor之前，所以一旦发生错误重试或者网络重定向，网络拦截器可能执行多次，因为相当于进行了二次请求，但是应用拦截器永远只会触发一次。另外如果在CacheInterceptor中命中了缓存就不需要走网络请求了，因此会存在短路网络拦截器的情况。

最后，从使用场景看，应用拦截器因为只会调用一次，通常用于统计客户端的网络请求发起情况；而网络拦截器一次调用代表了一定会发起一次网络通信，因此通常可用于统计网络链路上传输的数据。


  