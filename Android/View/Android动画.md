[View动画](#1)   
[属性动画](#2)


# 动画
<h3 id="1"></h3>

## View动画
**只改变显示位置，实际位置未改变。**

- 透明度动画

```kotlin
	val aa = AlphaAnimation(0,1)
	aa.setDuration(1000)
	view.startAnimation(aa)
```
- 旋转动画

```kotlin
	val ra = RotateAnimation(0,360,100,100)//起始角度 旋转中心坐标
	ra.setDuration(1000)
	view.startAnimation(ra)
	//以自身为中心点
	val ra = RotateAnimation(0f,360f,RotateAnimation.RELATIVE_TO_SELF,0.5f,RotateAnimation.RELATIVE_TO_SELF,0.5f)
```
- 位移动画

```kotlin
	val ta = TranslateAnimation(0,200,0,300)
	ta.setDuration(1000)
	view.startAnimation(ta)
```

- 缩放动画

```kotlin
	val sa = ScaleAnimation(0,2,0,2) //x:0->2  y:0->2
	aa.setDuration(1000)
	view.startAnimation(sa)
	//以自身为中心旋转
	val sa = ScaleAnimation(0,2,0,2,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f)
```

- 动画集合

```kotlin
	  val set = AnimationSet(true)
      set.duration = 1000
      set.addAnimation(sa)
      set.addAnimation(ra)
      view.startAnimation(set)
```


- 监听回调

```kotlin
	     set.setAnimationListener(object :Animation.AnimationListener{
            override fun onAnimationRepeat(animation: Animation?) {
                TODO("Not yet implemented")
            }

            override fun onAnimationEnd(animation: Animation?) {
                TODO("Not yet implemented")
            }

            override fun onAnimationStart(animation: Animation?) {
                TODO("Not yet implemented")
            }
        })

```

<h3 id="2"></h3>

## 属性动画
### ObjectAnimator
ObjectAnimator参数包好一个对象和对象的属性名字，但这个属性必须有get和set函数，内部会通过java反射机制来修改属性的值。

```kotlin
        //第一个参数是需要操作的对象，第二个参数是属性名字
        val animator = ObjectAnimator.ofFloat(view,"translationX",100f,200f,100f)
        animator.duration = 1000
        animator.start()

```
常用属性动画的属性值：

- translationX、translationY
- rotation、rotationX(以X轴为中心3D旋转)、rotationY(以Y轴为中心3D旋转)
- scaleX、scaleY
- pivotX、pivoY(view对象的中心点)
- x、y
- alpha

#### 当属性没有get、set时

- 包装类
自定义个属性类，设置get和set方法（代理）。ObjectAnimator操作该包装类。


#### 监听

```kotlin
  animator.addListener(object:Animator.AnimatorListener{
            override fun onAnimationRepeat(animation: Animator?) {
                TODO("Not yet implemented")
            }

            override fun onAnimationEnd(animation: Animator?) {
                TODO("Not yet implemented")
            }

            override fun onAnimationCancel(animation: Animator?) {
                TODO("Not yet implemented")
            }

            override fun onAnimationStart(animation: Animator?) {
                TODO("Not yet implemented")
            }
        })

        //可选择想要的监听
        animator.addListener(object: AnimatorListenerAdapter() {

            override fun onAnimationEnd(animation: Animator?) {
                super.onAnimationEnd(animation)
            }
        })

```

### Interpolator

根据时间计算属性变化的百分比。

常用插值器：

- LinearInterpolator（@android:anim/linear_interpolator，线性插值器，匀速动画）
- AccelerateDecelerateInterpolator（@android:anim/accelerate_decelerate_interpolator，加速减速插值器，动画两头慢中间快）  
- AccelerateInterpolator（@android:anim/accelerate_interpolator，加速插值器，动画越来越快）
- DecelerateInterpolator（@android:anim/decelerate_interpolator，减速插值器，动画越来越慢）
- BounceInterpolator（@android:anim/bounce_interpolator，动画结束的时候弹起）

### ValueAnimator

估值器：根据插值器算出来的属性变化百分比来计算具体变化的值。

本身不提供动画效果，更像一个数值发生器，用来产生具有一定规律的数字。

```kotlin
 val animator = ValueAnimator.ofFloat(0f,100f)
        animator.setTarget(tu)
        animator.setDuration(1000)
        animator.addUpdateListener {
            val value = it.animatedValue as Float
            //TODO use the value
        }
```

还可以使用估值器（TypeEvaluator）来自定义运动轨迹。使用插值器改变变化速率。


```kotlin
 val animator =
            ValueAnimator.ofObject(TypeEvaluator<Float> { fraction, startValue, endValue ->
                (startValue!! + sin(fraction * endValue!!))//自定义运动轨迹
            }, 0f, 360f)
        animator.duration = 1000
        animator.addUpdateListener {
            val value = it.animatedFraction
            view.scrollBy(value.toInt(), value.toInt())
        }
        animator.interpolator = BounceInterpolator()//动画结束时回弹
        animator.start()

```


### AnimatorSet

```kotlin
  val set = AnimatorSet()
        set.duration = 1000
        set.playTogether(animator1,animator2,animator3)
        set.start()

```
可通过playToget、playSequentially、with、play、after、before来控制播放顺序。

### xml中使用属性动画

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration = "1000"
    android:propertyName="scaleX"
    android:valueFrom="1.0"
    android:valueTo="2.0"
    android:valueType="floatType"
    >


</objectAnimator>

 val anim = AnimatorInflater.loadAnimator(context,R.animator.scale)
        anim.setTarget(view)
        anim.start()

```
