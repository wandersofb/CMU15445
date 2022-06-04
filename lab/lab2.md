# lab2

lab2 要实现一个动态哈希中的可扩展哈希策略，说实话，难是难。但没有网上说的那么难，一共花了五天时间，三天的时间在找资料，理解可扩展哈希，一天的时间在写主体逻辑（`split`，`merge`），一天在 debug。不过在学习群里找到了测试代码，debug 不是 lab1 那样要上传到线上才能看错误信息，可以本体调试 + vscode 打断点，总之 debug 还是挺舒服的。最后并没怎么优化，我也不知道怎么回事，最后排名还挺高的，拿个第五。&#x20;

![](../.gitbook/assets/lab2-1.png)

本篇重点聊一下可扩展哈希策略的大体逻辑，lab 的 hints 和一些值得注意 C++ 语法。都是 high level，不对代码做分析。

### 可扩展哈希

可扩展哈希的大体逻辑是 `split` 和 `merge`，把这两个搞懂就行了。

#### split

`split` 是当我插入一个 key-value 的时候，发生了溢出。此时要 new 一个新的 page，把 over flow page 的 key-value 重新哈希到这两个 page 上。 具体如何哈希，涉及到全局深度和局部深度，强烈建议看一下[这里](http://www.mathcs.emory.edu/\~cheung/Courses/554/Syllabus/3-index/extensible-hashing-new.html)对 `split` 说的十分清晰。

#### merge

`merge` 是当我删除 key-value 时，bucket 为空了。我要向 bucket 的 split image 去合并。`merge` 是一个类似递归的函数，当 `merge` 之后，要重新 `remove` 一下。而 `remove` 不管是否删除成功都要试着 `merge` 一下。

### Hints

* bucket page 的结构就是一个 key-value array，内部没有排序，插入一个值得时候，就遍历数组一下找到空位就插入。
* `readable_` 代表这里是否有一个值，`occupied_` 代表这里曾经是否有一个值。`array_` 是一个弹性数组，存储实际的 key-value。
* `comparator` 是用来比较 key 的，不是优先队列那样用来排序的。
* 虽然实验要求实现的函数不多，但其实其他的 helper function 对你的主体函数有很大的帮助，大部分都是要实现一下的，当然最好是要用到的时候去实现一下，而不是一上来就全部实现。
* fetch or new page 之后都要 unpin page。
* 对并发不是熟的话，就在 `GetValue` 中加读锁，在 `Insert` 和 `Remove` 中加写锁。先过 case 之后再修。
* 虽然实验指导说的是最好在 `hash_table_bucket_page` 中也加读写锁，但我只在 `extendible_hash_table` 中加锁就通过了 case。
* 对每一个 unpin 都用 assert。
* 别忘了构造函数。
* 尽量减少 `Hash` 函数的使用，它会严重拖慢运行速度。
* 去胡神的群（878405016）里获取测试代码，会极大降低 debug 的时间。

### 值得注意的 C++ 语法

* `reinterpret_cast` 是一种类似 C 风格的转换。 `HashTableDirectoryPage *dir_page = reinterpret_cast<HashTableDirectoryPage *>(buffer_pool_manager->NewPage(&directory_page_id_, nullptr)->GetData())` 可以把某一片地址空间转化为我们想要的类。
