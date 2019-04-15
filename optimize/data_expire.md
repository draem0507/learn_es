## 数据过期

### 删除索引

DELETE  /logs_2018*

删除索引，简单暴力；然而实际项目中，无法保证，这些非热点数据，哪一天又被突然想起来。

### 迁移旧索引

随着数据被记录，很有可能存在一个 *热点* 索引——今日的索引。 所有新文档都会被加到那个索引，几乎所有查询都以它为目标。那个索引应当使用你最好的硬件。

```bash
/bin/elasticsearch --node.box_type strong
```

`box_type` 参数是完全随意的——你可以将它随意命名只要你喜欢——但你可以用这些任意的值来告诉 Elasticsearch 将一个索引分配至何处。

我们可以通过按以下配置创建今日的索引来确保它被分配到我们最好的服务器上：

```json
PUT /logs_2019-04-01
{
  "settings": {
    "index.routing.allocation.include.box_type" : "strong"
  }
}
```

昨日的索引不再需要我们最好的服务器了，我们可以通过更新索引设置将它移动到标记为 `medium` 的节点上：

```json
POST /logs_2014-09-30/_settings
{
  "index.routing.allocation.include.box_type" : "medium"
}
```

### 索引优化

昨日的索引不大可能会改变。 日志事件是静态的：已经发生的过往不会再改变了。如果我们将每个分片合并至一个段（Segment），它会占用更少的资源更快地响应查询。 

对还分配在 `strong` 主机上的索引进行优化（Optimize）操作将会是一个糟糕的想法， 因为优化操作将消耗节点上大量 I/O 并对索引今日日志造成冲击。但是 `medium` 的节点没有做太多类似的工作，我们可以安全地在上面进行优化。

昨日的索引有可能拥有副本分片。 如果我们下发一个优化（Optimize）请求， 它会优化主分片和副本分片，这有些浪费。然而，我们可以临时移除副本分片，进行优化，然后再恢复副本分片：

```json
POST /logs_2014-09-30/_settings
{ "number_of_replicas": 0 }

POST /logs_2014-09-30/_optimize?max_num_segments=1

POST /logs_2014-09-30/_settings
{ "number_of_replicas": 1 }
```

关闭旧索引

```json
POST /logs_2014-01-*/_flush 
POST /logs_2014-01-*/_close 
POST /logs_2014-01-*/_open 
```