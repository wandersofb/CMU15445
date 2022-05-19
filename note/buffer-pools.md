# Buffer Pools

上次说到 DBMS 的硬盘管理层，这里我们要看一下 DBMS 如何在磁盘和内存之间搬运数据，分别从两个方面：空间和时间。

### Buffer Pool Manager

哈哈，没想到吧，DBMS 不仅有自己的 `DiskManager`，还有自己的 `BufferPoolManager`，这也就是 DBMS 如何摒弃 OS 的缓存策略，自己去缓存想要的页，当然这里不考虑 cache 层。

* DBMS 在启动的时候会申请一篇区域，这里就是 buffer pool 缓存的地方。它会把这片区域按页分割成 frame。
* 当 DBMS 想要某一个 page 时，从硬盘中一个精确的 page 副本会拷贝到 frame 中。
* 同时 DBMS 会维护一个 page table，这里面的 entry 会去追踪 frame 的 meta-data，包括 Dirty Flag 和 Pin Counter。
* 当 page table 的某个 entry 被 fetch 的时候，此时表明我的 DBMS 要读它或者写它，则不应该删除它。此时要修改 Pin Counter。
* 当要获取 page 的时候，可能会有并发问题，所以要用锁（latch），其实就是 C++ 的 mutex。

### Multiple Buffer Pools

唉，每当 DBMS 要获取一个页的时候不管它是否在 frame ，buffer pool 都要去获取锁来找页，那就对并发效果太差了。multiple buffer pools 就是多个 buffer pool 同时来缓存。它通过 page id 来区分此页要缓存到那一个 buffer pool。

### Buffer Replacement Policies

当 buffer pool 满了之后，我要替换其中的某个页。此处使用 LRU 替换政策。
