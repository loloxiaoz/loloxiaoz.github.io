# LevelDB

1、项目包含 test、benchmark和库
2、典型的LSM树（Log Structrured-Merge Tree)实现。核心思想上放弃部分读的性能，换取最大的写入能力。写性能极高，简单地说就是尽量减少随机写的次数。 不是将写直接到磁盘，而是通过1）一次日志文件的顺序写，2）一次内存中的数据插入，极高的写性能（顺序写60MB/s,随机写45MB/s)
3、顺序写，批量读（按block）进行块读取
4、块分为first/middle/last，当读到last时表示log结束
5、内存数据库，底层是使用跳表实现
优点：实现简单，效率log(n)，平衡树涉及到节点旋转
原理：利用概率平衡，构造一个快速且简单的数据结构
6、一个拥有k个指针的节点称为k层结点（level node），按照上面概率，50的结点为1层结点，25%的概率为2层几点，12.5%的概率为3层节点，将能获取log（n）的查询效率，插入和删除操作和链表一样
7、put/delete/get均依赖于底层的跳表的基本操作实现
8、sstable分为
data block、filter block、meta Index block、index block、footer
9、leveldb设计Restart point的目的是在读取sstable内容时，加速查找过程，由于每个Restart point存储的都是完整的key值，因此在sstable中进行数据查找时，可以首先利用restart point的数据进行键值比较

hash表的resize
旧hash表被冻结，新hash表进行构建
若此时有读/写需求，去新hash表对应的桶进行查找，如果该桶没有构建，则进行构建并返回值。
当新hash表后台构建进程发现该桶已经构建后，则跳过该桶的构建
因此只需要桶级别的锁就行

Compaction是leveldb最为复杂的过程之一，同样也是leveldb的性能瓶颈之一
内存数据持久化到硬盘，称为minor Compaction
sstable合并的过程，称为major compaction
major compaction本质是一个多路归并的过程，
当0层的文件数量超过 slowdown Trigger时，写入的速度减慢
当0层的文件数量超过pause Trigger时，写入暂停，直至Major Compaction完成
Compaction起到了平衡读写差异的作用

嵌套类，隐藏类名，减少全局的标识符，提高类的抽象能力
noexcept 表示某个函数不会抛出异常，可以给编译器更大的优化空间。
这个在移动构造函数，移动分配函数，析构函数中推荐使用

dllexport和dllimport 是windows下需要使用的，linux下不需要使用
dllimport表示函数是从库中来的，可以优化代码，避免间接寻址，能够提供dll中函数以及变量的信息，如果没有dllimport关键字，编译器不知道从哪里查找static的变量定义，所以会出现link错误

linux下 __attribute__((visibility("default")))

sstable sorting string table
Footer存放该SST元数据
DataBlock存放了具体数据
MetaBlock用于扩展，目前存放了bloomfilter
IndexBlock存放索引，全内存

通过compaction， 减少文件数量， 提高读性能
冷启动加载sstable index， 使用redolog 恢复成memtable
冷启动秒级别
热启动切换从节点到主节点
table可以支持离线和在线业务

1、LevelDB是能够处理10亿级别的KV持久性存储的C++程序库
2、LevelDB不会像redis那样狂吃内存，应该它是将数据持久化到硬盘上的
3、LevelDB再存储中是有序的，可以按照用户自定义的排序函数进行排序
4、LevelDB写性能非常出色，能达到40W+的随机写性能。 达到6W+的随机读性能。
5、LevelDB还支持快照功能(snapshot),使在读取不受写入的影响。 写入过程中，始终看到一致性的数据。 
6、LevelDb主要的静态结构为：  log、memtable、  immutable memtable、  manifest、  current、 sstable
LevelDb写入一条记录时，先顺序写log，为磁盘操作， 再写内存中的memtable。
当memtable满了后，变为immutable memtable。
不断的将immutable刷入磁盘中的sstable，sstable中kv是有序的。 
manifest用来记录每一级的sstable中的范围。由于manifest会随着sstable的改变不断生成新的manifest。因此用current来记录当前是哪一个manifest。 
7、一个log文件的物理大小是32k，每条记录分别有 chccksum、size、type、data。 checksum是校验码，size是每条记录的长度，type为存储类型 /first/middle/last/full。因为有可能一个log文件存不下一条记录。
8、restart point，重启点。由于LevelDB中存储的是有序的KV，因此相邻的两个key之间，可能有部分的重叠，例如the c与the color。    key共享长度、key非共享长度、value长度、key非共享内容、value内容。 
重启点就是完整存储“the color”的。
9、immutable memtable和memtable是完全一样的，区别是immutable是只可读的。
10、memtable 本质是一个skip table。skip table是平衡树的一种替代结构，其插入和删除是很简单的。相对于平衡树来说，skip table可以避免频繁的树节点调整操作，所以写入效率很高。 
11、skip table的核心思想，空间换时间。每个链表增加了向前的指针（即第一个节点有第二个节点和第三个节点的指针。）,提高了查找的效率。相当于给链表建了索引。 
插入元素时，元素所占的层数完全是随机的。
while(random(0,1))
{
    k++;
}
跳表有如下特征：1、每一层都是一个有序链表
                            2、如果一个元素在第i层出现了，那么在第i~0层也都出现
                            3、每个元素包含两个指针，一个指向后向结点，一个指向下一层结点
                            4、最底层的链表包含了所有结点。 
