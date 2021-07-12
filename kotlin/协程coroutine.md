## CoroutineScope

Coroutine 在设计的时候，要求在一个范围（Scope）内执行，这样当这个 Scope 取消的时候，里面所有的子 Coroutine 也自动取消。

```kotlin
public interface CoroutineScope {
    // Scope 的 Context
    public val coroutineContext: CoroutineContext
}
```



## 通道
通道提供了一种在流中传输值的方法。

### 通道基础
一个 Channel 是一个和 BlockingQueue 非常相似的概念。其中一个不同是它代替了阻塞的 put 操作并提供了挂起的 send，还替代了阻塞的 take 操作并提供了挂起的 receive。

```kotlin
val channel = Channel<Int>()
launch {
    // 这里可能是消耗大量 CPU 运算的异步逻辑，我们将仅仅做 5 次整数的平方并发送
    for (x in 1..5) channel.send(x * x)
}
// 这里我们打印了 5 次被接收的整数：
repeat(5) { println(channel.receive()) }
println("Done!")

```

### 关闭与迭代通道

和队列不同，一个通道可以通过被关闭来表明没有更多的元素将会进入通道。 在接收者中可以定期的使用 for 循环来从通道中接收元素。

从概念上来说，一个 close 操作就像向通道发送了一个特殊的关闭指令。 这个迭代停止就说明关闭指令已经被接收了。所以这里保证所有先前发送出去的元素都在通道关闭前被接收到。

```kotlin
val channel = Channel<Int>()
launch {
    for (x in 1..5) channel.send(x * x)
    channel.close() // 我们结束发送
}
// 这里我们使用 `for` 循环来打印所有被接收到的元素（直到通道被关闭）
for (y in channel) println(y)
println("Done!")

```

### 构建通道生产者

```kotlin
fun CoroutineScope.produceSquares(): ReceiveChannel<Int> = produce {
    for (x in 1..5) send(x * x)
}

fun main() = runBlocking {
    val squares = produceSquares()
    squares.consumeEach { println(it) }
    println("Done!")
}

```

## 管道

管道是一种一个协程在流中开始生产可能无穷多个元素的模式.  
被消费过的元素不再分发。？

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++) // 在流中开始从 1 生产无穷多个整数
}

fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
    for (x in numbers) send(x * x)
}
fun main() = runBlocking {
    val numbers = produceNumbers() // 从 1 开始生成整数
    val squares = square(numbers) // 整数求平方
    repeat(5) {
        println(squares.receive()) // 输出前五个
    }
    println("Done!") // 至此已完成
    coroutineContext.cancelChildren() // 取消子协程
}
```

协程的函数被定义在了 CoroutineScope 的扩展上， 所以我们可以依靠结构化并发来确保没有常驻在我们的应用程序中的全局协程。

### 扇出

多个协程处理同一个管道。

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1 // 从 1 开始
    while (true) {
        send(x++) // 产生下一个数字
        delay(100) // 等待 0.1 秒
    }
}

fun CoroutineScope.launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    for (msg in channel) {
        println("Processor #$id received $msg")
    }
}

fun main() = runBlocking {
    val producer = produceNumbers()
    repeat(5) { launchProcessor(it, producer) }
    delay(950)
    producer.cancel() // 取消协程生产者从而将它们全部杀死
}
```
管道中的元素将会交给不同的协程处理。

取消生产者协程将关闭它的通道，从而最终终止处理器协程正在执行的此通道上的迭代。

### 扇入

多个协程可以发送到同一个通道。

```kotlin
suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
    while (true) {
        delay(time)
        channel.send(s)
    }
}

fun main() = runBlocking {
    val channel = Channel<String>()
    launch { sendString(channel, "foo", 200L) }
    launch { sendString(channel, "BAR!", 500L) }
    repeat(6) { // 接收前六个
        println(channel.receive())
    }
    coroutineContext.cancelChildren() // 取消所有子协程来让主协程结束
}
```

### 带缓冲的通道

到目前为止展示的通道都是没有缓冲区的。无缓冲的通道在发送者和接收者相遇时传输元素（也称“对接”）。如果发送先被调用，则它将被挂起直到接收被调用， 如果接收先被调用，它将被挂起直到发送被调用。

Channel() 工厂函数与 produce 建造器通过一个可选的参数 capacity 来指定 缓冲区大小 。缓冲允许发送者在被挂起前发送多个元素， 就像 BlockingQueue 有指定的容量一样，当缓冲区被占满的时候将会引起阻塞。

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>(4) // 启动带缓冲的通道
    val sender = launch { // 启动发送者协程
        repeat(10) {
            println("Sending $it") // 在每一个元素发送前打印它们
            channel.send(it) // 将在缓冲区被占满时挂起
        }
        println("被挂起不会执行")
    }
