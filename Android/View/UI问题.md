## 为什么不能子线程更新UI？

子线程更新UI报错是因为ViewRootImpl中checkThread（）方法。

```java

    void checkThread() {
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }

```

其中mThread是ViewRootImpl创建时所在的线程，而其创建时机是WindowManager添加view的时候。所以通常情况下这个mThread就是主线程。

也可以实现在子线程更新UI：可以在子线程中调用WindowManager去addView；或者绕开checkThread方法。

## 获取View宽高的几种方法？

首先，在 view 测量过后就可以获得 view 的测量宽高，另外系统可能需要多次 measure 才能确定最终的测量宽高。
其次，由于 view 的测量过程和 Activity 的生命周期不是同步，所以无法保证在 onCreate，onStart， onResume 时已经测量完毕，如果没有测量完毕获取的宽高都是0。

能获取真实 view 宽高的方法：

- Activity/View 的 onWindowFocusChanged 方法: 方法含义是 View 已经初始完毕，宽高已经准备好了, 得到焦点和失去焦点会被多次调用
- view.post：将 Runnable 投递到消息队列的尾部，当执行到的时候，view 已经初始化好了
- ViewTreeObserver: 使用 ViewTreeObserver 众多回调可以完成这功能，比如 onGlobalLayoutListener，会调用多次
- view.addOnLayoutChangeListener

### 为什么view.post可以拿到宽高？

在Looper内循环取出消息的方式来处理消息，当一个消息处理完之后再取出另一个消息，由Handler处理。
我们的View的绘制加载和View.post()都是又主线Handler来，所以只有当View的绘制加载的消息完成之后，
才会处理我们View.post()发送来的消息，所以我们才够在View.post()内获取到View的宽高。

为什么view.post添加在绘制加载后面？
view.post将任务添加到一个队列中，当dispatchAttachToWindow被调用后，才会将队列中的任务post到主线程Handler中。

## LayoutInflater.inflate

内部是通过XmlPull解析标签，反射创建View。

View.inflate 和 LayoutInflater.inflate 的区别？

View.inflate实际上最终调用的还是LayoutInflater.inflate(@LayoutRes int resource, @nullable ViewGroup root)三个参数的方法，
这里如果传入的root如果不为空，那么解析出来的View会被添加到这个ViewGroup当中去。

## Fragment懒加载

重写Fragment的setUserVisibleHint()，判断Fragment可见的时候加载数据。


{
        "printKey": "6500313931326685185",
        "printerType": 1,
        "printerIp": "192.168.102.140",
        "printerPort": 9100,
        "pageSize": "80",
        "printTimes": 1,
        "printRows": [
               {
                "contentType": "BlankRow",
                "blankRow":{
                      "lineNumber":4
                }
            }
        ]
    }

## 滑动冲突

### 外部拦截

在父布局中拦截ACTION_MOVE。

```java  
override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {
    ev?.run {
        if (action == MotionEvent.ACTION_MOVE && 父容器需要点击事件){
            return true
        }
    }
    return super.onInterceptTouchEvent(ev)
}

```

如果我们拦截掉 ACTION_UP 的话，肯定会导致子元素的点击事件无法被处理，因为大家肯定都知道一个点击事件从 ACTION_DOWN 开始，从 ACTION_UP 结束，二者缺一不可。

### 内部拦截

内部拦截法的话，需要 requestDisallowInterceptTouchEvent() 方法的支持，请求是否不允许拦截事件，其接收一个 boolean 参数，true表示是否不允许拦截。  
requestDisallowInterceptTouchEvent实际上是决定调不调用父布局的onInterceptTouchEvent方法。



重写子元素的 dispatchTouchEvent() 方法，得到伪代码如下：

```java  
override fun dispatchTouchEvent(ev: MotionEvent?): Boolean {
    ev?.run {
        when(action){
            MotionEvent.ACTION_DOWN -> parent.requestDisallowInterceptTouchEvent(true)
            MotionEvent.ACTION_MOVE ->{
                if(满足需要让外部容器拦截事件){
                    parent.requestDisallowInterceptTouchEvent(false)
                }
            }
        }
    }
    return super.dispatchTouchEvent(ev)
}

```

然后再重写父容器的onInterceptTouchEvent()方法

```java  
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
   int action = ev.getAction();
   if(action == MotionEvent.ACTION_DOWN){
       return false;
   }else {
       return true;
   }
}


```

TODO：  
ViewDragHelper  
LayoutManager
