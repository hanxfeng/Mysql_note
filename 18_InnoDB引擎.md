逻辑存储结构：
	擅长事务处理，具有崩溃恢复特性，InnoDB引擎的存储结构类似包含关系，上级包含n个下级，具体结构为：
		表空间
		段
		区
		页
		行
	表空间：
		对应磁盘中的.ibd文件，一个mysql实例可以对应多个表空间，用于存储记录，索引等数据。
	段：
		分为数据段，索引段，回滚段，InnoDB是索引组织表，数据段就是B+树的叶子节点，索引段为B+树的非叶子节点，段用来管理多个区。
	区：
		表空间的单元结构，每个区大小为1M，默认情况下一个区有64个页，每个页16k
	页：
		InnoDB磁盘的最小单元，默认大小为16k,为了保证页的连续性，InnoDB引擎每次从磁盘申请4到5个区
	行：
		InnoDB引擎数据按行进行存放
架构：
	内存架构：
		缓冲区（buffer pool）：
			缓冲区是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增删改查操作时，先操作缓冲区中的数据，（如果缓冲区没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘IO，加快处理速度
			缓冲区以页为单位。底层采用链表数据结构管理页，根据状态，将页分为三种类型：
				空闲页（free page）：未被使用的页
				clean page：被使用的页，页中数据没有被修改过
				dirty page：被使用的页，页中数据被修改过与磁盘中数据不一致
		更改缓冲区（change buffer）：
			更改缓冲区，针对非唯一二级索引，在执行DML语句时，如果这些数据页没有在缓冲区中，不会直接操作磁盘，而是会将数据变更存在更改缓冲区中，在未来数据被读取时，再将数据合并到缓冲区中，再将合并后的数据刷新到磁盘中
			意义：
				与聚集索引不同，二级索引通常是非唯一的，并且以相对随机的方式插入二级索引，同样，删除和更新可能影响索引树中不相邻的二级索引页，如果每一次都操作磁盘，则会造成大量磁盘IO,有了更改缓冲区后，可以在缓冲池中进行合处理，减少磁盘IO
		自适应哈希索引（adaptive hash index）：
			自适应哈希索引，用于优化对缓冲区数据的查询，InnoDB引擎会监控表上各索引页的查询，如果观察到建立哈希索引可以提升速度，则建立哈希索引，称之为自适应哈希索引
		日志缓冲区（log buffer）：
			用来保存要写入磁盘中的日志数据。默认大小为16MB。日志缓冲区的日志会定期刷新到磁盘中，如何需要更新，插入，删除许多行的事务，增加日志缓冲区大小可节省磁盘IO
	磁盘架构：
		系统表空间（system tablespace）：
			系统表空间是更改缓冲区的存储区域，如果表是在系统表空间而不是每个表文件或通用表空间中创建的，它也可能包含表和索引数据
		独立表空间（file-per-table tablespace）：
			每个表的文件表空间包含单个InnoDB表的数据和索引，并储存在文件系统上的单个数据文件中
		通用表空间（general tablespace）：
			需要通过`creat tablespcace 表空间名 add 表名 engine = '引擎名'`创建，在创建表时可以指定该空间
		撤销表空间（undo tablespace）：
			MySQL实例在创建时会自动创建两个撤销表空间（初始大小16M），用于存储undo log日志
		临时表空间（temporary tablespace）：
			存放InnoDB引擎的临时表空间和全局临时表空间，储存用户创建的临时表等数据
		双写缓冲区（doublewrite tablespace）：
			InnoDB引擎将数据从缓冲区（buffer pool）刷新到磁盘前，先将数据写入双写缓冲区，便于系统异常时恢复数据，文件格式为.dblwr
		重做日志（redo log）：
			用来实现事务的持久性，该日志文件由两部分组成，重做日志缓冲区（redo log buffer）及重做日志文件（redo log），前者在内存中后者在磁盘中，事务提交后所有修改信息都会存到该日志中，用于刷新脏页到磁盘时，发生错误时恢复磁盘使用
	后台线程：
		master thread：
			核心后台线程，负责调度其他线程，还负责将缓冲区的数据异步刷新到磁盘中，保持数据一致性，还负责脏页的刷新，合并插入缓存，undo页的回收
		IO thread：
			InnoDB引擎中使用了大量异步IO（AIO）来处理IO请求，极大提升数据库性能，IOthread主要负责这些IO请求的回调，有：
				read thread：默认4个，负责读操作
				write thread：默认4个，负责写操作
				log thread：默认1个，负责将日志缓冲区刷新到磁盘
				insert buffer thread ：默认1个，负责将更改缓冲区内容刷新到磁盘
		purge thread：
			主要用于回收事务已经提交了的undo log,事务提交后，不再需要undo log防止出现错误，就使用purge thread回收undo logo
		page cleaner thread：
			协助master thread刷新脏页到磁盘的线程，用于减轻master thread的压力，减少阻塞
	架构部分总结：
		在操作时直接操作缓冲区的数据，如果缓冲区没有数据，那么会读取磁盘中的数据，增删改查操作是对缓冲区中的数据进行操作。同时，缓冲区中的数据会以一定频率通过各个线程刷新到磁盘中