// 没有接收到东西……只是等待……
    delay(1000)
    sender.cancel() // 取消发送者协程
}

```

### 通道是公平的的

```kotlin

data class Ball(var hits: Int)

fun main() = runBlocking {
    val table = Channel<Ball>() // 一个共享的 table（桌子）
    launch { player("ping", table) }
    launch { player("pong", table) }
    table.send(Ball(0)) // 乒乓球
    delay(1000) // 延迟 1 秒钟
    coroutineContext.cancelChildren() // 游戏结束，取消它们
}

suspend fun player(name: String, table: Channel<Ball>) {
    for (ball in table) { // 在循环中接收球
        ball.hits++
        println("$name $ball")
        delay(300) // 等待一段时间
        table.send(ball) // 将球发送回去
    }
}
```
发送和接收操作是 公平的 并且尊重调用它们的多个协程。它们遵守先进先出原则.  

“乒”协程首先被启动，所以它首先接收到了球。甚至虽然“乒” 协程在将球发送会桌子以后立即开始接收，但是球还是被“乓” 协程接收了，因为它一直在等待着接收球:

ping Ball(hits=1)  
pong Ball(hits=2)  
ping Ball(hits=3)  
pong Ball(hits=4)  


## 异常

### 异常的传播

协程构建器有两种形式：自动传播异常（launch 与 actor）或向用户暴露异常（async 与 produce）。 当这些构建器用于创建一个根协程时，即该协程不是另一个协程的子协程， 前者这类构建器将异常视为未捕获异常，类似 Java 的 Thread.uncaughtExceptionHandler， 而后者则依赖用户来最终消费异常。


``` kotlin
fun main() = runBlocking {
    val job = GlobalScope.launch { // launch 根协程
        println("Throwing exception from launch")
        //未捕获异常
        throw IndexOutOfBoundsException() // 我们将在控制台打印 Thread.defaultUncaughtExceptionHandler
    }
    job.join()
    println("Joined failed job")
    val deferred = GlobalScope.async { // async 根协程
        println("Throwing exception from async")
        throw ArithmeticException() // 没有打印任何东西，依赖用户去调用等待
    }
    try {
        deferred.await()
        println("Unreached")
    } catch (e: ArithmeticException) {
        println("Caught ArithmeticException")
    }
}

```

### CoroutineExceptionHandler

**全局未捕获异常处理器**


你无法从 CoroutineExceptionHandler 的异常中恢复。当调用处理者的时候，协程已经完成并带有相应的异常。通常，该处理者用于记录异常，显示某种错误消息，终止和（或）重新启动应用程序。

CoroutineExceptionHandler 仅在未捕获的异常上调用 — 没有以其他任何方式处理的异常。 特别是，所有子协程委托它们的父协程处理它们的异常，然后它们也委托给其父协程，以此类推直到根协程， 因此永远不会使用在其上下文中设置的 CoroutineExceptionHandler。 除此之外，async 构建器始终会捕获所有异常并将其表示在结果 Deferred 对象中， 因此它的 CoroutineExceptionHandler 也无效。

```kotlin


fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception")
    }
    val job = GlobalScope.launch(handler) { // 根协程，运行在 GlobalScope 中
        throw AssertionError()
    }
    val deferred = GlobalScope.async(handler) { // 同样是根协程，但使用 async 代替了 launch
        throw ArithmeticException() // 没有打印任何东西，依赖用户去调用 deferred.await()
    }
    joinAll(job, deferred)
}
```

//结果ExceptionHandler只会处理job协程。


### 取消与异常

取消与异常紧密相关。协程内部使用 CancellationException 来进行取消，这个异常会被所有的处理者忽略，所以那些可以被 catch 代码块捕获的异常仅仅应该被用来作为额外调试信息的资源。 当一个协程使用 Job.cancel 取消的时候，它会被终止，但是它不会取消它的父协程。

```kotlin
fun main() = runBlocking {
    val job = launch {
        val child = launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                println("Child is cancelled")
            }
        }
        yield()
        println("Cancelling child")
        child.cancel()
        child.join()
        yield()
        println("Parent is not cancelled")
    }
    job.join()
}