12、随机写都是先记录到日志文件中去，在日志文件满之前只是简单的更新memtable，那么就把随机写转化为了顺序写。 在日志满了后，把日志里面的数据排序写成sst表的同时和之前的sst进行合并，这个动作也是顺序读和写。写日志的时候，用到的是buffer IO，也就是，也就是说如果操作系统有足够的内存。 这个读写全部由操作系统缓冲。 传统的磁盘raid 的顺序读写吞吐量很大。100M左右是没有问题。多线程更新的性能非常好。 
13、levelDb的删除操作也非常简单，插入删除标记，并不真正去删除记录，在compaction时才去做真正的删除操作。
14、读取时，先读取memtable，再读取immutable memtable。是在不行去level0中找，再一层层往下找。
15、查询的时候，levelDb一般会现在内存中Cache中查找是否包含这个文件的缓存记录，如果包含，则从缓存中读取，如果不包含，则打开SSTable文件，同时将索引部分加载到内存中并放入Cache中，这样Cache里面就有了SSTable的缓存索引。 

Bigtable中有三种compaction。
minor compaction就是将immutable memtable导出到SSTable中，major compaction就是将不同层级的SSTable文件合并，full compaction就是将所有SSTABLE 合并。

LEVEL DB中包含其中两种。 minor compaction 和 major compaction
minor compaction 就是将immutable compaction的内容，从小到大遍历一遍，并依此写入level 0的新建SSTable文件中，写完后建立文件的index数据，就完成了一次minor compaction
当某个level下的SSTable文件数据超过一定设置值后，levelDb会从这个level的SSTable中选择一个文件，将其和level+1的SSTable文件合并，这就是major compaction。
对于level0的层级的，由于其key并不会相互重叠。 因此每次major compaction时需要找到所有的重叠的文件和level 1文件进行合并。
对于level 的层级，由于key是有顺序的，因此每次只需要选定一个层级的文件与  level +1的文件进行合并。 具体规则是如果选择了level的key A,那么就选择 level+1的文件与A有重叠的所有文件进行合并，生成level+1的文件。 合并完之后，将之前level 和level+1的所有文件删除。 

redis与memcached比较
1、redis部分数据能够存储到硬盘2、redis具有更加丰富的数据类型
3、如果你对数据同步和数据持久化有比较高的要求的话，推介redis
4、在10w级别数据量上，memcached更加快
5、memcached使用预分配的内存池的方法，预先分配内存，可能会造成内存浪费较大。 redis使用线程申请内存的方式，使用free-list的方式来优化内存分配。可能会造成内存随便，会造成一定内存碎片。
6、memcached提供cas命令，可以保证并发操作数据的一致性。redis没有提供cas命令，但redis提供了事务的功能，可以保证命令的原子性。中间不会被任何操作终端。
7、异构复制，按照不同的维度复制，空间换时间
8、小表异步广播，将小表复制到各个节点上
9、NOS 开放消息系统。 异步与解耦。  异步、解耦、最终一致性、并行
10、无单点、无瓶颈点、集群级别的高可用
11、可以scale out，也可以scale up