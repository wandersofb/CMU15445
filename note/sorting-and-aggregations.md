# Sorting & Aggregations

本篇主要介绍在 DBMS 中是如何 sorting 和 aggregations。

### Sorting

DBMS 去 sort data 主要因为关系模型下存的数据是无序的，而 sorting 则被用在 ORDER BY, GROUP BY, JOIN, 和 DISTINCT operators。如果 data 是可以完全放在内存中的，则可以 quick-sort。但是 data 不能完全放在内存中，则需要 external sorting 合适的溢出到硬盘中，并且最好 sequential 而不是 random I/O。

**external merge sort** 是基础的算法用来排序不能完全放在内存的数据。它的思想是分治，就是把大数据分成独立的 ***runs*** 再分别 sort 它们。它们可以写回到硬盘，也可以读出来。这个算法包含两步：
**Phase #1 - Sorting：** 首先算法会 sort 可以放在内存的小 chunk，然后写回到硬盘中。
**Phase #2 - Merge：** 然后合并这些子文件到一个大的单独文件中。

其实和 merge sort 思想是一样的，只不过多了一个写回磁盘。

#### 2-Way External Merge Sort

**Pass #0**
* Read all B pages of the table into memory
* Sort pages into runs and write them back to disk

**Pass #1,2,3,…**
* Recursively merge pairs of runs into runs twice as long
* Uses three buffer pages (2 for input pages, 1 for output)

#### Double Buffering Optimization

通俗点说就是提前获取下一个要 sort 的 runs，不单单 sort 一个 runs 写回，可以同时 sort 两个 runs。

#### General External Merge Sort

**Pass #0**
* Use **B** buffer pages
* Produce **⌈N / B⌉** sorted runs of size B

**Pass #1,2,3,…**
* Merge **B-1** runs (i.e., K-way merge)

#### Using B+ Trees
如果说 B+ Trees 是 clustered index，那么数据被存储在正确的顺序，I/O 是 sequential 的，遍历 B+ Trees 是要由于 external merge sort。


###  Aggregations

Aggregation 就是再一次查询中将一组的 tuples 转化为一个标量。通常有两种方法来实现：1. Sorting 2. Hashing。

#### Sorting

通过排序去去重 DISTINCT。

#### Hashing

Hash table 就有天然去重操作，可以帮助我们去完成 DISTINCT，GROUP BY。
但是如果不能完全放到内存中：
**Phase #1 – Partition**
* Divide tuples into buckets based on hash key
* Write them out to disk when they get full

**Phase #2 – ReHash**
* Build in-memory hash table for each partition and compute the aggregation

