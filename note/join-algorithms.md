# Join Algorithms

### Joins

---

优秀的数据库设计应该减少数据冗余。而我们的 Joins 应该要做到这一点。

本篇关注两个针对两个表的 **inner equijoin** 算法。 ***equijoin*** 算法是对于 keys 一样的表。


#### Operator Output

对于一个 **tuple** $$r \in R$$ 和另一个 **tuple** $$s \in S$$ 要匹配 join 属性，join 操作会串联 $$r$$ 和 $$s$$。
其内容取决于 processing model, storage model, and the query itself。
两种方法决定输出的内容的方式：
* **Data**： 直接把所有值拷贝到输出表中，这样不需要回头再看初始表。缺点是需要更多的内存来存储表，
* **Record Ids**：仅仅拷贝 join keys，其他的拷贝其相对应的 record ids。这是列式存储的理想方法，因为 DBMS 不需要任何 query 不需要的数据。这个称作 ***late materialization***。

#### Cost Analysis

如果数据量过大，势必要溢出到硬盘上，I/O 次数就是我们衡量 join 算法的标准。
| Variables used in this lecture:                    |
|----------------------------------------------------|
| **M** pages in table **R** (Outer Table), **m** tuples total |
| **N** pages in table **S** (Inner Table), **n** tuples total |

###  Nested Loop Join

#### Simple Nested Loop Join
最简单也是最蠢的做法，就是双重嵌套循环 join，很简单，两个表，两个 for 循环，如果满足我的 join 谓词，那么输出。一般而言 DBMS 会让 smaller 的 table 作为 outer for。
**Cost:** $$M + (m × N)$$

#### Block Nested Loop Join
这个算法的意思是在外层加上两次循环，分别对于 outer table block 和 inner table block。这样做以 block 为单位，减少了一些 I/O。
**Cost:** $$M + (M × N)$$

当然以上是以 3 个 buffers 为计算量，如果分配了 B 个 buffers，一个分配给 inner block，一个分配给 input block，剩余的都分配为 outer block。
**Cost:** $$M +(\lceil{M \over (B - 2)}\rceil × N)$$

#### Index Nested Loop Join

如果 inner table 有 index，速度还可以再提升，因为我们不用一次次的扫描整个磁盘，而是使用索引直接找到。

**Cost:** $$M + (m × C)$$

#### Conclusion

* Pick the smaller table as the outer table.
* Buffer as much of the outer table in memory as possible.
* Loop over the inner table or use an index.


### Sort-Merge Join

**Phase #1: Sort**
* Sort both tables on the join key(s).
* We can use the external merge sort algorithm that we talked about last class.

**Phase #2: Merge**
* Step through the two sorted tables with cursors and emit 
* matching tuples.
* May need to backtrack depending on the join type.

* Sort Cost for Table  **R**: $$2M × 1 + {\lceil \log_{B-1}{\lceil {M \over B} \rceil} \rceil}$$
* Sort Cost for Table **S**: $$2N × 1 + {\lceil \log_{B-1}{\lceil {N \over B} \rceil} \rceil}$$
* Merge Cost: $$(M + N)$$

#### Conclusion
* One or both tables are already sorted on join key.
* Output must be sorted on join key.
* The input relations may be sorted by either by an explicit sort operator, or by scanning the relation using an index on the join key.

### Hash Join

Hash Join 的 high level 思想是把 join attributes 通过 hash table 分成 small chuncks，这样就减少了各式各样的比较运算了。

**Phase #1: Build**
* Scan the outer relation and populate a hash table using the hash function h1 on the join attributes.

**Phase #2: Probe**
* Scan the inner relation and use h1 on each tuple to jump to a location in the hash table and find a matching tuple.

但是这样有一个缺点，就是不一定完全放进内存中。

### Grace Hash Join / Hybrid Hash Join

* **Build Phase:** Hash both tables on the join 
attribute into partitions.
* **Probe Phase:** Compares tuples in 
corresponding partitions for each table.


Partitioning Phase Cost: $$2 × (M + N)$$
Probe Phase Cost: $$(M + N)$$
**Total Cost:** $$3 × (M + N)$$

### Summary

| Algorithm |IO Cost | Example                          |
|-----------|--------|----------------------------------|
| Simple Nested Loop Join | $$M + (m * N)$$ | 1.3 hours     |
| Block Nested Loop Join | $$M + (M * N) 50$$ | seconds     |
| Index Nested Loop Join | $$M + (M * log N)$$ | 20 seconds |
| Sort-Merge Join | $$M + N + (sort cost)$$ | 0.59 seconds  |
| Hash Join | $$3(M + N)$$ | 0.45 seconds                   |

### Conclusion

Hash join 几乎是最好的选择胜过 sort-merge join。当然在一些情况 sort-merge 更优。所以 DBMS 会混用这两种算法。

