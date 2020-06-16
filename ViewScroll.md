#View Scroll

[ScrollX&ScrollY](#1)  
[Drag](#2)  
[多触点Drag](#3)  
[Fling](#4)  


```kotlin
 val vc = ViewConfiguration.get(context)
 mTouchSlop = vc.scaledTouchSlop
 mMinFlingVelocity = vc.scaledMinimumFlingVelocity
 mMaxFlingVelocity = vc.scaledMaximumFlingVelocity
 val metric: DisplayMetrics = context.getResources().getDisplayMetrics()
 SCREEN_WIDTH = metric.widthPixels
 SCREEN_HEIGHT = metric.heightPixels
```
==要监听ACTION POINTER DOWN和ACTION POINTER UP时需要使用`MotionEvent.actionMasked`来获取事件类型.==


<h3 id="1"></h3>

##ScrollX&ScrollY
mScrollX和mScrollY移动的是view的内容，ViewGroup就是移动其中的View，ImageView移动其中的图片。它是通过移动外面的可视窗口来相对移动内容的。  
如果你将mScrollX和mScrollY的数值都增大10，然后调用invalidate()重新绘制界面的话，你会发现视图中的内容都向左上角移动啦！

<h3 id="2"></h3>
##Drag
 Drag是最为基本的手势：用户可以使用手指在屏幕上滑动，以拖动屏幕相应内容移动。实现Drag手势其实很简单，步骤如下：

1.在*ACTION_DOWN*事件发生时，调用getX和getY函数获得事件发生的x,y坐标值，并记录在mLastX和mLastY变量中。  

2.在*ACTION_MOVE*事件发生时，调用getX和getY函数获得事件发生的x,y坐标值,将其与mLastX和mLastY比较，如果二者差值大于一定限制（ScaledTouchSlop）,就执行scrollBy函数，进行滚动,最后更新mLastX和mLastY的值。      

3.在*ACTION_UP*和*ACTION_CANCEL*事件发生时，清空mLastX，mLastY。

```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int actionId = MotionEventCompat.getActionMasked(event);
        switch (actionId) {
            case MotionEvent.ACTION_DOWN:
                mLastX = event.getX();
                mLastY = event.getY();
                mIsBeingDragged = true;
                break;
            case MotionEvent.ACTION_MOVE:
                float curX = event.getX();
                float curY = event.getY();
                int deltaX = (int) (mLastX - curX);
                int deltaY = (int) (mLastY - curY);
                if (!mIsBeingDragged && (Math.abs(deltaX)> mTouchSlop ||
                                                        Math.abs(deltaY)> mTouchSlop)) {
                    mIsBeingDragged = true;
                    // 让第一次滑动的距离和之后的距离不至于差距太大
                    // 因为第一次必须>TouchSlop,之后则是直接滑动
                    if (deltaX > 0) {
                        deltaX -= mTouchSlop;
                    } else {
                        deltaX += mTouchSlop;
                    }
                    if (deltaY > 0) {
                        deltaY -= mTouchSlop;
                    } else {
                        deltaY += mTouchSlop;
                    }
                }
                // 当mIsBeingDragged为true时，就不用判断> touchSlopg啦，不然会导致滚动是一段一段的
                // 不是很连续
                if (mIsBeingDragged) {
                        scrollBy(deltaX, deltaY);
                        mLastX = curX;
                        mLastY = curY;
                }
                break;
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                mIsBeingDragged = false;
                mLastY = 0;
                mLastX = 0;
                break;
            default:
        }
        return mIsBeingDragged;
    }
```
<h3 id="3"></h3>
##多触点Drag
1.当ACTION_POINTER_DOWN事件发生时，我们要记录第二触摸点事件发生的x,y值为mSecondaryLastX和mSecondaryLastY，和第二触摸点pointer的id为mSecondaryPointerId。  

2.当ACTION_MOVE事件发生时，我们除了根据第一触摸点pointer的x，y值进行滚动外，也要更新mSecondayLastX和mSecondaryLastY。  

3.当ACTION_POINTER_UP事件发生时，我们要先判断是哪个触摸点手指被抬起来啦，如果是第一触摸点，那么我们就将坐标值和pointer的id都更换为第二触摸点的数据；如果是第二触摸点，就只要重置一下数据即可。  

```kotlin
override fun onTouchEvent(event: MotionEvent?): Boolean {
        event?.let {
            when(event.actionMasked){
                MotionEvent.ACTION_DOWN ->{
                    val activePointerIndex = event.actionIndex
                    activePointerId = event.findPointerIndex(activePointerIndex)
                    mLastY = event.y
                    if (getParent() != null) {
                        getParent().requestDisallowInterceptTouchEvent(true);
                    }
                    mIsBeingDragged = false
                }

                MotionEvent.ACTION_POINTER_DOWN -> {
                    val activePointerIndex = event.actionIndex
                    secondPointerId = event.findPointerIndex(activePointerIndex)
                    mSecondLastY = event.getY(activePointerIndex)
                }

                MotionEvent.ACTION_MOVE ->{
                    if(!mScroller.isFinished){
                        mScroller.abortAnimation()
                    }

                    if(secondPointerId!= INVALID_ID){
                        val index = event.findPointerIndex(secondPointerId)
                        mSecondLastY = event.getY(index)
                    }

                    var dy = mLastY - event.y

                    if(!mIsBeingDragged&&abs(dy)>mTouchSlop){
                        mIsBeingDragged = true

                        if(dy>0){
                            dy-=mTouchSlop
                        }else{
                            dy+=mTouchSlop
                        }
                    }

                    if(mIsBeingDragged){
                        scrollBy(0,(dy+0.5f).toInt())
                        mLastY = event.y
                    }
                    0
                }

                MotionEvent.ACTION_POINTER_UP ->{
                    val curIndex = event.actionIndex
                    val curId = event.findPointerIndex(curIndex)
                    if(curId == activePointerId){
                        mLastY = mSecondLastY
                        activePointerId = secondPointerId
                        secondPointerId = INVALID_ID
                        mSecondLastY = 0f
                    }else{
                        secondPointerId = INVALID_ID
                        mSecondLastY = 0f
                    }
                }

                MotionEvent.ACTION_UP ->{
                    mIsBeingDragged = false
                    mLastY = 0f
                    activePointerId = INVALID_ID
                }
                
                else -> {

                }
            }
        }
        return true
    }
```
<h3 id="4"></h3>
## Fling
- 我们首先使用VelocityTracker.obtain()这个方法获得其实例
- 然后每次处理触摸时间时，我们将触摸事件通过addMovement方法传递给它  
- 最后在处理ACTION_UP事件时，我们通过computeCurrentVelocity方法获得滑动速度;  
- 我们判断滑动速度是否大于一定数值(MinFlingSpeed),如果大于，那么我们调用Scroller的fling方法。然后调用invalidate()函数。  
- 我们需要重载computeScroll方法，在这个方法内，我们调用Scroller的computeScrollOffset()方法啦计算当前的偏移量，然后获得偏移量，并调用scrollTo函数,最后调用postInvalidate()函数。  
- 除了上述的操作外，我们需要在处理ACTION_DOWN事件时，对屏幕当前状态进行判断，如果屏幕现在正在滚动(用户刚进行了Fling手势)，我们需要停止屏幕滚动。  

```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        .....
        if (mVelocityTracker == null) {
            //检查速度测量器，如果为null，获得一个
            mVelocityTracker = VelocityTracker.obtain();
        }
        int action = MotionEventCompat.getActionMasked(event);
        int index = -1;
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                ......
                                if (!mScroller.isFinished()) { //fling
                    mScroller.abortAnimation();
                }
                .....
                break;
            case MotionEvent.ACTION_MOVE:
                ......
                break;
            case MotionEvent.ACTION_CANCEL:
                endDrag();
                break;
            case MotionEvent.ACTION_UP:
                if (mIsBeingDragged) {
                //当手指立刻屏幕时，获得速度，作为fling的初始速度     mVelocityTracker.computeCurrentVelocity(1000,mMaxFlingSpeed);
                    int initialVelocity = (int)mVelocityTracker.getYVelocity(mActivePointerId);
                    if (Math.abs(initialVelocity) > mMinFlingSpeed) {
                        // 由于坐标轴正方向问题，要加负号。
                        doFling(-initialVelocity);
                    }
                    endDrag();
                }
                break;
            default:
        }
        //每次onTouchEvent处理Event时，都将event交给时间
        //测量器
        if (mVelocityTracker != null) {
            mVelocityTracker.addMovement(event);
        }
        return true;
    }
    private void doFling(int speed) {
        if (mScroller == null) {
            return;
        }
        mScroller.fling(0,getScrollY(),0,speed,0,0,-5000,10000);
        invalidate();
    }
    @Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(),mScroller.getCurrY());
            postInvalidate();
        }
    }
```




