# Tree Indexes

~~噩梦 B+ Tree 要来了。~~&#x20;

本节主要介绍 Table Indexes 的最常见数据结构：B+ Tree。

表索引是表属性的子集的副本，这些属性被组织和 / 或排序以使用这些属性的子集进行有效访问。

### B-Tree Family

其实 B-Tree 是一个家族其中包括了 B Tree、B+ Tree、B\* Tree、B-Link Tree。 B Tree 把 key-value 存储在所有结点，而 B+ Tree 只在 leaf node 存储 key-value，inner node 指导搜索过程。

### B+ Tree

B+ Tree 是一种自平衡树，searches, sequential access, insertions, and deletions in O(log n)。 A B+ Tree is an M-way search tree with the following properties:

* It is perfectly balanced (i.e., every leaf node is at the same depth).
* Every node other than(除了) the root, is at least half-full $${M \over 2} -1 ≤ keys ≤ M - 1$$
* Every inner node（非叶子结点） with k keys has k+1 non-null children

#### B+ Tree Nodes

每一个 B+ Tree node 都是由 key/value 组成的，一般情况下都根据 key 进行排序。

* The keys are derived from the attributes(s) that the index is based on.
* The values will differ based on whether the node is classified as inner nodes or leaf nodes.

key 都没啥变化，主要是 value 会在 leaf node 上有不同。

**Leaf Node values**

在 leaf node 上的 value 有两种存储情况：

* Approach #1: Record Ids A pointer to the location of the tuple that the index entry corresponds to. 一个指针指向 tuple 的位置。
* Approach #2: Tuple Data The actual contents of the tuple is stored in the leaf node.Secondary indexes have to store the record id as their values. tuple 的实际内容。

#### B+ Insert

1. 找到正确的叶子节点 **L**。
2. 把 data entry 放到 **L** 并且排序。
3. 如果 **L** 有足够的空间，结束；否则分裂 **L** 成两个 **L** 和 **L2**，重新分配 entry，插入 **L2** 的 index entry 到 **L** 的父节点，如果父节点没有足够空间，递归分裂。

#### B+ delete

1. 从 root Node 开始，找到正确的 leaf node **L**。
2. 删除 entry。
3. 如果 **L** 仍处于半满状态，结束；否则首先尝试从 sibling （adjacent node with same parent as **L**），如果失败，则合并 **L** 和 sibling。
4. 如果合并成功，则可能需要递归的删除父节点的 entry。

#### Clustered Indexes

通俗的说，就是表按照主键排序而存储。

### B+ Tree Design Choices

#### Node Size

一般而言，越慢的硬盘越大的 node size。

#### Merge Threshold

由于 merge 的操作太大，偶尔延迟 merge 操作或者定时的重建 tree 会更好。

#### Variable Length Keys

* Approach #1: Pointers Store the keys as pointers to the tuple’s attribute.
* Approach #2: Variable Length Nodes
  * The size of each node in the index can vary.
  * Requires careful memory management.
* Approach #3: Padding Always pad the key to be max length of the key type.
* Approach #4: Key Map / Indirection Embed an array of pointers that map to the key + value list within the node.

#### Non-unique Indexes

* Approach #1: Duplicate Keys Use the same leaf node layout but store duplicate keys multiple times.
* Approach #2: Value Lists Store each key only once and maintain a linked list of unique values.

#### Intra-node Search

* Approach #1: Linear Scan node keys from beginning to end.
* Approach #2: Binary Jump to middle key, pivot left/right depending on comparison.
* Approach #3: Interpolation Approximate location of desired key based on known distribution of keys. 个人感觉二叉更好一点，无论是时间上还是代码上。

### Optimizations

#### Prefix Compression

在 leaf node 的 keys 可能会有相同的 prefix，为了节省空间可以提取相同的 prefix，时间换空间。这个的确能带来性能提升。

#### Suffix Truncation

在 inner node 中，如果一定长度的 prefix 已经可以指引搜索了，那可以把后缀斩断，只存储前面的 prefix。这个用的少一定。

#### Bulk Insert

建造 B+ Tree 最快的方式是先将 keys 排好序，然后自底而上的建造。

#### Pointer Swizzling

Nodes 使用 page id 来存储其它 nodes 的引用，DBMS 每次需要首先从 page table 中获取对应的内存地址，然后才能获取相应的 nodes 本身，如果 page 已经在 buffer pool 中，我们可以直接存储其它 page 在 buffer pool 中的内存地址作为引用，从而提高访问效率。感觉没啥必要的优化。

