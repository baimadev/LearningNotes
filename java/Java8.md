
```gradle
	//app modlue android目录下
   compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
```
**目录**  

- [Optional](#1) 
	- [创建Optional](#11)
	- [常用操作](#12)

- [Stream](#2) 
	- [创建Stream](#21)
	- [Intermediate](#22)
	- [Terminal](#23)   
	- [Short-circuiting](#24)   

<h3 id="1"></h3>
# Optional
Java 8引入Optional类来防止空指针异常.Optional类实际上是个容器：它可以保存类型T的值，或者保存null。

<h3 id="11"></h3>
## 创建Optional
- empty:创建一个空的Optional
- of:为非null的值创建一个Optional
- ofNullable:创建一个可为null的Optional

<h3 id="12"></h3>
## 常用操作
### isPresent
用来判断Optinal是否有值。如果值存在返回true，否则返回false。

### filter
filter方法接受一个条件函数，对Optional进行过滤。如果有值并且满足断言条件返回包含该值的Optional，否则返回空Optional。

### map

### flatmap

### get
get方法将获取Optional中value的值，如果存在值，则返回该值，否则抛出NoSuchElementException。

### orElse
如果Optional实例有值则将其返回，否则返回orElse方法传入的参数。

```java
System.out.println(empty.orElse("There is no value present!"));
//输出：There is no value present!

System.out.println(name.orElse("There is some value!"));
//输出：Sanaulla

```
### orElseGet
orElseGet与orElse方法类似，区别在于得到的默认值。orElse方法将传入的字符串作为默认值，orElseGet方法可以接受Supplier接口(只有一个get方法的接口)的实现用来生成默认值。

```java
System.out.println(empty.orElseGet(() -> "Default Value"));
//输出：Default Value

System.out.println(name.orElseGet(() -> "Default Value"));
//输出：Sanaulla
```

### orElseThrow
在orElseThrow中我们可以传入一个lambda表达式或方法，如果值不存在来抛出异常。

```java
try {
    empty.orElseThrow(ValueAbsentException::new);
} catch (Throwable ex) {
  //输出: No value present in the Optional instance
  System.out.println(ex.getMessage());
}

```


<h3 id="2"></h3>
# Stream
如何使用：  

- 创建Stream:通过stream()方法，取得集合对象的数据集。  
- Intermediate:通过一系列中间（Intermediate）方法，对数据集进行过滤、检索等数据集的再次处理。如上例中，使用filter()方法来对数据集进行过滤。  
- Terminal通过最终（terminal）方法完成对数据集中元素的处理。如上例中，使用forEach()完成对过滤后元素的打印。

Stream操作分类

- **Intermediate：**map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 skip、 parallel、 sequential、 unordered

- **Terminal：**forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、iterator

- **Short-circuiting：**
anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit

<h3 id="21"></h3>
## 创建Stream
### Stream.of()

```java
Stream<Integer> integerStream = Stream.of(1, 2, 3);
Stream<String> stringStream = Stream.of("A");
Stream<String> stringStream = Stream.of(Iterable<? extends T> iterable);
```

### Stream.generator()
generator方法，返回一个无限长度的Stream,其元素由Supplier接口的提供。在Supplier是一个函数接口，只封装了一个get()方法，其用来返回任何泛型的值，该结果在不同的时间内，返回的可能相同也可能不相同，没有特殊的要求。

```java 
Stream<Double> generateB = Stream.generate(()-> java.lang.Math.random());
Stream<Double> generateC = Stream.generate(java.lang.Math::random);
```

### Stream.iterate()
iterate方法，其返回的也是一个无限长度的Stream，与generate方法不同的是，其是通过函数f迭代对给指定的元素种子而产生无限连续有序Stream，其中包含的元素可以认为是：seed，f(seed),f(f(seed))无限循环。

```java
Stream.iterate(1, item -> item + 1)
        .limit(10)
        .forEach(System.out::println); 
        // 打印结果：1，2，3，4，5，6，7，8，9，10
```

### empty()
empty方法返回一个空的顺序Stream，该Stream里面不包含元素项。

### Arrays.stream(array)
在Arrays类，封装了一些列的Stream方法，不仅针对于任何类型的元素采用了泛型，更对于基本类型作了相应的封装，以便提升Stream的处理效率。

```java
int ids[] = new int[]{1, 2, 3, 4};
Arrays.stream(ids)
        .forEach(System.out::println);

```

<h3 id="22"></h3>
## Intermediate

### concat
concat方法将两个Stream连接在一起，合成一个Stream。若两个输入的Stream都时排序的，则新Stream也是排序的。

### distinc
去重.

### filter
只包含满足条件的元素，将不满足条件的元素过滤掉。

### map
map方法将对于Stream中包含的元素使用给定的转换函数进行转换操作，新生成的Stream只包含转换生成的元素。为了提高处理效率，官方已封装好了，三种变形：mapToDouble，mapToInt，mapToLong。

### flatmap
### peek
peek方法生成一个包含原Stream的所有元素的新Stream，同时会提供一个消费函数（Consumer实例），新Stream每个元素被消费的时候都会执行给定的消费函数，并且消费函数优先执行.

```java
Stream.of(1, 2, 3, 4, 5)
        .peek(integer -> System.out.println("accept:" + integer))
        .forEach(System.out::println);
// 打印结果
// accept:1
//  1
//  accept:2
//  2
//  accept:3
//  3
//  accept:4
//  4
//  accept:5
//  5
```

### skip
skip方法将过滤掉原Stream中的前N个元素，返回剩下的元素所组成的新Stream。如果原Stream的元素个数大于N，将返回原Stream的后（原Stream长度-N）个元素所组成的新Stream；如果原Stream的元素个数小于或等于N，将返回一个空Stream。

<h3 id="23"></h3>
## Terminal

### count 
count方法将返回Stream中元素的个数。

### forEachOrdered
forEachOrdered方法与forEach类似，都是遍历Stream中的所有元素，不同的是，如果该Stream预先设定了顺序，会按照预先设定的顺序执行（Stream是无序的），默认为元素插入的顺序。

### max
max方法根据指定的Comparator，返回一个Optional，该Optional中的value值就是Stream中最大的元素。  
原理：Stream根据比较器Comparator，进行排序(升序或者是降序)，所谓的最大值就是从新进行排序的，max就是取重新排序后的最后一个值，而min取排序后的第一个值。

```java
Optional<Integer> max = Stream.of(1, 2, 3, 4, 5)
        .max((o1, o2) -> o2 - o1);
System.out.println("max:" + max.get());// 打印结果：max:1
```

### reduce
根据指定的计算模型将Stream中的值计算得到一个最终结果。
  
方式一：对Stream中的数据通过累加器accumulator迭代计算，最终得到一个Optional对象

```java
  Optional accResult = Stream.of(1, 2, 3, 4).reduce((acc, item) -> {
  			acc += item;
             return acc;
        });
 //执行结果：Optional[10]
```

方式二:提供一个跟Stream中数据同类型的初始值identity，通过累加器accumulator迭代计算Stream中的数据，得到一个跟Stream中数据相同类型的最终结果.

```java
   int accResult = Stream.of(1, 2, 3, 4)
                .reduce(100, (acc, item) -> {
                    acc += item;
                    return acc;
                });

 //执行结果：110
```
方式三：提供一个不同于Stream中数据类型的初始值，通过累加器规则迭代计算Stream中的数据，最终得到一个同初始值同类型的结果。


```java
 		ArrayList<Integer> newList = new ArrayList<>();
        ArrayList<Integer> accResult_ = Stream.of(2, 3, 4)
                .reduce(newList,
                        (acc, item) -> {
                            acc.add(item);
                            return acc;
                        }, (acc, item) -> null);

 //执行结果：将2、3、4添加到list中。
 //第三个参数是在使用parallelStream的reduce操作时，合并各个流结果的，本例中使用的是stream，所以第三个参数是不起作用的。
```


<h3 id="24"></h3>
## Short-circuiting

### allMatch
allMatch操作用于判断Stream中的元素是否全部满足指定条件。如果全部满足条件返回true，否则返回false。

```java
boolean allMatch = Stream.of(1, 2, 3, 4)
    .allMatch(integer -> integer > 0);
System.out.println("allMatch: " + allMatch); // 打印结果：allMatch: true
```

### anyMatch
anyMatch操作用于判断Stream中的是否有满足指定条件的元素。如果最少有一个满足条件返回true，否则返回false。

### noneMatch
noneMatch方法将判断Stream中的所有元素是否满足指定的条件，如果所有元素都不满足条件，返回true；否则，返回false.

### findFirst
findFirst操作用于获取含有Stream中的第一个元素的Optional，如果Stream为空，则返回一个空的Optional。若Stream并未排序，可能返回含有Stream中任意元素的Optional。(Stream不保证顺序)

### limit(截取) findAny(随机)

