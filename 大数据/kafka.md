# kafka

> 分布式消息引擎系统

## 使用场景

kafka的设计理念就是同时提供离线处理和实时处理。

- 解耦
- 冗余
- 削峰填谷
- 异步
- 扩展性

## kafka消息可靠性机制

- at most once: 最多一次，有可能会丢，但绝不会重复传输, consumer读消息前，先commit offset 再处理，这种就是 at most once
- at least one 消息绝不会丢，但有可能重复, consumer读消息先处理，再commit offset，就是 at least once
- exactly once 每条消息肯定会被传输，并且仅传输一次。 如果消息有主键，可以近似认为是 exactly once

## kafka为什么性能高

- 零拷贝: 通过DMA烤包到NIC buffer，无需cpu拷贝，这也是零拷贝的说法来源。
- 顺序写:顺序写磁盘比随机写内存效率还要高，这是kafka高吞吐率的一个重要保证。
- 多partition: partition是最小并发粒度，提供了并行处理的能力, 如果partition设置合理，所有消息可以均匀分布到不同的partition里。 不同的消息可以并行写入不同的partition，极大的提高了吞吐率。
- 多磁盘: 支持多磁盘driver。
- 批处理: 减小网络开销。
- 高效的序列化方式。
- 压缩算法: ： LZ4 > Snappy > zstd > GZIP, 压缩比： zstd > LZ4 > GZIP > Snappy。

## kafka为什么能保证消息可靠性

- 一致性协议: zookeeper的zab协议,  强一致性协议，消息提交需要全部的foller都确认后再发送ack。
- 消息持久化: kafka集群保留所有的消息，无论是否被消费，基于策略删除旧数据, 只对已提交的消息做有限度的持久化保证。
- ISR: ISR实现了可用性与数据一致性的动态平衡（in-sync replica set), 如果follower长时间未向leader同步数据， 该follower将被提出ISR。 1、避免了最慢的follow拖慢整体速度2、只有commit过的消息才会被consumer消费，故提高了数据的一致性3、可以只包含leader，极大提高了可容忍的宕机的follower的数量

## kafka的典型设计

- 消息发送: producer通过zookeeper找到partition的leader，然后producer只将该消息发送到该partition的leader上，leader会将消息写入本地log， 每个follower都从leader pull数据。 当所有follow都收到消息并写入log后，向leader发送ack。 leader收到所有replica的ack后，向producer发送ack
- 消费方式: 同一topic的一条消息只能被同一个consumer group内的一个consumer消费，但可以被多组consumer group消费。
- 分区发送方式: 在发送一条消息时，可以指定这条消息的key，producer根据这个key和partition机制判断应该将这条消息发送到哪一个partition里。
- 消费模式: push模式很难适应消费速率不同的消费者,因为消息发送速率是由broker决定的。push模式的目标是尽可能以最快的速度传递消息。 但是容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。kafka而言，pull模式更合适。 consumer可自主控制消费消息的速率，同时也可以控制消费方式，  即可批量消费也可逐条消费。
- broker完全无状态: 不需要标记哪些消息被消费过，也不需要通过broker保证同一个consumer Group只有一个consumer能消费一条消息
- broker是否活着: 1、必须维护与zookeeper的sesstion  2、必须能及时将leader的消息复制过来，不能“落后太多”
- controller: 从所有的broker中选出一个controller，所有的leader选举都由controller决定。 controller会将leader的改变直接通过rpc的方式通知为此作为响应的broker。 同时controller也负责增删topic和replica的重新分配

## 与其他消息队列的对比

- rabiitMQ，本身支持很多协议，非常重量级，适合企业级开发。
- zeroMQ 最快的消息队列系统，针对大吞吐量的需求场景，但仅提供非持久性的队列
- kafka相对于ActiveMQ 是一个轻量级的消息系统，还是一个工作良好的分布式系统