# Hash Tables

这是 andy 最喜欢的一节课，也是我最喜欢的，因为比较简单，哈哈。 这节主要介绍 DBMS 的 Access Methods。主要解决如何从让 DBMS 快速的从 page 中读取数据。

### Hash Tables

Hash Table 主要分为两部分：

* Hash Function：
  * How to map a large key space into a smaller domain
  * Trade-off between being fast vs. collision rate
* Hashing Scheme：
  * How to handle key collisions after hashing
  * Trade-off between allocating a large hash table vs. additional instructions to find/insert keys

#### Hash Function

这就不多言，里面的知识太深奥了，对于 DBMS 设计者而言使用 XXHash3 就行了。

#### Static Hash Scheme

**Liner Probe Hashing**

我们熟知的开放地址寻址法就是这个方法。具体的策略就不多言了。

* 当 hashing collision 时，我们指定一个方向，去找到一个空位，存放我们的 key-value。注意 liner probe hashing 必须同时 key-value，这样才能分辨，hashing collision。
* 当要删除时，有两个方法。
  * Tombstone：在删除的位置放个尸体，以便于当 hashing collision 时的查找。
  * Movement：这个方法不太行，我不是很看好。
* 当遇见相同 key 时， 两个办法。
  * Separate Linked List：Store values in separate storage area for each key.
  * Redundant Keys：Store duplicate keys entries together in the hash table.

**Robin Hood Hashing & Cuckoo Hashing**

这俩就不多说了，用的也少。

#### dynamic Hash Scheme

与静态哈希策略需要预测数据量大小的情况不同，动态哈希策略，自己去扩容。

**Chained Hashing**

这是我们熟知的开链法，不过这里不单单只是一个结点，而是开辟一个桶。不过容易退化为链表，而且并发性能不好。

**Extendible Hashing**

这个 lab2 里实现了，具体可以看看 lab2's blog。

**Linear Hashing**

维护一个指针指向我们要拆分的 bucket，每当 bucket 溢出时，只拆分指针指向的 bucket。这个用的挺多的，建议多了解一下。
