##elastic查询语法
###相关性，query方式；基于score进行排序
```java
"sort": [
  {
    "post_date": { "order": "desc"     }
  }
]

//也可以使用这种方式
//GET test_index/test_type/_search?sort=post_date:desc&q=search
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
```java
 2.4 返回name中包含term "海" 或 "花" 的所有document

POST /school/_search
{
  "query": { "match": { "name": "海 花" } }


2.5 匹配phrase "海 花"

POST /school/_search
{
 "query": { "match_phrase": { "name": "海 花" }}



2.7 返回name中包含"海"或"花"的所有document

POST /school/_search
{
  "query": {
   "bool": {
     "should": [
      { "match": { "name": "海" } 		},
        { "match": { "name": "花" 		} 
      }
    ]   
    }
  }
  
  
   2.6 返回name中包含"海"和"哥"的所有账户(AND)

POST /school/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "海" } },
        { "match": { "name": "哥" } }
      ]
    }
  }
  
  2.9 返回name中包含"海",且地址不是"山东烟台"的所有document

POST /school/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "海" } }
      ],
      "must_not": [
        { "match": { "address": "山东烟台" } }
      ]
    }
  }
}

3.1 在所有document中寻找age在0-25岁之间(闭区间)的学生


POST /school/_search
{
  "query": {
    "filtered": {
      "query": { "match_all": {} },
      "filter": {
        "range": {
          "age": {
            "gte": 0,
            "lte": 25
          }
        }
      }
    }
  }
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



### 滚动查询

```json
GET index_employee/_search?scroll=1m
{
 "size" :1000,
 "query": {

   "term": {
    "name": {
       "value": "liu"
     }
     
   }
   
 },
 "sort": [
   {
     "emp_id": {
       "order": "desc"
     }
   },
   {
     "tenant_id": {
       "order": "asc"
    }
   }
 ]
}
```







**谈论query和filter的效率**

   一般认为filter的速度快于query的速度 
   \- filter不会计算相关度得分，效率高 
   \- filter的结果可以缓存到内存中，方便再用

