- JUC: java.uitl.concurrent 工具包
	- jdk1.5+
	- JSR 166 规范的实现
	- 大量使用了CAS
- CAS
	- Compare and Swap
		- 1. 线程读取当前内存中的值，这个值就是期待的值（expected value）。
		- 2. 线程计算出新的值（即要更新的值）。
		- 3. 线程尝试用 CAS 操作将新的值写入内存中。这个操作会比较内存中当前的值和之前读取的期待的值，如果相等则更新，否则不做任何操作。
		- 4. 如果 CAS 操作成功，则线程完成操作；如果 CAS 操作失败，则线程会重新读取内存中的值，重新计算新的值，然后再次尝试 CAS 操作。
	- ![image.png](../assets/image_1710834901903_0.png){:height 388, :width 680}
	- 是一种乐观锁机制，通过原子性地比较当前内存中的值与预期值，如果相等则进行交换操作。
	- CAS 操作能保证线程安全的主要原因
		- **原子性：** CAS操作是原子性的，即在执行过程中不会被其他线程中断，要么执行成功，要么失败，不会出现线程切换的情况。这保证了多线程环境下对共享变量的操作是原子的，从而避免了竞态条件和数据不一致的问题。
		- **比较与交换：** 在比较的过程中，如果发现当前内存中的值与预期值不相等，则说明该共享变量在此时已经被其他线程修改了，此时CAS操作会失败，不会执行交换操作。这种不可分割性保证了在执行CAS操作期间其他线程不会修改共享变量，从而避免了竞态条件。
			- **比较：** CAS操作首先会比较当前内存中的值与预期值是否相等。这个预期值是我们期望的旧值。
			- **交换：** 如果当前内存中的值与预期值相等，则执行更新操作，将新值写入到内存中；否则，不做任何操作。
		- **无锁化：** CAS操作是一种无锁的同步机制，与传统的锁机制相比，它不需要线程阻塞和唤醒、不需要进入内核态，减少了线程上下文切换和系统调用的开销，提高了并发性能。
			- 通常会在一个循环中不断尝试执行CAS操作，直到操作成功。这种方式称为自旋（Spin）
	- ABA 问题
		- 共享变量的值被修改了两次的情况下。假设共享变量的初始值为A，在某个时刻被修改为B，然后又被修改回A，这样看似没有变化，但实际上已经发生了两次修改。
		- 解决方式
			- 业务逻辑上不在乎过程，只在乎结果是否是预期值，可以忽略ABA问题
			- 添加版本号，如 **AtomicStampedReference**
			- 添加修改标记，如 **AtomicMarkableReference**
- 锁的分类
	- 按 Lock 方式
		- 隐式锁
			- synchronized, 不需要显式的 Lock 和 UnLock
		- 显式锁
			- JUC 包中提供的锁，需要显示的 Lock 和 UnLock
	- 按特性
		- 乐观锁、悲观锁：按照线程在使用共享资源时，要不要锁住同步资源，划分为悲观锁和乐观锁
			- 悲观锁：JUC锁，synchronized
			- 乐观锁：CAS
		- 重入、不可重入：按照同一个线程是否可以重复获取同一把锁，划分为重入锁和不可重入锁
			- 重入锁： ReentrantLock、synchronized
			- 不可重入锁：基本没有了
		- 公平锁、分公平锁：按照多个线程竞争同一锁时需不需要排队，能不能插队，划分为公平锁和非公平锁。
			- 公平锁：new ReentrantLock（true）多个线程按照申请锁的顺序获取锁
				- 任务调度器
				- 消息队列消费者
				- 资源池管理
				- 请求处理器
			- 非公平锁：new ReentrantLock（false）多个线程获取锁的顺序不是按照申请锁的顺序（可以插队、synchronized
				- 优点：可以实现更高的吞吐量，因为它允许后到达的线程直接争夺锁，而无需等待，从而减少了线程切换的开销。
				- 缺点：非公平锁可能会导致线程饥饿（某些线程始终无法获得锁），因为总是有一些线程可能会被其他线程连续抢占锁。
		- 独享锁、共享锁：按照多个线程能不能同时共享同一个锁，锁被划分为独享锁和排他锁。
			- 独享锁：独享锁也叫排他锁，synchronized、ReentrantLock、ReentrantReadWritelock 的WriteLock写锁
			- 共享锁：ReentrantReadWriteLock的ReadLock读
		- 其他
			- 自旋锁
				- CAS: Atomic
				- synchronized 的轻量级锁（状态）
			- 分段锁
				- ConcurrentHashMap (jdk1.8+)
					- 首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。
			- 无锁/偏向锁/轻量级锁/重量级锁
				- synchronized独有的四种状态，级别从低到高
				- 是JVM为了提高synchronized锁的获取与释放效率而做的优化
				- 四种状态会随着竞争的情况逐渐升级，而且是不可逆的过程，即不可降级。
- 与 Synchronize 主要对比
	- 无法控制阻塞时长，阻塞不可中断
	- 不能细粒度控制，比如JUC的读写锁可以区分读写的场景
