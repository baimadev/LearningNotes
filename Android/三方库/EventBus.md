## 原理
EventBus中维护了三个Map。   

- 1.map<eventType,订阅信息集合> 用于查找事件订阅者
- 2.map<订阅者,evenType集合> 用于取消注册
- 3.map<eventType,事件> 用于维护粘性事件

### Subscription.class (订阅信息)

```java  
final class Subscription {
	//订阅者对象,一般情况下多为activity/fragment等
    final Object subscriber;
	//订阅者方法对象,主要包括订阅的
    final SubscriberMethod subscriberMethod;
}
```

## 注册

注册的时候会把信息填入1、2两个map。  

而订阅信息是怎么获取的呢?  
1.runtime注解, 通过反射机制获取订阅者中被注解标注的方法,然后把类信息和方法信息存入map。

2.编译时注解, eventbus3 提供了额外的注解处理器,可以在编译时找到这些注解方法，类信息，然后用代码生成工具生成索文件，运行时去找到这些索引。

## 反注册

- 通过订阅者和map2，找到该订阅者的所有订阅事件类型
- 然后根据订阅事件类型，删除map1中相应的订阅者。

## post

- 获取到当前线程的事件队列
- 通过事件类型查找map1，找到所有订阅者集合
- 反射执行订阅者中的订阅方法

## 粘性事件

当注册订阅者时，会去查找map3，然后取出事件执行。
