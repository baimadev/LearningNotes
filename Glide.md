#Glide
***
`Glide.with(context).load(url).into(imageView)`   
&emsp; 如果传入的是activity或是Fragment，**则会向当前的视图中添加一个隐藏的Fragment**，用来感知宿主生命周期。宿主死亡关闭图片加载，防止内存泄漏  
##缓存
Glide处理的图片分两种类型：RESULT（处理后的图片）、SOURCE（原图），它默认只显示处理后的图片，对图片进行了压缩、旋转等操作。
###内存缓存
Glide默认是会在内存中缓存处理图(RESULT)的。  
`skipMemoryCache(true)`关闭内存缓存，默认是打开的。  
 Glide首先是从内存缓存中获取图片，从两个地方拿取。`LruCache`和`ActiveResource`
 
####`LruCache`
使用的是`LruCache`来管理，如果从这里拿到文件，会调用`remove`从`LruCache`中移除，再加入到`ActiveResource`

####`ActiveResource`
使用的是弱引用的HashMap管理，缓存对象是正在使用中的图片，图片资源类`EngineResou`有个acquired变量来记录引用次数，当资源被releas时，`acquired--`。当acquired小于0时,又将它写入到`LruCache`。  
使用`ActiveResource`的目的是分担`LruCache`的压力，提高效率。  

访问顺序：`ActiveResource` --> `LruCache`
###硬盘缓存
Glide磁盘缓存策略(`diskCacheStrategy`)分为四种,默认的是RESULT： 
   
1.ALL:缓存原图(SOURCE)和处理图(RESULT)

2.NONE:什么都不缓存

3.SOURCE:只缓存原图(SOURCE)

4.RESULT:只缓存处理图(RESULT) —默认值

内部是使用DiskLruCache来管理的，当都缓存时，首先会去找处理图(RESULT)，获取不到再去找原图(SOURCE)，原图和处理图的获取是通过他们的key来查找（处理图和原始图的key不同，原始图的key是由传入的url决定的，处理图的key由图片的宽高、旋转等10多个参数决定的）。

###token缓存无效

Glide加载图片，会使用它的url地址来组成缓存Key，当图片url带有token时，token作为一个验证身份的参数并不是一成不变的。而如果token变了，缓存Key也就跟着变了，导致Glide的缓存功能完全失效了。
####解决方法
创建一个`MyGlideUrl`继承自`GlideUrl`，重写这个`getCacheKey(）`。

```
Glide.with(this)
     .load(new MyGlideUrl(url))
     .into(imageView);
```