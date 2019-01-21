## 一、_all 是什么

在Elasticsearch中，_all field维护这一个很大的字符串数组(text类型)。这个字符串是其他字段先经过分词，然后组合在一起形成的。**组合方式**是以空格为分隔符、无序。

官网有个例子

```
PUT my_index/user/1 
{
  "first_name":    "John",
  "last_name":     "Smith",
  "date_of_birth": "1970-10-24"
}
GET my_index/_search
{
  "query": {
    "match": {
      "_all": "john smith new york"
    }
  }
}
```

PUT进去三个field的数据，此时_all维护的字符串数组如下： **[ "john", "smith", "1970", "10", "24" ]**

## 二、_all 作用是什么

允许在不知道**field**的情况下，进行search

> 有些情况下你根本不知道你搜的value属于哪个field。上文脚本中直接用_all代替了

> _all field是一个text类型，并且同其他的普通字符串field一样能接受如下参数来修饰：**analyzer**、**term_vectors**、**index_options**、**store**

你可以通过如下命令来disable掉_all field

```
PUT my_index
{
  "mappings": {
    "type_1": { 
      "properties": {...}
    },
    "type_2": { 
      "_all": {
        "enabled": false
      },
      "properties": {...}
    }
  }
}
```

## 三、disable掉_all会怎样

- 消极影响：无法在不知field的情况下进行一些value搜索等操作
- 积极影响：减轻CPU负担，减少磁盘占用