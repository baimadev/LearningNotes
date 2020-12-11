#Glide
***
`Glide.with(context).load(url).into(imageView)`   
&emsp; 如果传入的是activity或是Fragment，**则会向当前的视图中添加一个隐藏的Fragment**，用来感知宿主生命周期。宿主死亡关闭图片加载，防止内存泄漏  
***
##目录  
[缓存](#1)  
[Glide回调](#2)  
[图片变换](#3)  
[Generated API](#4)
<h3 id="1"></h3>
***
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
   
1.ALL:缓存原图(DATA)和处理图(RESOURCE)

2.NONE:什么都不缓存

3.DATA:只缓存原图(DATA)

4.RESOURCE:只缓存处理图(RESOURCE) —默认值

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
<h3 id="2"></h3>
##Glide回调
### SimpleTarget

```java
SimpleTarget<GlideDrawable> simpleTarget = new SimpleTarget<GlideDrawable>() {
    @Override
    public void onResourceReady(GlideDrawable resource, GlideAnimation glideAnimation) {
        imageView.setImageDrawable(resource);
    }
};

public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
         .load(url)
         .into(simpleTarget);
}
```
当然，SimpleTarget中的泛型并不一定只能是GlideDrawable，如果你能确定你正在加载的是一张静态图而不是GIF图的话，我们还能直接拿到这张图的Bitmap对象.`asBitmap()`
### ViewTarget

```java
public class MyLayout extends LinearLayout {

    private ViewTarget<MyLayout, GlideDrawable> viewTarget;

    public MyLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        viewTarget = new ViewTarget<MyLayout, GlideDrawable>(this) {
            @Override
            public void onResourceReady(GlideDrawable resource, GlideAnimation glideAnimation) {
                MyLayout myLayout = getView();
                myLayout.setImageAsBackground(resource);
            }
        };
    }

    public ViewTarget<MyLayout, GlideDrawable> getTarget() {
        return viewTarget;
    }

    public void setImageAsBackground(GlideDrawable resource) {
        setBackground(resource);
    }

}
```
在MyLayout的构造函数中，我们创建了一个ViewTarget的实例，并将Mylayout当前的实例this传了进去。ViewTarget中需要指定两个泛型，一个是View的类型，一个图片的类型（GlideDrawable或Bitmap）。然后在onResourceReady()方法中，我们就可以通过getView()方法获取到MyLayout的实例，并调用它的任意接口了。比如说这里我们调用了setImageAsBackground()方法来将加载出来的图片作为MyLayout布局的背景图。
### preload()

```java
Glide.with(this)
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.SOURCE)
     .preload();
```
需要注意的是，我们如果使用了preload()方法，最好要将diskCacheStrategy的缓存策略指定成DiskCacheStrategy.SOURCE。因为preload()方法默认是预加载的原始图片大小，而into()方法则默认会根据ImageView控件的大小来动态决定加载图片的大小。因此，如果不将diskCacheStrategy的缓存策略指定成DiskCacheStrategy.SOURCE的话，很容易会造成我们在预加载完成之后再使用into()方法加载图片，却仍然还是要从网络上去请求图片这种现象。
### submit()
当调用了submit(int width, int height)方法后会立即返回一个FutureTarget对象，然后Glide会在后台开始下载图片文件。接下来我们调用FutureTarget的get()方法就可以去获取下载好的图片文件了，如果此时图片还没有下载完，那么get()方法就会阻塞住，一直等到图片下载完成才会有值返回。

```java
FutureTarget<File> target = Glide.with(context)
                                  .load(url)
                                  .submit(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);

```
之后我们可以使用如下代码去加载这张图片，图片就会立即显示出来，而不用再去网络上请求了：

```java
public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
            .load(url)
            .diskCacheStrategy(DiskCacheStrategy.SOURCE)
            .into(imageView);
}
```
需要注意的是，这里必须将硬盘缓存策略指定成DiskCacheStrategy.SOURCE或者DiskCacheStrategy.ALL，否则Glide将无法使用我们刚才下载好的图片缓存文件。
### listener
监听图片加载状态

```java
public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
            .load(url)
            .listener(new RequestListener<String, GlideDrawable>() {
                @Override
                public boolean onException(Exception e, String model, Target<GlideDrawable> target,
                    boolean isFirstResource) {
                    return false;
                }

                @Override
                public boolean onResourceReady(GlideDrawable resource, String model,
                    Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
                    return false;
                }
            })
            .into(imageView);
}
```
<h3 id="3"></h3>
##图片变换
###override()
```java
Glide.with(this)
     .load(url)
     .override(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL)
     .into(imageView);
```
Glide已经内置了几种图片变换操作，我们可以直接拿来使用，比如CenterCrop、FitCenter、CircleCrop等。
###BitmapTransformation 
自定义一个类让它继承自BitmapTransformation ，然后重写transform()方法，并在这里去实现具体的图片变换逻辑就可以了。

```java
public class CircleCrop extends BitmapTransformation {

    public CircleCrop(Context context) {
        super(context);
    }

    public CircleCrop(BitmapPool bitmapPool) {
        super(bitmapPool);
    }

    @Override
    public String getId() {
        return "com.example.glidetest.CircleCrop";//当前类名
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        int diameter = Math.min(toTransform.getWidth(), toTransform.getHeight());

        final Bitmap toReuse = pool.get(outWidth, outHeight, Bitmap.Config.ARGB_8888);
        final Bitmap result;
        if (toReuse != null) {
            result = toReuse;
        } else {
            result = Bitmap.createBitmap(diameter, diameter, Bitmap.Config.ARGB_8888);
        }

        int dx = (toTransform.getWidth() - diameter) / 2;
        int dy = (toTransform.getHeight() - diameter) / 2;
        Canvas canvas = new Canvas(result);
        Paint paint = new Paint();
        BitmapShader shader = new BitmapShader(toTransform, BitmapShader.TileMode.CLAMP, 
                                            BitmapShader.TileMode.CLAMP);
        if (dx != 0 || dy != 0) {
            Matrix matrix = new Matrix();
            matrix.setTranslate(-dx, -dy);
            shader.setLocalMatrix(matrix);
        }
        paint.setShader(shader);
        paint.setAntiAlias(true);
        float radius = diameter / 2f;
        canvas.drawCircle(radius, radius, radius, paint);

        if (toReuse != null && !pool.put(toReuse)) {
            toReuse.recycle();
        }
        return result;
    }
}
```
使用方式：

```java 
Glide.with(this)
     .load(url)
     .transform(new CircleCrop(this))
     .into(imageView);
```
[图片变换库]( https://github.com/wasabeef/glide-transformations )  

<h3 id="4"></h3>
##Generated API
###自定义模块

原理：创建Glide实例时，通过反射去创建MyAppGlideModule，再调用applyOptions、registerComponents。

```java

@GlideModule
public class MyAppGlideModule extends AppGlideModule {

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {

    }

    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {

    }

}
```
###Generated API

```java

@GlideExtension
public class MyGlideExtension {

    private MyGlideExtension() {

    }

    @GlideOption
    public static void cacheSource(RequestOptions options) {
        options.diskCacheStrategy(DiskCacheStrategy.DATA);
    }

}
```
然后在Android Studio中点击菜单栏Build -> Rebuild Project,你会发现你已经可以使用这样的语句来加载图片了：

```java
GlideApp.with(this)
        .load(url)
        .cacheSource()
        .into(imageView);

```