- 常用的JUC锁
	- [[ReentrantLock 重入锁]]
	- [[ReentrantReadWriteLock 重入读写锁]]
		- 维护一个读锁和一个写锁
		- 适合读多写少的场景
	- #StampedLock 冲压锁
		- ReentrantReadWriteLock 的增强版
		- 实现了一种乐观读取的锁机制，允许读取操作不需要获取独占锁，但在乐观读取过程中，可能会失败并需要重新尝试。
- 容器工具
	- [[CountDownLatch]]
	- [[Semaphore]]
	- [[CyclicBarrier]]
	- [[Condition]]
- 并发容器
	- List
		- Vector
			- #synchronized 实现的同步容器，性能差，适合于对数据有强一致性要求的场景
		- CopyOnWriteArrayList
			- 底层数组实现，使用复制副本进行有锁写操作（数据不一致问题）
			- 适合读多写少，允许短暂的数据不一致的场景
				- 读不加锁，读写不冲突
					- 平时查询时，不加锁，更新时从原来的数据cOpy副本，然后修改副本，最后把原数据替换为副本。修改时，不阻塞读操作，读到的是旧数据。
	- Map
		- Hashtable
			- #synchronized 实现的同步容器，性能差，适合于对数据有强一致性要求的场景
		- ConcurrentHashMap
			- 底层数组＋链表＋红黑树（JDK1.8）实现，对table数组entry加锁（synchronized）
				- 存入key值，使用hashCode映射数组索引
				- 集合会自动扩容：加载因子0.75f
				- 链表长度超过8时，链表转换为红黑树
			- 存在一致性问题，适合存储数据量小，读多写少，允许短暂的数据不一致的场景
				- **复合操作的非原子性**
					- 虽然 `ConcurrentHashMap` 中的单个操作是原子性的，但某些复合操作不是原子性的。例如，`putIfAbsent(key, value)` 是一个常见的复合操作，它先检查键是否存在，如果不存在则插入键值对。在多线程环境下，多个线程可能同时检查到键不存在，然后都尝试插入，最终导致重复插入相同的键值对。
				- **并发更新相同键的情况**
					- 多个线程同时对相同的键进行更新操作可能会导致数据不一致。例如，如果两个线程同时尝试在相同的键上执行 `put(key, value)` 操作，可能会导致其中一个线程的更新被覆盖，从而丢失数据。
				- **迭代与修改操作同时进行**
					- 虽然 `ConcurrentHashMap` 提供了弱一致性的迭代器，但在迭代集合的同时对其进行修改仍然可能导致数据不一致的情况。如果在迭代集合的同时有其他线程对其进行插入、删除等修改操作，可能会导致迭代器无法准确地反映出集合的最新状态。
				- **不一致的读取操作**
					- 在并发环境下，即使是读取操作也可能会导致数据不一致的情况。例如，在一个线程对某个键进行更新操作时，另一个线程可能同时读取到旧值，导致读取操作不一致。
		- ConcurrentSkipListMap
			- 底层跳表实现，使用CAS实现无锁读写操作。
			- 适合与存储数据量大，读写频繁，允许短暂的数据不一致的场景
		- Set
			- CopyOnWriteArraySet
				- 底层基于跳表实现的有序Set
			- ConcurrentSkipListSet
				- 底层基于跳表实现的有序Set
		- Queue
			- 队列是线程协作的利器，通过队列可以很容易的实现数据共享，并且解决上下游处理速度不匹配的问题，典型的生产者消费者模式。
			- 阻塞队列
				- 一端是给生产者put数据使用，另一端给消费者take数据使用
				- 是线程安全的，生产者和消费者都可以是多线程
				- take方法：获取并移除头元素，如果队列无数据，则阻塞
				- put方法：插入元素，如果队列已满，则阻塞
				- 有界和 无界
					- 有界指队列有大小限制
					- 无界队列不是无限队列，最大值Integer.MAX_VALUE
			- 常用
				- ArrayBlockingQueue 基于数组实现的有界阻塞队列
				  id:: 6601567b-d4d0-4ecf-abd3-d57d044f3f19
				- LinkedBlockingQueue 基于链表实现的有界阻塞队列
				- SynchronousQueue 不存储元素的阻塞队列
				- PriorityBlockingQueue 支持按优先级排序的无界阻塞队列
				- DelayQueue 优先级队列实现的双向无界阻塞队列
				- LinkedTransferQueue 基于链表实现的无界阻塞队列
				- LinkedBlockingDeque 基于链表实现的双向无界阻塞队列
					- **容量限制**:
						- `PriorityBlockingQueue` 是无界的，可以动态地增长以容纳更多的元素。
						- `ArrayBlockingQueue` 是有界的，其容量在创建时被固定，无法存储超过其容量限制的元素。
					- **元素排序**:
						- `PriorityBlockingQueue` 中的元素是按照优先级顺序进行排序的，元素被取出时，总是取出优先级最高的元素。它按照元素的自然顺序或者通过构造函数提供的 `Comparator` 进行排序，插入和取出操作的时间复杂度为 O(log n)
						- `ArrayBlockingQueue` 中的元素按照 FIFO 的顺序进行排序，即先进入队列的元素会被先取出。插入和取出操作的时间复杂度为 O(1)
	-