```
如果一个协程遇到了 CancellationException 以外的异常，它将使用该异常取消它的父协程。 这个行为无法被覆盖，并且用于为结构化的并发提供稳定的协程层级结构。 CoroutineExceptionHandler 的实现并不是用于子协程。

当父协程的所有子协程都结束后，原始的异常才会被父协程处理， 见下面这个例子。

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    println("CoroutineExceptionHandler got $exception")
}
val job = GlobalScope.launch(handler) {
    launch { // 第一个子协程
        try {
            delay(Long.MAX_VALUE)
        } finally {
            withContext(NonCancellable) {
                println("Children are cancelled, but exception is not handled until all children terminate")
                delay(100)
                println("The first child finished its non cancellable block")
            }
        }
    }
    launch { // 第二个子协程
        delay(10)
        println("Second child throws an exception")
        throw ArithmeticException()
    }
}
job.join()
```

### 异常聚合

当协程的多个子协程因异常而失败时， 一般规则是“取第一个异常”，因此将处理第一个异常。 在第一个异常之后发生的所有其他异常都作为被抑制的异常绑定至第一个异常。

```kotlin
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception with suppressed ${exception.suppressed.contentToString()}")
    }
    val job = GlobalScope.launch(handler) {
        launch {
            try {
                delay(Long.MAX_VALUE) // 当另一个同级的协程因 IOException  失败时，它将被取消
            } finally {
                throw ArithmeticException() // 第二个异常
            }
        }
        launch {
            delay(100)
            throw IOException() // 首个异常
        }
        delay(Long.MAX_VALUE)
    }
    job.join()
}


```

CoroutineExceptionHandler got java.io.IOException with suppressed [java.lang.ArithmeticException]

### 监督

一个子协程出错，其他子协程不会受牵连，只是向下传递。

SupervisorJob 可以用于这些目的。 它类似于常规的 Job，唯一的不同是：SupervisorJob 的取消只会向下传播。

```kotlin
fun main() = runBlocking {
    val supervisor = SupervisorJob()
    with(CoroutineScope(coroutineContext + supervisor)) {
        // 启动第一个子作业——这个示例将会忽略它的异常（不要在实践中这么做！）
        val firstChild = launch(CoroutineExceptionHandler { _, _ ->  }) {
            println("The first child is failing")
            throw AssertionError("The first child is cancelled")
        }
        // 启动第二个子作业
        val secondChild = launch {
            firstChild.join()
            // 取消了第一个子作业且没有传播给第二个子作业
            println("The first child is cancelled: ${firstChild.isCancelled}, but the second one is still active")
            try {
                delay(Long.MAX_VALUE)
            } finally {
                // 但是取消了监督的传播
                println("The second child is cancelled because the supervisor was cancelled")
            }
        }
        // 等待直到第一个子作业失败且执行完成
        firstChild.join()
        println("Cancelling the supervisor")
        supervisor.cancel()
        secondChild.join()
    }
}

```

The first child is failing  
The first child is cancelled: true, but the second one is still active  
Cancelling the supervisor  
The second child is cancelled because the supervisor was cancelled  

### 监督作用域
对于作用域的并发，可以用 supervisorScope 来替代 coroutineScope 来实现相同的目的。它只会**单向的传播**并且**当作业自身执行失败的时候将所有子作业全部取消**。作业自身也会在所有的子作业执行结束前等待， 就像 coroutineScope 所做的那样。

```kotlin
fun main() = runBlocking {
    try {
        supervisorScope {
            val child = launch {
                try {
                    println("The child is sleeping")
                    delay(Long.MAX_VALUE)
                } finally {
                    println("The child is cancelled")
                }
            }
            // 使用 yield 来给我们的子作业一个机会来执行打印
            yield()
            println("Throwing an exception from the scope")
            throw AssertionError()
        }
    } catch(e: AssertionError) {
        println("Caught an assertion error")
    }
}
```

The child is sleeping  
Throwing an exception from the scope  
The child is cancelled  
Caught an assertion error  

### 监督协程中的异常

监督协程中的每一个子作业应该通过异常处理机制处理自身的异常。 这种差异来自于子作业的执行失败不会传播给它的父作业的事实。 这意味着在 supervisorScope 内部直接启动的协程确实使用了设置在它们作用域内的 CoroutineExceptionHandler，与父协程的方式相同。

```kotlin
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception")
    }
    supervisorScope {
        val child = launch(handler) {
            println("The child throws an exception")
            throw AssertionError()
        }
        println("The scope is completing")
    }
    println("The scope is completed")
}

```

The scope is completing  
The child throws an exception  
CoroutineExceptionHandler got java.lang.AssertionError  
The scope is completed  
