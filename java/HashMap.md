## HashMap

数组和链表组合构成的数据结构。
数组中存放的是Entry节点，包含key，value，next。

### put
put操作时会根据key的hash计算出在桶中的位置。  
当计算出多个key的hash值相同时，即在同一个桶中，会使用链表去存放节点。java8之后是尾插入。  
当链表长度超过8会转为红黑树。效率为logn


### get
根据hash找到对应的桶，然后根据key的equals方法去找到对应的节点，返回value。

### 扩容
Capacity:当前容量  
LoadFactor：负载因子，默认0.75

当超过Capacity*LoadFactor时，会触发扩容，分为两步：

- 创建一个新的Entry数组，长度是原来的2倍。
- 遍历原数组，重新计算hash值放到新的数组中。

### hash算法

`hash = key.hashcode() ^ key.hashcode() >>> 16`  
`index = hash & (size - 1)`

- 为什么要右移16位？
这样算出来的hash值不会丢失高区特征。

- 使用异或运算的原因
 异或运算能更好的保留各部分的特征，如果采用&运算计算出来的值会向0靠拢，采用|运算计算出来的值会向1靠拢。

- **size**需要是2^n  
2^n-1的二进制各位都是1，更加散列。算出来的index < size

## HashTable

线程安全，直接对put、get方法上锁（synchronized），并发1，效率低。

### 与HashMap的不同之处

- HashMap的键值可以为null
- HashTable线程安全
- 迭代器不同
HashMap使用的是fail-fast，HashTable使用的是fail-safe

**fail-fast**：用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。
原理：
迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。
集合在被遍历期间如果内容发生变化，就会改变modCount的值。
每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。

**fail-safe**:可以在多线程下并发使用，并发修改。


## ConcurrentHashMap
采用了分段锁，线程安全的Hashmap。
也是数组+链表实现的，数组中存放的是Segment。

**Segment**：继承了ReentrantLock，用volatile修饰了他的HashEntry数组。

每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。  
就是说如果容量大小是16他的并发度就是16，可以同时允许16个线程操作16个Segment而且还是线程安全的。  

### put
1.尝试自旋获取锁；
2.如果重试的次数达到了 MAX_SCAN_RETRIES 则改为阻塞锁获取，保证能获取成功。

### get
get 逻辑比较简单，只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。  
get 方法是非常高效的，因为整个过程都不需要加锁。

### jdk1.8
抛弃了分段锁，采用CAS+synchronized，用volatile修饰next。

流程大致和HashMap相同。
如果桶中不存在key，则用CAS写入，失败则自旋保证成功。；
如果桶中存在这样的节点，给节点上锁，用synchronized的方式写入。


# LinkedHashMap

直接继承了HashMap，比HashMap多了head、tail、accessorder。

- head 记录头节点
- tail 记录尾结点
- accessorder 访问顺序，true:按访问顺序排序，false：按插入顺序排序

节点Entry<K,V>也是直接继承HashMap的Node，多了before和after的引用。  

- **next**记录的是桶中的Entry顺序。
- before和after用于维护双向链表，记录Entry的插入顺序。

## accessorder

当为true时，get方法会将当前结点从链表中移除，然后插入到尾部。
可以实现LRU缓存。
