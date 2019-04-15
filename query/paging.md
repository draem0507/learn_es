

## 引言

elasticsearch对分页查询，有三种实现方式：

- from+size：类似于关系型数据库的offset+limit
- scroll：在显示使用scroll请求后，es会返回scroll_id，下一次查询就只需要使用scroll_id进行查询即可
- searchAfter：类似于scroll，区别在于偏移量是由唯一排序字段指定

### from+size

基本语法

```java
POST /my_index/my_type/_search
{
    "query": { "match_all": {}},
    "from": 100,
    "size":  10
}
//默认from:0,size:10
```

每次分页查询，都需要获取的数量是shades*(from+size)；然后内存排序后，返回size条；当遇到长翻页场景时，获取的数据会很大，IO、CPU、网络开销也随着增加，请求性能随着下降。

更多请移步：[elasticsearch query+fetch+merge](http://lxwei.github.io/posts/%E4%BD%BF%E7%94%A8scroll%E5%AE%9E%E7%8E%B0Elasticsearch%E6%95%B0%E6%8D%AE%E9%81%8D%E5%8E%86%E5%92%8C%E6%B7%B1%E5%BA%A6%E5%88%86%E9%A1%B5.html)

### scroll

`scroll` 查询 可以用来对 Elasticsearch 有效地执行大批量的文档查询，而又不用付出深度分页那种代价。

游标查询允许我们 先做查询初始化，然后再批量地拉取结果。 这有点儿像传统数据库中的 *cursor* 。

游标查询会取某个时间点的快照数据。 查询初始化之后索引上的任何变化会被它忽略。 它通过保存旧的数据文件来实现这个特性，结果就像保留初始化时的索引 *视图* 一样。

深度分页的代价根源是结果集全局排序，如果去掉全局排序的特性的话查询结果的成本就会很低。 游标查询用字段 `_doc` 来排序。 这个指令让 Elasticsearch 仅仅从还有结果的分片返回下一批结果。

基本语法

```java
//first call
GET /old_index/_search?scroll=1m 
{
    "query": { "match_all": {}},
    "sort" : ["_doc"], 
    "size":  1000
}
//after the first call
GET /_search/scroll
{
    "scroll": "1m", 
    "scroll_id" : "cXVlcnlUaGVuRmV0Y2g7NTsxMDk5NDpkUmpiR2FjOFNhNnlCM1ZDMWpWYnRROzEwOTk1OmRSamJHYWM4U2E2eUIzVkMxalZidFE7MTA5OTM6ZFJqYkdhYzhTYTZ5QjNWQzFqVmJ0UTsxMTE5MDpBVUtwN2lxc1FLZV8yRGVjWlI2QUVBOzEwOTk2OmRSamJHYWM4U2E2eUIzVkMxalZidFE7MDs="
}
```

search after

类似于scroll,search afer通过制定排序唯一字段进行过滤。而与scroll的区别在于，search after可以实时获取最新的数据。



基本语法

```java
//first call
GET my_index/my_type/_search
{
  "query": { "match_all": {}},
  "size": 100,
  "sort": [
         {"user_id": "asc"}
       ]
}
//after the first
GET index_mdm_org/_search
{
  "query": { "match_all": {}},
  "search_after": [
    150380   
  ],
  "size": 100,
  "sort": [
         {"user_id": "asc"}
       ]
}
//150380 是上一次请求返回数据集合中，最后一条记录的user_id
//eg: query_set[99].user_id is the next search_afer value
```



## 参考

[elasticsearch官网之游标查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scroll.html#scroll)

[elastcisearch官网之searchafter](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-request-search-after.html)

[Elasticserach search after 源码分析](https://www.jianshu.com/p/91d03b16af77)

