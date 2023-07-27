# kafka

作用
1、解耦 
2、冗余
3、削峰填谷
4、异步
5、扩展性

rabiitMQ，本身支持很多协议，非常重量级，适合企业级开发。
zeroMQ 最快的消息队列系统，针对大吞吐量的需求场景，但仅提供非持久性的队列

kafka相对于ActiveMQ 是一个轻量级的消息系统，还是一个工作良好的分布式系统

基于zookeeper
顺序写磁盘比随机写内存效率还要高，这是kafka高吞吐率的一个重要保证
kafka集群保留所有的消息，无论是否被消费，基于策略删除旧数据

kafka的broker完全是无状态的，不需要标记哪些消息被消费过，也不需要通过broker保证同一个consumer Group只有一个consumer能消费一条消息
如果partition设置合理，所有消息可以均匀分布到不同的partition里。 不同的消息可以并行写入不同的partition，极大的提高了吞吐率。

在发送一条消息时，可以指定这条消息的key，producer根据这个key和partition机制判断应该将这条消息发送到哪一个partition里。

同一topic的一条消息只能被同一个consumer group内的一个consumer消费，但可以被多组consumer group消费。

实现单播只要所有的consuemer在同一个group里。 用consumer group还可以将consumer进行自由的分组而不需要发送多次消息到不同的topic

kafka的设计理念就是同时提供离线处理和实时处理。

push模式很难适应消费速率不同的消费者。 因为消息发送速率是由broker决定的。 push模式的目标是尽可能以最快的速度传递消息。 但是容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。

kafka而言，pull模式更合适。 consumer可自主控制消费消息的速率，同时也可以控制消费方式，  即可批量消费也可逐条消费。


at most once 最多一次，有可能会丢，但绝不会重复传输
at least one 消息绝不会丢，但有可能重复
exactly once 每条消息肯定会被传输，并且仅传输一次。 很多时候也是用户想要的

consumer读消息前，先commit offset 再处理，这种就是 at most once
consumer读消息先处理，再commit offer，就是 at least once
如果消息有主键，可以近似认为是 exactly once

producer通过zookeeper找到partition的leader，然后producer只将该消息发送到该partition的leader上，leader会将消息写入本地log， 每个follower都从leader pull数据。 当所有follow都收到消息并写入log后，向leader发送ack。 leader收到所有replica的ack后，向producer发送ack

consumer也是从leader读取消息，只有被commit过的消息，才会暴露给consumer

kafka 定义一个broker是否活着。 1、必须维护与zookeeper的sesstion  2、必须能及时将leader的消息复制过来，不能“落后太多”。   follow可以批量从leader复制数据

kafka使用的选举算法更像微软的 pacificA算法。 
kafka在所有的broker中选出一个controller，所有的leader选举都有controller决定。 controller会将leader的改变直接通过rpc的方式通知为此作为响应的broker。 同时controller也负责增删topic和replica的重新分配
consumer的 rebalance算法


partition是最小并发粒度，提供了并行处理的能力
ISR实现了可用性与数据一致性的动态平衡（in-sync replica set), 如果follower长时间未向leader同步数据， 该follower将被提出ISR
1、避免了最慢的follow拖慢整体速度2、只有commit过的消息才会被consumer消费，故提高了数据的一致性3、可以只包含leader，极大提高了可容忍的宕机的follower的数量。

顺序写磁盘
多disk driver
零拷贝， 通过DMA烤包到NIC buffer，无需cpu拷贝，这也是零拷贝的说法来源
批处理
减小网络开销
高效的序列化方式


去除zookeeper的kafka， 主要是减少运维成本， 单节点可以承载的分区规模。 采用了raft协议，实现分布式选举等功能
Pulsar 是计算存储分离，但也依赖zookeeper

使用zk强一致性选举集群的controller， controller对整个集群的管理，包括ISR列表的维护，分区的新增，很多功能都要靠controller来实现，基本思想是 kafka on kafka  将kafka元数据存储在kafka本身中

kafka是分布式消息引擎系统

压缩： LZ4 > Snappy > zstd > GZIP
压缩比： zstd > LZ4 > GZIP > Snappy
Producer 端压缩、 Broker端保持、Consumer端解压缩 

Kafka 只对已提交的消息做有限度的持久化保证

kafka消息保证
1、Producer永远使用producer.send(msg, callback)
