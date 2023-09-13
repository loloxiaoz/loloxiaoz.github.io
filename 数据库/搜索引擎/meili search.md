# MeiliSearch

> MeiliSearch是一个强力、极速、开源且易于使用和部署的搜索引擎, 搜索和索引都可以高度定制化, 搜索词修正,同义词等特性开箱即用，使用Rust编写。

## 简介

### 特性

- 即时搜索，响应时间小于50毫秒
- 全文检索
- 可识别错别字和拼写错误
- 停用词，忽略常见的不相关词，如of or the
- 分面搜索(多标签搜索)
- 支持同义词
- 安装、部署和维护简单
- 返回整个文档
- 高度定制化
- 支持Restful API

### 技术

- 存储引擎使用了[LMDB](http://www.lmdb.tech/doc/)(Lightning Memory-Mapped Database Manager),支持事务的KV存储, 使用C语言开发,用于OpenLDAP. 从性能和稳定性上LMDB是最好的选择
- LMDB将数据存储以[mmap](https://en.wikipedia.org/wiki/Memory-mapped_file)形式存储,内存映射文件. 由操作系统管理真正的内存分配, 当MeiliSearch和其他程序一起部署时, 操作系统将按照各个程序的需求灵活的管理内存

### 适用场景

- MeiliSearch vs Elasticsearch
- MeiliSearch无法处理复杂查询和海量数据
- ES大部分时候响应速度慢于MeiliSearch
- ES能够按照Index进行sharding，并实现高可用
- 如果需要一个简单、容错、提供前缀搜索能力并能根据关联度排序的查询搜索bar, MeiliSearch是一个很好的选择

## 使用介绍

### 查询设置

| Name                                                                                                         | Type             | Default value                                                                                    | Description                                                        |
| ------------------------------------------------------------------------------------------------------------ | ---------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| **[`displayedAttributes`](https://docs.meilisearch.com/reference/api/settings.html#displayed-attributes)**   | Array of strings | All attributes: `["*"]`                                                                          | 查询数据时可显示的属性，默认显示全部                               |
| **[`distinctAttribute`](https://docs.meilisearch.com/reference/api/settings.html#distinct-attribute)**       | String           | `null`                                                                                           | 查询时指定唯一属性，当多条查询结果的某个字段具有相同值时只返回一条 |
| **[`faceting`](https://docs.meilisearch.com/reference/api/settings.html#faceting)**                          | Object           | [Default object](https://docs.meilisearch.com/reference/api/settings.html#faceting-object)       | 分面搜索                                                           |
| **[`filterableAttributes`](https://docs.meilisearch.com/reference/api/settings.html#filterable-attributes)** | Array of strings | Empty                                                                                            | 查询时可用来做过滤条件的属性                                       |
| **[`pagination`](https://docs.meilisearch.com/reference/api/settings.html#pagination)**                      | Object           | [Default object](https://docs.meilisearch.com/reference/api/settings.html#pagination-object)     | 分页查询设置                                                       |
| **[`rankingRules`](https://docs.meilisearch.com/reference/api/settings.html#ranking-rules)**                 | Array of strings | `["words",` `"typo",` `"proximity",` `"attribute",` `"sort",` `"exactness"]`                     | 排序规则的影响程序设置，第一个影响最大                             |
| **[`searchableAttributes`](https://docs.meilisearch.com/reference/api/settings.html#searchable-attributes)** | Array of strings | All attributes: `["*"]`                                                                          | 指定可用来搜索匹配的字段，影响查询结果的排序顺序                   |
| **[`sortableAttributes`](https://docs.meilisearch.com/reference/api/settings.html#sortable-attributes)**     | Array of strings | Empty                                                                                            | 设置用于排序的属性                                                 |
| **[`stopWords`](https://docs.meilisearch.com/reference/api/settings.html#stop-words)**                       | Array of strings | Empty                                                                                            | 查询时忽略的单词                                                   |
| **[`synonyms`](https://docs.meilisearch.com/reference/api/settings.html#synonyms)**                          | Object           | Empty                                                                                            | 同义词                                                             |
| **[`typoTolerance`](https://docs.meilisearch.com/reference/api/settings.html#typo-tolerance)**               | Object           | [Default object](https://docs.meilisearch.com/reference/api/settings.html#typo-tolerance-object) | 错别字容忍                                                         |

## 总结

### 限制

- 检索项长度: **检索项中包含的单词不能超过10个**
- 字段数量: 每个文档最多**1000**个属性字段
- 属性值长度: **属性字段长度不超过65535**
- 索引最大文档条数: 不能超过**4,294,967,296**
- 索引大小限制: 不能超过**500GB**

### 功能性缺陷

- 不支持lucene部分语法，如分词查询、通配符查询、近似查询等
- 不支持multi-index search

### 非功能性缺陷

- 扩展性: 不支持水平扩展, 只能单实例部署
- 高可用: 不支持
- 可维护性: 版本暂时未达到1.0，不保证向后兼容
- 性能: 查询性能与资源受限程度、数据量大小成反比， 资源受限时，写入性能不高, 只有不到1000
- 资源控制: 内存、磁盘占用较高, 且内存管理不受程序自身控制, 而是由操作系统控制，当写入大批量数据时，如果机器本身资源不够系统会中断meilisearch及其所有进程

### 优点

- 成熟度高, 开箱即用, 提供完善的Restful API, 多语言SDK(Go/Java/Rust/Python)
- 工具链完善, 自带查询界面, 支持K8s部署
- 性能高, 查询速度在50ms内
- 查询功能完备且强大, 自定义程度高(同义词、停用词、排序规则、查询属性和分面搜索属性)

### 结论

根据以上总结，Meilisearch目前在功能性方面还不满足告警管理要求，与ElasticSearch相比，功能性、成熟度和可用度都还有一定差距。但是可以在大禹官网中先尝试应用起来，例如用来搜索问题文档或者帮助手册。另外meilisearch官网的文档搜索也是用meilisearch实现的，具有较高的参考性。

## 参考文献

- <https://docs.meilisearch.com/learn/what_is_meilisearch/comparison_to_alternatives.html#comparison-table/>
- <https://typesense.org/typesense-vs-algolia-vs-elasticsearch-vs-meilisearch/>