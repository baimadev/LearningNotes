#LinkedHashMap
HashMap额外维护了一个双向链表，记录插入顺序。
## 使用
accessOrder设为false，按插入顺序排序，最先插入的在头部，最后插入的在尾部。

accessOrder设为true，按插入访问顺序排序，最先插入的在头部，最后插入的在尾部,使用get或put会将元素放到队尾。
## 源码
LinkedHashMap继承于HashMap，额外多了双向链表头节点属性header，和标志位accessOrder。

它重新定义了Entry键，额外多了befor、after来维护LinkedHashMap中的双向链表（记录插入顺序）；特别注意next是用来记录各个桶中的Entry的连接顺序。都作用于Entry，但互不影响。
