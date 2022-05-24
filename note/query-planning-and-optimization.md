# Query Planning & Optimization

SQL 是声明性的，它告诉 DBMS 它想要的，而不是告诉 DBMS 怎么得到答案。因此 DBMS 需要翻译 SQL 语句成为一个可执行的 query plan。但即便是同一个 SQL 语句也会有多个 query plan，不同的 query plan 有着不同的性能，DBMS 要从中找到最优的 query plan，这个工作就被称作 DBMS's optimizer。

通常有两种优化策略：
* **Heuristics/Rules:** 重写 query 移除无效的东西，
* **Cost-based Search:** 使用成本模型来评估对等的 query plan，挑选出最优的。

![](../.gitbook/assets/20220523230202.png)

### Rule-based Query Optimization

如果两个关系代数表达式的结果是相同的子集，那么称它们是相同的。基于此，DBMS 可以确认更好的 query plan 而不需要 cost model。这被称作 query rewriting。

Examples of query rewriting:
* **Predicate Push-down:** 在 join 之间执行谓词过滤。
* **Projections Push down:** 提前预测出需要的 tuples attributes，制造更少 tuples，减少中间结果。
* **Expression Simplification:** 利用布尔逻辑的传递特性重写谓词将表达式转换为更简单的形式。

### Cost-based Query Optimization

DBMS 会使用内部的 ***cost model*** 来预估执行损耗对于特定的 query plan。这可以提供一个预估值来决定最好的决策方案。

这个估值只要有四个方面派生出：
* **CPU:** 消耗
* **Disk:** 转换快的数量
* **Memory:** DRAM 的使用数量
* **Network:** 报文传输的数量

为了完成这些，DBMS 需要记录一些关于表，属性和索引的内部统计值（statistics）。

###  Statistics

对于一个关系模型 **R**，DBMS 会维护一些信息：
* $$N_R$$ : **R** 中 tuples 的数量。
* $$V(A,R)$$ : 属性 **A** 中不同的值的数量。

$$
SC(A,R) = {N_R \over {V(A,R)}}
$$

**selection cardinality**（选择基数） 是属性 **A** 下每个值的平均记录个数。

注意这里假设数据平均分配（***data uniformity***）。

#### Complex Predicates

谓词的选择性（selectivity）是满足的 tuples 的分数。

对于常数查询
$$
sel(A = constant) = {SC(P) \over V(A,R)}
$$

对于范围查询：
$$
sel(A >= a) = (A_{max} - {a / {(A_{max} - A_{min})}})
$$

对于否定查询：
$$
sel(notP) = 1 - sel(P)
$$

对于关联查询：
$$
sel(P1 \land P2) = sel(P1) * sel(P2)
$$

#### Statistics Storage

**Histograms（直方图）:** 我们假设值是平均分布的，但其实不是，而且维护一个完整的直方图是昂贵的。我们可以放值到桶里来减少直方图的大小。但这可能导致某些值是不准确的。为了抵消这个缺点，桶的宽度和它的大小一样。

**Sampling（取样）:** 现代 DBMS 也会随机选取并且维护 tuples 的子集来预估 selectivity。

###  Search Algorithm

基本的 cost-based 搜索算法对于一个查询优化器而言：
1. 把内部形式的查询带到典范形式。
2. 生成多个可选择的计划。
3. 对于每个计划生成消耗。
4. 挑选最少消耗的计划。

对于查询的每一个表挑选出最好的 access method (i.e., sequential scan, binary search, index scan)是很重要的。

简单的  heuristics 对于 OLTP 已经足够了。

对于多关系查询计划，随着连接表的增多，可选择的 plans 也会快速增加。对于 n-way join，排序 join 操作的不同方法的数量是 catalan number （$$4^n$$）。这太大了。因此我们需要一个方法要减少搜索的复杂性。例如 IBM's System R，它们仅仅考虑 left-deep join trees。

Left-deep joins 允许你将数据流水线，仅仅需要在内存中维护单个 join table。

