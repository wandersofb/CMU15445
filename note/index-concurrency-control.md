# Index Concurrency Control
此节考虑多线程情况下的锁的使用。

### Locks vs. Latches
二者都是锁的概念
|          | Locks                                | Latches                   |
|----------|--------------------------------------|---------------------------|
| Separate | User Transactions                    | Threads                   |
| Protect  | Database Contents                    | In-Memory Data Structures |
| During…  | Entire Transactions                  | Critical Sections         |
| Modes    | Shared, Exclusive, Update, Intention | Read, Write               |
| Deadlock | Detection & Resolution               | Avoidance                 |
| …by…     | Waits-for, Timeout, Aborts           | Coding Discipline         |
| Kept in… | Lock Manager                         | Protected Data Structure  |

简单来说，Latches 是针对下层的线程的概念用于保护数据结构，它有读写之分，分别用于不同的临界情况。而 Lockes 是对于上层的事务的概念用于保护数据内容，针对事务的分享、更新等。

### Latch Modes
- Read Mode
  1. Multiple threads can read the same object at the same time.
  2. A thread can acquire the read latch if another thread has it in read mode.
- Write Mode
  1. Only one thread can access the object.
  2. A thread cannot acquire a write latch if another thread has it in any mode.

关于读写锁就不多说了。

### Latch Implementations
我们自己来实现一下读写锁。

#### Blocking OS Mutex
说的就是 `std::mutex`，但其实用 OS 提供的锁并不好，它是会启用 OS 的线程调度，那样就太昂贵了且不可扩展。所以基本不会用。

#### Test-and-Set Spin Latch (TAS)
自己实现 spin latches 是一个更好的替代品相较于 OS mutex。简单来说就是第一个进入 spin 的线程通过，其他进入 spin 的都会陷入 while 循环中或者交给 OS 重新调度。但是这样不可扩展，对 cache 不友好，

#### Reader-Writer Latches
- Allows for concurrent readers
- Must manage read/write queues to avoid starvation
- Can be implemented on top of spin latches

### Hash Table Latching
给 hash table 上锁很简单，因为 hash table 都是单向的，直接上锁，到下一个的时候，解除上一个即可。
不过上锁有两种方式。
- Approach #1: Page Latches
  1. Each page has its own reader-writer latch that protects its entire contents.
  2. Threads acquire either a read or write latch before they access a page.
- Approach #2: Slot Latches
  1. Each slot has its own latch.
  2. Can use a single-mode latch to reduce meta-data and computational overhead.

当然也可能实现 lock-free 代码，这就不细说了，太难了。

### B+ Tree Concurrency Control
重头戏来了，我们希望有多个线程能够在 B+ Tree 上在同一时间读取和更新数据。有两个难点。
- 多个线程在同一时间修改 node 的内容。
- 一个线程遍历 tree 同时另一个线程 split/merge node。

#### Latch Crabbing/Coupling
**Basic Idea**:
- Get latch for parent
- Get latch for child
- Release latch for parent if “safe”

A **safe node** is one that will not split or merge when updated.
- Not full (on insertion)
- More than half-full (on deletion)

两种操作：
**Find**: Start at root and go down; repeatedly,
- Acquire R latch on child
- Then unlatch parent

**Insert/Delete**: Start at root and go down, obtaining W latches as needed. Once child is latched, check if it is safe:
- If child is safe, release all latches on ancestors

#### Better Latching Algorithm
**Search**: Same as before.
**Insert/Delete**: 
- Set latches as if for search, get to leaf, and set W latch on leaf.
- If leaf is not safe, release all latches, and restart thread using previous insert/delete protocol with write latches.

乐观锁其实相较于朴素锁，就是假设 split/merge 的操作并不多，每次我都优先获取 read 锁，当到达 leaf node 看是否 split/merge，如果不，则获取写锁，更新，如果是，则重新来一遍朴素锁的过程。

#### Leaf Node Scans
通常做范围查询时，会在 leaf node 进行横向扫描，这个时候如果出现两个不同方向的 scan 时，就会 deadlock。解决办法是如果一个锁要等等待时，就不等待直接 abort，重现操作。