事务原理：
	原理分类：
		原子性，一致性，持久性：
			事务的这三个特性主要由redo log和undo log保证
		隔离性：
			事务的这个特性主要由锁机制和MVCC（多版本并发控制）保证
	redo log（重做日志）：
		保证日志的持久性，具体介绍见  18_InnoDB引擎-架构-磁盘架构-重做日志（redo log）
		在执行事务的增删改操作时，相关操作的物理修改日志会立即记录到内存的 redo log buffer（重做日志缓冲区）中； 事务提交时，InnoDB 会将该事务对应的 redo log 从 对应的reao log buffer 强制刷入磁盘的 redo log 文件（ib_logfile），这是保障事务持久性的核心； 如果数据库崩溃（如断电），此时 Buffer Pool 中修改后的脏页可能还未异步刷入磁盘，重启时 InnoDB 会重放 redo log 中所有已提交事务的日志，将数据恢复到崩溃前的状态，以此保证事务的持久性（即已提交的修改不会丢失）。
	undo log：
		undo log（回滚日志），用于保证日志的原子性，作用有两个：1.提供回滚和MVCC（多版本并发控制），和redo log记录物理日志不同，undo是逻辑日志，比如对数据进行delete操作时，undo log中记录的是对这条数据的insert操作，进行update操作时，undo log记录的是反向的update操作
		销毁：
			undo log 在事务执行时产生，事务提交时不会立即删除undo log，未删除的undo log还可能用于MVCC
		储存：
			采用段的方式进行储存，存放在段中的回滚段（rollback segment ）中，内部包含1024个undo log segment
	MVCC（多版本并发控制）：
		基本概念：
			当前读：
				读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录加锁，select ... lock in share mode（共享锁）、select ... update（排他锁），insert,update,delete都是一种当前读
			快照读：
				简单的select（不加锁）就是快照读，读取的是记录数据的可见版本，有可能是历史数据，读取时不加锁，是非阻塞读
				不同隔离级别效果：
					read committed：每次select都生成一个快照读
					reqeatabel read：开启事务后第一个select产生快照读，之后的select都是读取的这个快照读
					serializable：快照读会退化为当前读
			MVCC介绍：
				指维护一个数据的多个版本，是读写操作没有冲突，快照读为MySQL实现MVCC提供了一个非阻塞读功能，MVCC的实现还依赖数据库记录中的三个隐式字段（DB_TRX_ID , DB_ROLL_PTR , DB_ROW_ID），undo log日志 readview
			三个隐藏字段：
				 DB_TRX_ID：
					 代表最近修改事务id,记录插入这条记录或最后一次修改该记录的事务id
				DB_ROLL_PTR：
					代表回滚指针，指向这条记录的上一个版本，用于配合undo log日志，找到这条数据的上一个版本
				DB_ROW_ID：
					代表隐式主键，不是每个表都有，如果一个表结构没有主键，这个隐藏字段则不会出现
			undo log日志：
				回滚日志，在insert/delete/update时产生
				insert时undo log日志只在回滚时需要，事务提交后立刻删除
				update/delete时，undo log日志还在快照读时使用，不会被立即删除
				undo log版本链：
					不同事务或相同事务对同一条记录修改，会导致该记录的undo log日志生成一条版本链，链表头部是最新的旧记录，尾部是最旧的旧记录
			readview（读视图）：
				readaview 是快照读sql执行时MVCC提取数据的核心依据记录并维护系统当前活跃的事务（未提交的）id
				readview包括四个字段：

| 字段             | 含义                 |
| -------------- | ------------------ |
| m_ids          | 当前活跃事务的id集合        |
| min_trx_id     | 最小活跃事务id           |
| max_trx_id     | 预分配事务id，为当前最大事务id+ |
| creator_trx_id | readview创建者的事务id   |
				![[Pasted image 20251126143812.png]]
				不同隔离级别生成readview时机不同：
					read committed：在事务每一次执行快照读时生成readview
					repeatable read：仅在事务中第一次执行快照读时生成readview,后续复用该readview