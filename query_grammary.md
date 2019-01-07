##elastic查询语法
###相关性，query方式；基于score进行排序
```java
  "sort": [
  {
    "post_date": { "order": "desc"     }
  }
]
```
###排序
```java
GET /index_china/fulltext/_search
{
  "query": {
    "match": {   "name": "小张" } 
      },
  "sort": "age"
}
```
###多级排序
```java
 GET /index_china/fulltext/_search
{
"query": {
"match_all": {}
},
"sort":[ {"age":{"order":"asc"}},
{"_score":{"order":"desc"}}]
}
```
###保存完整的字符串作为一个词条，同时又分词索引
```java
//在ES中为实现该功能，ES提供了 multi-field mapping 机制，配置通过配置即可实现字段即分词，又可排序
 "title": {  
 "type": "string",
 "analyzer": "ansj",
 "fields": {
     "sort": { 
         "type": "string",
         "index": "not_analyzed"
     }
  }
}
//通过title字段作为搜索，而用title.sort字段作为排序： 
 GET /_search
 {
     "query": {
         "match": {
             "title": "elasticsearch"
         }
     },
     "sort": "title.sort"
 }   
```



