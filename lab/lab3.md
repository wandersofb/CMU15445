# lab3

lab3 要依据火山模型，实现众多的 executors，代码量还是挺多的，但其实思路不是很难，不过要读和用的 API 需要很多，前期准备会很痛苦，但是等熟悉了 `abstract_expression`，`schema`，`tuple`，`colum`，`value` 等类之后，就会对整体的结果有很深刻的理解，以及火山模型是怎么运转的。

本篇重点聊一下 lab 的坑点和 hints。

### SEQUENTIAL SCAN

* 普通的序列扫描，大致意思是给我们一个谓词和一个输出 schema，让我们扫描一遍指定的表，如果 tuple 符合谓词则向上发射。当然谓词有不存在的时候。

### INSERT

* 插入分为俩种。一种是 `raw insert`，意思是已经准备好 tuples，直接向表中插入这些 tuples 即可。另一种是 `select insert`，是插入 child executor 向上发射的 tuples。别忘了修改 indexs。

* 更新 indexs 的时候要注意不能直接 `insertEntry`，index 有自己的 schema 和 key atributes，注意转化一下。

### UPDATE

* 更新就一种，只需更新 child executor 向上发射的 tuples，同样别忘了删除先前 indexs，插入当前的 indexs。

### DELETE

* 删除和更新一样，标记删除，更新 indexs即可。

### NESTED LOOP JOIN

* 嵌套循环连接就要动动脑子了，它的大致意思是循环俩个 child executors 发射的 tuples，如果符合谓词则 join。
* 别忘了会存在重复键，意思是我们不能找到一个符合条件的 join tuple 就发射，而是记录 join tuples，等扫描一遍完整的表之后，再把结果发射。
* 不能用额外的 `table_heap` 来记录无论是 inner table 还是 outer table，这样不会通过 I/O 测试。

### HASH JOIN

* 哈希连接的意思是指先把 left table 插入到我们的 hash table 中， 再扫描一遍 right table，如果有相同的 key 则是我们要 join 的对象。
* 这里要自己实现一下 hash table，仿照 aggregation 的来即可。迭代器可以不用写。记住 `HashKey` 不能声明在 `hash_join_plan.h`，因为我们并不 zip 这个文件，而是要在 `hash_join_executor.h` 的开头自己 bustub 命名空间里声明，下面的 DISTINCT 也同样。 

### AGGREGATION

* 聚簇是指让我们实现那些 max，min，count，sum 等操作，当然其内部已经实现好了，哈希表也都实现好了。
* aggregate 有许多谓词- having_ - group_bys_ - aggregates_;
* group_bys_ 是分组谓词，因为要将整个表按照分组名词进行分组，所以它是 hash table 的 key，有着相同的 key 会被分到一起
* aggregates_ 是 select max(*) 中的 max 谓词，当然可能会有很多，它是 hash table 的 value。其中的 aggregate 计算是在
* hash table 内部进行的 having_ 是 等到所有的 aggregate 计算出来之后的谓词，而 where 是在尚未 aggregate 的谓词
* 大致的思想是我们将 child executor 发射的 tuples 插入到 hash table 中，然后扫描一遍 hash tale 看是否符合 having 谓词，然后按照 outputschema 输出即可。

### LIMIT

* 限制是限制输出的 tulpes 个数，记个数即可

### DISTINCT

* 去重也要实现一个 hash set，但是有了前面的经验这个就简单多了。


### Hints
* 注意头文件
* 注意 `abstract_expression` 和 `colum->GetExpr`，往往在计算 output tuples 是有用。
* 要循环整个表才能找到完整表的手，中途需要一个 vector 来记录结果。
* 注意右值引用，往往需要 std::move() 进行转化。
