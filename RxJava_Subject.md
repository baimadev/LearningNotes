
#Subject
既是 Observable 又是 Observer(Subscriber)。

[AsyncSubject](#1) 
<h3 id="1"></h3>

##AsyncSubject

Observer会接收AsyncSubject的onComplete()之前的最后一个数据。

```java
        AsyncSubject<String> subject = AsyncSubject.create();
        subject.onNext("asyncSubject1");
        subject.onNext("asyncSubject2");

        subject.subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                System.out.println("asyncSubject:"+s);
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(@NonNull Throwable throwable) throws Exception {
                System.out.println("asyncSubject onError");  //不输出（异常才会输出）
            }
        }, new Action() {
            @Override
            public void run() throws Exception {
                System.out.println("asyncSubject:complete");  //输出 asyncSubject onComplete
            }
        });

        subject.onNext("asyncSubject3");
        subject.onNext("asyncSubject4");
        subject.onComplete();
        
        输出结果：      
       asyncSubject:asyncSubject4  
	   asyncSubject:complete
        
```

##BehaviorSubject
Observer会接收到BehaviorSubject被订阅之前的最后一个数据，再接收订阅之后发射过来的数据。如果BehaviorSubject被订阅之前没有发送任何数据，则会发送一个默认数据。

```java
        BehaviorSubject<String> subject = BehaviorSubject.createDefault("behaviorSubject1");
        subject.onNext("behaviorSubject2");

        subject.subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                System.out.println("behaviorSubject:"+s);  //输出asyncSubject:asyncSubject3
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(@NonNull Throwable throwable) throws Exception {
                System.out.println("behaviorSubject onError");  //不输出（异常才会输出）
            }
        }, new Action() {
            @Override
            public void run() throws Exception {
                System.out.println("behaviorSubject:complete");  //输出 behaviorSubject onComplete
            }
        });

        subject.onNext("behaviorSubject3");
        subject.onNext("behaviorSubject4");
        
执行结果：
behaviorSubject:behaviorSubject2
behaviorSubject:behaviorSubject3
behaviorSubject:behaviorSubject4
        
```

##RePlaySubject
ReplaySubject会发射所有来自原始Observable的数据给观察者，无论它们是何时订阅的。

```java 
        ReplaySubject<String> subject = ReplaySubject.create();
        subject.onNext("replaySubject1");
        subject.onNext("replaySubject2");

        subject.subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                System.out.println("replaySubject:"+s);
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(@NonNull Throwable throwable) throws Exception {
                System.out.println("replaySubject onError");  //不输出（异常才会输出）
            }
        }, new Action() {
            @Override
            public void run() throws Exception {
                System.out.println("replaySubject:complete");  //输出 replaySubject onComplete
            }
        });

        subject.onNext("replaySubject3");
        subject.onNext("replaySubject4");
        
执行结果：  
replaySubject:replaySubject1
replaySubject:replaySubject2
replaySubject:replaySubject3
replaySubject:replaySubject4
```

稍微改一下代码，将create()改成createWithSize(1)只缓存订阅前最后发送的1条数据.

```java
        ReplaySubject<String> subject = ReplaySubject.createWithSize(1);
        subject.onNext("replaySubject1");
        subject.onNext("replaySubject2");

        subject.subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                System.out.println("replaySubject:"+s);
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(@NonNull Throwable throwable) throws Exception {
                System.out.println("replaySubject onError");  //不输出（异常才会输出）
            }
        }, new Action() {
            @Override
            public void run() throws Exception {
                System.out.println("replaySubject:complete");  //输出 replaySubject onComplete
            }
        });

        subject.onNext("replaySubject3");
        subject.onNext("replaySubject4");
        
执行结果：  
replaySubject:replaySubject2
replaySubject:replaySubject3
replaySubject:replaySubject4
```

##PublishSubject
Observer只接收PublishSubject被订阅之后发送的数据。

```java 
        PublishSubject<String> subject = PublishSubject.create();
        subject.onNext("publicSubject1");
        subject.onNext("publicSubject2");

        subject.subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                System.out.println("publicSubject:"+s);
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(@NonNull Throwable throwable) throws Exception {
                System.out.println("publicSubject onError");  //不输出（异常才会输出）
            }
        }, new Action() {
            @Override
            public void run() throws Exception {
                System.out.println("publicSubject:complete");  //输出 publicSubject onComplete
            }
        });

        subject.onNext("publicSubject3");
        subject.onNext("publicSubject4");
        subject.onComplete();
        
执行结果：
publicSubject:publicSubject3
publicSubject:publicSubject4
publicSubject:complete

```

