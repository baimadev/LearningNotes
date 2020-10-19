

[跳转目录](#1) 
<h3 id="1"></h3>

# 基础组件

## Animation

主要功能是保存动画的插值和状态，还可控制动画的播放方式（正向、反向、开始、结束）。常用的有Animation\<Double>、Animation\<Color>、Animation\<Size>，在动画的每一帧中，可以通过Animation对象的value属性获取动画的当前状态值。

## 动画通知

- addListener 每一帧都会被调用
- addStatusListener 获取动画的开始、结束、正向、反向

## Curve

用Curve（曲线）来描述动画过程。

## AnimationController
派生自Animation\<Double>用于控制动画，包含动画的启动forwar、停止stop、反向reverse等方法。会在动画的每一帧都生成一个新的值，默认从0到1.生成的数字区间可由**upperBound和lowerBound**控制。

## Ticker
当创建一个AnimationController时需要传入一个vsync参数，它接收一个*TickerProvider*，它的主要职责是创建Ticker。  
Flutter在每次启动时都会绑定一个SchedulerBinding，它可以为每次屏幕刷新添加回调，而Ticker就是通过它来添加屏幕刷新回调的，这样一来每次屏幕刷新都会调用TickerCallback。  
通常将==SingleTickerProviderStateMixin==添加到State的定义中，然后将它作为vsync的值。

## Tween
为AnimationController添加映射不同的取值范围或数据类型。

```dart 
final Tween colorTween = ColorTween(begin:Colors.transparent,end Colors.black54);
```

### Tween.animate
以下代码会在500ms内生成从0到255的整数值。

```dart
  final controller = AnimationController(duration: const Duration(milliseconds: 500),vsync: this);
  Animation<int> alpha = IntTween(begin: 0,end: 255).animate(controller); 
```

```dart
  final controller = AnimationController(duration: const Duration(milliseconds: 500),vsync: this);
  final Animation curve = CurvedAnimation(parent: controller,curve: Curves.easeOut);
  Animation<int> alpha = IntTween(begin: 0,end: 255).animate(curve);
```

# 动画基本结构

```dart

    controller = AnimationController(duration: Duration(seconds: 3,),vsync: this);
    animation = CurvedAnimation(parent: controller,curve: Curves.bounceIn);
    animation = Tween(begin: 0.0,end: 300.0).animate(animation);
    controller.forward();
    
    。。。
    
   return AnimatedBuilder(
      animation: animation,
      child: Image.asset("assets/test.png"),
      builder: (context,child){
        return Center(
          child: Container(
            height: animation.value,
            width: animation.value,
            child: child,
          ),
        );
      },
    );

```

# 动画状态监听

` controller.addStatusListener((status) {}`  
status有四种枚举值。 
 
- AnimationStatus.completed（动画在终点停止）；  
- AnimationStatus.dismissed（动画在起始点停止）；  
- AnimationStatus.forward(动画正在正向执行)；  
- AnimationStatus.reverse（动画正在反向执行）。

# 自定义路由切换动画

```dart
 Navigator.push(context, PageRouteBuilder(
                transitionDuration: Duration(milliseconds: 500),
                pageBuilder: (context,animation,secondAnimation){
                  return FadeTransition(
                    opacity: animation,
                    child: DragWidget(),
                  );
                }
            ));
```

或者使用继承PageRoute，实现自定义切换组件。

# Hero动画
从一个路由跳转到另一个路由时，指定某个widget过渡时的动画（位置、外观发生变化时）。

```dart
   child: Hero(
                tag: "avatar",
                child: ClipOval(
                  child: Image.asset("assets/test.png", width: 100, height: 100,),
                ),
              ),
```
前后两个路由用Hero包裹widget并且用相同的tag即可。


