# MongoDB

## 适用场景

- 非结构化数据的存储
- 不需要做join查询
- 支持map/reduce进行数据处理
- 支持基于位置的查询
- 性能高、速度快

## 特性

- Capped Collection, 一种固定大小的集合，当集合的大小达到指定的大小时，新数据覆盖老数据
- 千万级别的对象文档，近10G的数据, 对比mysql
  - 对有索引的ID的查询，不会比mysql慢
  - 对于非索引字段的查询，则是全面胜出。mysql实际上无法胜任大数据量下任意字段的查询
  - 写入性能同样令人满意。 百万级别的数据，mongoDB基本10分钟内可以解决
- 内置GridFS，支持大容量存储。本质依然是将文件分块后存储到files.file和files.chunk 2个collection中, 由于GridFs也是一个Collection，你可以直接对文件的属性进行定义和管理。通过这些属性就可以快速找到需要的文件，轻松管理海量的文件，无需费神如何hash才能避免文件系统检索性能问题
- 提供基于Range的auto sharding机制，一个collection可按照记录的范围，分成若干个段，切分到不同的shard上，不同的shard之间可以负载均衡。 查询对客户端是透明的，客户端执行查询，统计、mapReduce等操作，这些会被MongoDb自动路由到后端的数据节点。
- shards可以和复制结合，配合Replica sets能够实现sharding+fail-over