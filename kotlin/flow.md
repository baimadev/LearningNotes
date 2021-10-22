# kotlin flow操作符详解

## 1.流是冷的

当流调用collect时才会执行流中的任务。

## 2.超时取消

```kotlin
withTimeoutOrNull(250) { // 在 250 毫秒后超时,flow会被取消
    flow
}

```

## 3.流的构建

```kotlin
  ArrayList<Int>().asFlow()
  HashSet<String>().asFlow()
  (1..3).asFlow()

    flow{
      emit(1)
      emit(2)
      emit("3")
   }

  flowOf(1,2,3)
```

## 4.transform

类似map，可以发射多个数据。

```kotlin
(1..3).asFlow()
    .transform{ request ->
        emit("make request $request ")
        emit("make request again $request ")
    }
    .collect {
        println(it)
    }
```

collect将会收集到6个数据。

## 5.限长操作符 take

```kotlin
fun numbers(): Flow<Int> = flow {
    try {
        emit(1)
        emit(2)
        println("This line will not execute")
        emit(3)
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking<Unit> {
    numbers()
        .take(2) // 只获取前两个（数据源发射两条数据后将不再执行之后的代码，会执行finally）
        .collect { value -> println(value) }
}   
```

## 6.末端流操作符

### toList/toSet

```kotlin
val sum = (1..5).asFlow()
    .map { it * it } // 数字 1 至 5 的平方
    .toList()
```

### first

```kotlin
val first = (1..5).asFlow()
    .map { it * it } // 数字 1 至 5 的平方
    .first()

```

直接返回流的第一个值。

### single

等到只有一个数据的流发射的数据，如果是空流或者多个数据的流，则抛出错误。



### reduce/fold

```kotlin

fun main() = runBlocking<Unit> {
    val sum = (1..5).asFlow()
        .map { it * it } // 数字 1 至 5 的平方
        .reduce { a, b -> a + b } // 求和（末端操作符）
        // result = 55

    /* 拥有一个初始值   
    .fold(5, { a, b ->a + b })
    */
    // result = 60
}


```

## 7.buffer
上流发送数据很慢，下游处理的也很慢的时候。
可以使用buffer缓存发射项。

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // 假装我们异步等待了 100 毫秒
        emit(i) // 发射下一个值
    }
}

fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        simple()
            .buffer()
            .collect { value ->
            delay(300) // 假装我们花费 300 毫秒来处理它
            println(value)
        }
    }
    println("Collected in $time ms")
}

```

## 8.conflate

```kotlin
val time = measureTimeMillis {
    simple()
        .conflate() // 合并发射项，不对每个值进行处理
        .collect { value ->
            delay(300) // 假装我们花费 300 毫秒来处理它
            println(value)
        }
}   
println("Collected in $time ms")
```
下游正在处理数据时，上游不会发送数据，直接跳过中间数据，只发送最新的值。
应用场景：数据发送的很频繁，不必处理每一个数据。

## 9.collectLast

```kotlin
val time = measureTimeMillis {
    simple()
        .collectLatest { value -> // 取消并重新发射最后一个值
            println("Collecting $value")
            delay(300) // 假装我们花费 300 毫秒来处理它
            println("Done $value")
        }
}   
println("Collected in $time ms")
```

发送新值的时候会取消collectLast里代码的执行。  
应用场景：只需要处理最新值的时候。

## 10.组合多个流

### Zip

```kotlin

val nums = (1..3).asFlow() // 数字 1..3
val strs = flowOf("one", "two", "three") // 字符串
nums.zip(strs) { a, b -> "$a -> $b" } // 组合单个字符串
    .collect { println(it) } // 收集并打印
```
按顺序一一组合，最后数据的个数取最短的数据源。  
Zip会等待两个源的值，当两个源都有数据时才发送。

### Combine

```kotlin

val nums = (1..3).asFlow().onEach { delay(300) } // 发射数字 1..3，间隔 300 毫秒
val strs = flowOf("one", "two", "three").onEach { delay(400) } // 每 400 毫秒发射一次字符串
val startTime = System.currentTimeMillis() // 记录开始的时间
nums.combine(strs) { a, b -> "$a -> $b" } // 使用“combine”组合单个字符串
    .collect { value -> // 收集并打印
        println("$value at ${System.currentTimeMillis() - startTime} ms from start")
    }
```
Combine会组合多次，当其中任一数据源有新值得时候都会组合并发送一次。

## 11.展开流

### flatMapConcat

将上游的flow转换成另一个flow。

流还是按照顺序执行的。


### flatMapMerge

```kotlin
val startTime = System.currentTimeMillis() // 记录开始时间
(1..3).asFlow().onEach { delay(100) } // 每 100 毫秒发射一个数字
    .flatMapMerge {
        flow{
            emit("$it: First")
            delay(3000) // 等待 500 毫秒
            emit("$it: Second")
        }
    }
    .collect { value -> // 收集并打印
        println("$value at ${System.currentTimeMillis() - startTime} ms from start")
    }
```

上流会按顺序并发执行flatMapMerge中的转换flow。

### flatMapLast

上流会按顺序并发执行flatMapLast中的转换flow，当有新的数据源来临时会取消之前的数据执行。

## 12.流异常

### check

```kotlin
fun main() = runBlocking<Unit> {
    try {
        simple().collect { value ->
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    }
}
```

满足check条件则会抛出错误。

### catch

```kotlin
fun simple(): Flow<String> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit("$i") // 发射下一个值
        error("sd")
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> emit("Caught $e") } // 发射一个异常
        .collect { value -> println(value) }
}
```
error用于发送一个错误。  
catch用于捕获上流中的错误（仅上流），即不能捕获末端操作符中的异常。

我们可以将对数据的操作放在onEach中，然后下游调用catch。

```kotlin
simple()
    .onEach { value ->
        check(value <= 1) { "Collected $value" }                 
        println(value)
    }
    .catch { e -> println("Caught $e") }
    .collect()
```

### onCompletion

- 它在流完成收集时调用。  
- onCompletion 的主要优点是其 lambda 表达式的可空参数 Throwable 可以用于确定流收集是正常完成还是有异常发生。  
- 当末端操作符（collect）中抛出了错误，onCompletion也能收到。


```kotlin

simple()
    .onCompletion {e->
        if (e!=null){
            println("失败")
        }
    }
    .catch { e -> emit("Caught $e") } // 发射一个异常
    .collect { value -> println(value) }
```

## 13.启动流

### launchIn

- 可指定流在哪一个协程中去完成。launchIn（Scope）将流与scope生命周期绑定，工作方式就像addEventListener 一样。
而且，这不需要相应的 removeEventListener。

- launchIn 也会返回一个Job，可以取消该流的收集。

### cancel()

```kotlin
fun foo(): Flow<Int> = flow {
    for (i in 1..5) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value ->
        if (value == 3) cancel()  
        println(value)
    }
}
```

可以取消流的发送。流取消发送时会抛出JobCancellationException。
