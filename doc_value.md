## doc_value是什么

1. 绝大多数的fields在默认情况下是indexed，因此字段数据是可被搜索的。倒排索引中按照一定顺序存放着terms供搜索，当命中搜索时，返回包含term的document。

   > terms 中包含很多term

2. 当Sorting、aggregations、scripts access to field这三种情况的时候，我们需要另外的data access模式。这种模式和上述在terms中寻找term并且返回document是不同的

3. Doc values 为on-disk 数据结构，在document索引时被创建。Doc values 存放的values和 _source这个[meta-Fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-fields.html)是一致的。支持除了analyzed string 以外的所有类型。

## 二、doc_value特性

1. doc_value 默认情况下是enable的。
2. column-oriented 存放field，以便sort、aggregate、access the field from a script
3. disable doc_value:

```
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "mystring": { 
          "type":       "keyword",
          "doc_values": false
        }
      }
    }
  }
}
```

## 三、disable doc_value会怎样

- 消极影响：**sort、aggregate、access the field from script将会无法使用**
- 积极影响：**节省磁盘空间**