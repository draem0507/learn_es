## 前言

索引创建后，你可以在索引当中添加新的类型，在类型中添加新的字段。但是如果想修改已存在字段的属性（修改分词器、类型等），目前ES是做不到的。如果确实存在类似这样的需求，只能通过重建索引的方式来实现。但想要重建索引，请保证索引_source属性值为true

## 步骤

备注：index_source 为现有索引；index_dest为目标索引

### 设置名别

```java
PUT /index_source/_alias/index_source_alias
```

### 重建索引

```java
POST _reindex
{
  "source": {
    "index": "index_source",
    "size":1000 //表示通过scoll方式获取数据
  },
  "dest": {
    "index": "index_dest"
  }
}
```

### 切换别名

```java
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "_index_source",
        "alias": "index_source_alias"
      }
    },
    {
      "add": {
        "index": "index_dest",
        "alias": "index_source_alias"
      }
    }
  ]
}
```



### 补充

可能存在部分数据调整，可通过update_by_query来实现

```java
POST index_desc/update_by_query
{
 "script": {
   "lang": "painless",
   "inline": "ctx._source.status=1" //更新status为1
 }
}
```

## 参考

[elastic redinx](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html)

