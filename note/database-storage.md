# Database Storage

由于 cmu15445 课上讲的太多了，不得不写点文字来记录一下学到的知识，记录的不全，会有很多跳跃的地方，仅作自己参考使用。

### 数据库存储层

传统的关系型数据库都是基于硬盘架构的，这一点解决了易失性，但也带来了其他问题，数据移动和速度。 数据库通常都会自己来管理硬盘，做 lab 的时候都知道有个 `DiskManager` 类，这就是数据库管理磁盘的类。 在此基础上，我们不得不问了。

* _为什么数据库要有磁盘管理器？_ 磁盘管理器是为了便于既获得磁盘的非易失性和容量又可以帮助数据搬运到内存中实现速度提升。
* _为什么不适用 OS 的磁盘管理模块？_
  * OS 不懂 DBMS ~~（你不懂我的心）~~
  * 在合适的时机把脏页写入到磁盘中
  * 特定的提前获取
  * OS 的缓存不适合 DBMS，DBMS 要有自己缓存管理池
  * 线程或进程管理

#### File Storage

* DBMS 的数据是作为文件存储在磁盘中的。
* DBMS 也分页管理。页大小不定，different systems do different things。
* heap file 其实就是链表，有两个，分别是空闲页和数据页。
* directory 会追踪数据页的位置和其容量。

#### Page Layout

这里是一般性概念，page 一般分为 header 和 data。

#### Data Layout

Strawman Idea：一个挨着一个存呗，header 记录一下大小。那么问题来了。

* 怎么删除？
* 如何处理变长的属性？

最常见的 layout scheme 是 slotted pages。

header 追踪已用的 slots 的数量以及起始位置和最后一个被使用的 slot 的 offset。

#### Tuple Layout

我们建表的时候会事先声明属性有多少，是 int 还是 char，这就我们的每个 tuple 所存的值。诶没想到吧，tuple page 不单单存储这些 data，它也有一个 header。当然这就比较 low level。

* Visibility info （concurrency control）
* Bit Map for NULL values

#### Log-Structure File Organization

log 结构很简单就是记录操作，如果要有查询，从当前位置向上查找。但是为了速度，一般会对根据操作 id 建立索引。当然要定期压缩索引文件。

#### Tuple Storage

DBMS 中的 catalog 会记录类型的信息，包括 INTERGE、VARCHAR等。有边长的，有等长的。 大部分 DBMS 不允许一个 tuple 的大小超过 page 大小。（不要存小电影在 DBMS 中）

#### System Catalog

Catalog 一般存储一些 meta-data。

* Tables, columns, indexes, views
* Users, permissions
* Internal statistics

#### OLTP && OLAP

应用场景描述： OLTP：只读或者小范围修改的数据 OLAP：查询大量数据并计算聚合操作

#### 数据存储模型

两种模型分别是行存储（N-ary Storage Model：NSM）和列存储（Decomposition Storage Model：DSM）。 NSM 适合 OLTP，DSM 适合 OLAP。

现在大部分都是 DSM。
