# lab1

终于开始做 cmu15445 了， 之前就一直想做，但碍于英语和 C++ 一直没下去手，这次逼自己一把，开始受苦之路。其实 lab1 不难，每个函数都给出了逻辑结构，按照指示走，大体的代码就搞定了，但也有几个 tricky 的细节没有指示到，导致 debug 了很久。

### 值得注意的 C++ 语法

* 当给指针赋值时 `*frame_id = _frame_order.back();` 这是给 frame\_id 这个指针的指向的地址改变值。 `frame_id = &_frame_order.back();` 这是更改 frame\_id 指针指向的地址。
* 锁 不建议使用 `mutex`。强烈建议使用 `std::lock_guard<std::mutex>`，省去许多并发引起的错误。
* `dynamic_cast` 用于类继承层次间的指针或引用转换用于子类，可以安全的向下转化。

### Hints

* LRU 没什么好说的，**unordered\_map + list** 就行了，其实这样实现代码也简单，逻辑也清楚，效率也好。
* 大部分坑在 BPM，注意 `UnpinPgImp` 不能写入磁盘，只改变 `page` 的脏位。这是我遇见最大的坑点，我想的是我既然要 unpin 一个 page 了，不就是放入 LRU 内嘛，如果脏了，直接写入磁盘即可。但其是达咩，写入磁盘的任务交给其他的函数，这里只更新脏位。
* 还有强烈建议使用 `std::lock_guard<std::mutex>`，说不定你用 `std::mutex`，哪里就出问题了。
* 优化点，当 `FlushAllPgsImp` 时，判断一下是不是脏位，可以优化很多时间。
* PBPM 的话，维护一个 index，来便于 newpage。
