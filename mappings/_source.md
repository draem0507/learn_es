## 一、_source是什么

_source field是我们在PUT数据时候的**json body**：

```
PUT store_index/my_type/1
{
  "title":   "Some short title",
  "date":    "2015-01-01",
  "content": "A very long content field..."
}
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "store_index",
        "_type": "my_type",
        "_id": "1",
        "_score": 1,
        "_source": {
          "title": "Some short title",
          "date": "2015-01-01",
          "content": "A very long content field..."
        }
      }
    ]
  }
}
```

## 二、_source disable的方法

```
PUT tweets
{
  "mappings": {
    "tweet": {
      "_source": {
        "enabled": false
      }
    }
  }
}
```

## 三、disable掉_source会怎样

- 消极影响：（总的来说后果比较严重最好不要disable）
  1. **update**、**update_by_query**、**reindex** API将不再可用
  2. **highlighting**将会受影响（？）
  3. elasticsearch索引的reindex、修改mapping、改变分词、更新索引等都会不可用
  4. 通过查看原始文档来调试查询或者聚合的功能将不可用
  5. elasticsearch失去自动修复index损坏的能力
- 积极影响:节省磁盘空间