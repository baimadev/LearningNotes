### 1.空安全检测
空安全特性让kotlin移除了实时出现的空指针异常的风险。区分空引用和非空引用也是可能的。

### 2.为什么kotlin和java是可交互的
因为它也编译成jvm字节码。把它直接编译成字节码有助于实现更快的编译时间且对于jvm而言和java无差异。

### 3.kotlin如何处理空指针异常？
处理空指针异常

```kotlin
val l: Int = if (b != null) b.length else -1
//可替换为
val l = b?.length ?: -1
```

### 4.Kotlin类的默认行为？
Kotlin中所有的类默认为final。  
Kotlin支持多继承，Open class比final class造成更多开销。

### 4.== === equals
java中：   
==用来比较基本数据类型的值；在比较复合数据类型的时候是比较2个对象的地址。

equals在Object中的初始行为是比较内存地址。像String之类的equals，是先比较内存地址（地址相同直接返回true），在比较值是否相等。

java唯一有点小区别的是int装箱问题, 在-128~127的Integer值并且以Integer x = value;的方式赋值的Integer值在进行\==和equals比较时，都会返回true，因为Java里面对处在在-128~127之间的Integer值，用的是原生数据类型int，会在内存里供重用，也就是说这之间的Integer值进行\==比较时只是进行int原生数据类型的数值比较，而超出-128~127的范围，进行==比较时是进行地址及数值比较。

kotlin中：

==用来比较2个变量的值,只比较值。

=== 用来比较2个对象的地址,如果用在基本数据类型中则和== 一样表示比较2个变量的值。

equals看具体实现。

### 5.什么是 const? 和val有什么区别？
Val是在运行时被设置， 加一个const 修饰符在val上将会变成编译时常量。const不能修饰var，不能用于局部变量

### 6.kotlin默认的可见修饰符
public（java是default，当前类和同一个包中的类能访问）

### 7.data class
会自动生成getter setter hash copy toString componentN（解构，给多个变量赋值）函数。

它是一个 final 类，并且没有无参的构造函数，所以在使用过程非常不方便，但是我们可以利用官方给出的插件来解决这些问题（noarg、allopen）

### 8.lazy & lateinit

lateinit:

- lateinit只能修饰变量var，不能修饰常量val
- lateinit不能对可空类型使用
- lateinit不能对java基本类型使用，例如：Double、Int、Long等
- 在调用lateinit修饰的变量时，如果变量还没有初始化，则会抛出未初始化异常，报错

lazy:

 - lazy只能对常量val使用，不能修饰变量var
 - lazy的加载时机为第一次调用常量的时候，且只会加载一次（毕竟是个常量，只能赋值一次）
 - 底层实现是单例模式，可选参数 线程安全、同步锁、非线程安全模式。

### 9.Object
底层实现是饿汉式单例（线程安全）

### 10.双重检查锁
为什么要两次if。  
第一次避免不必要的内存开销（加锁）。  
第二次，防止两个线程同时进入同步代码块，一个线程阻塞，另一个线程创建实例，如果不加if，当第一个线程退出后，第二个线程会再次创建实例。


### 11.Kotlin有没有static 关键字？ 如何创建static函数？
companion object，全局只有一个单例。它需要声明在类的内部，在类被装载时会被初始化。


### 12.JvmOverloads

在有默认参数值的方法中使用@JvmOverloads注解，则Kotlin就会暴露多个重载方法。

```java

@JvmOverloads
fun f(a: String, b: Int=0, c:String="abc"){
}

//相当于java
void f(String a)
void f(String a, int b)
void f(String a, int b, String c)
```
### 13.构造函数

满足两个规则：
1.次级构造函数必须调用主构造函数；
2.子类构造函数必须调用父类构造函数。

### 14.inline
- 编译器做的优化，实际上是把函数体复制到调用处  
- 未内联的情况下，整个执行过程中会产生两个方法栈帧，每一个方法栈帧都包括了 局部变量表、操作数栈、动态连接、方法返回地址和一些额外的附加信息 。  
- 使用内联的情况下，只需要一个方法栈帧，降低了方法调用的成本
- 普通方法没必要使用内联，当方法参数为lambda时，且调用方法时传入的lambda代码块中还含有外部变量,最好使用内联。


### noinline

一个高阶函数一旦被标记为内联，它的方法体和所有 Lambda 参数都会被内联。

但是由于我要传入的 block1  代码块巨长(或者其他原因)，我并不想将其内联，这时候就要使用 noinline 。

### crossinline

首先，普通的 lambda 是不允许直接使用 return 的。  

而inline可以让我们突破这个限制。

```java  

inline fun runCatch(block: () -> Unit) {
    try {
        print("before lambda")
        block()
        print("after lambda")
    } catch (e: Exception) {
        e.printStackTrace()
    }
}

fun run() {
    runCatch { return }
}

```
这种返回称为非局部返回。

如果想仅仅退出 Lambda 的运行，就可以这样写。

```java  

inline fun runCatch(block: () -> Unit) {
    try {
        print("before lambda")
        block()
        print("after lambda")
    } catch (e: Exception) {
        e.printStackTrace()
    }
}

fun run() {
    // 从 lambda 中返回
    runCatch { return@runCatch }
}

```
这样的场景叫做局部返回。

crossinline 可以阻止非局部返回，但并不能阻止局部返回。

### 协程
本质上还是通过线程去执行任务，是kotlin提供的语法糖，协程任务可以在线程中串行执行，可以挂起也可以主动恢复，不会阻塞线程。是编译器的小把戏，通过插入代码，通过任务调度运行在同一个线程中。

解决了回调嵌套的问题，可以用同步的方式写异步代码；切换线程也方便

runBlocking可以阻塞线程。

suspend修饰的方法会挂起所在的协程。

协程挂起后再恢复，不一定运行在原来的线程，通过协程调度器和后台线程池决定，但是调度器能够保证它的执行顺序。

协程作用域：管理协程的生命周期。

协程作用域内部的取消是默认是双向的。父协程< == >子协程。
