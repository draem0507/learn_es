##elastic查询语法

### 基本语法

```java
GET /index_china/fulltext/_search
{
　　  "query": {
      	"match_all": {}
      },
      "from": 10,
 　　 "size": 10,
     "_source:"["name","age"],
       "sort": "age" //基于score进行排序
　　}'
     //size默认为10；from为0,_source 返回的字段过滤
    //等同于 //GET test_index/test_type/_search?sort=post_date:desc&q=search
```

基本匹配查询

```java
GET /employee/_search?q=John'
//或则
GET /employee/_search{
    "query": {
        "multi_match" : {
            "query" : "John",
            "fields" : ["_all"]//or Appoint clo["name","desc"]
        }
    }
}'



```

###排序（默认根据query匹配的分数进行排序）

```java
GET/index_china/fulltext/_search
{
"query": {
	"match_all": {}
 },
"sort":[ 
	{"age":{"order":"asc"}},
	{"_score":{"order":"desc"}}
	]
}
```

### Boosting

```java
GET/index_china/fulltext/_search
{
    "query": {
        "multi_match" : {
            "query" : "rock",
            "fields": ["about", "interests^3"]//interests权重*3
        }
    }
}

```

### Bool Query

Bool查询中的关键字有：

- must
  所有的语句都 必须（must） 匹配，与 AND 等价。
- must_not
  所有的语句都 不能（must not） 匹配，与 NOT 等价。
- should
  至少有一个语句要匹配，与 OR 等价。

```java
//返回name中包含"海"或"花"的所有document

GET /school/_search
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
  
  
//返回name中包含"海"和"哥"的所有账户(AND)

GET /school/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "海" } },
        { "match": { "name": "哥" } }
      ]
    }
  }
  
//返回name中包含"海",且地址不是"山东烟台"的所有document

GET /school/_search
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
//查询某个为空的字段
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "字段名"
        }
      }
    }
  }
}


```

### range查询

```java
//基础用法
"query": {
        "range" : {
            "birthday": {
                "gte": "2017-02-01",
                "lte": "2017-05-01"
            }
        }
    }

//在所有document中寻找age在0-25岁之间(闭区间)的学生

GET /school/_search
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

### Term查询

```java
//
"query": {
        "term" : {
            "interests": "music"
        }
    }
//多个
{
    "query": {
        "terms" : {
            "publisher": ["oreilly", "packt"]
        }
    }
}

//返回name中包含term "海" 或 "花" 的所有document

GET /school/_search
{
  "query": { "match": { "name": "海 花" } 
 }

//匹配phrase "海 花"

GET /school/_search
{
 "query": { "match_phrase": { "name": "海 花" }}
```

### Filtered查询

过滤查询允许我们对查询结果进行筛选

```java
//查询about和interests中包含music关键字的员工，同时过滤出birthday大于2017/02/01的结果 
"query": {
        "filtered": {
            "query" : {
                "multi_match": {
                    "query": "music",
                    "fields": ["about","interests"]
                }
            },
            "filter": {
                "range" : {
                    "birthday": {
                        "gte": 2017-02-01
                    }
                }
            }
        }
    }
```



### 查询为空的字段

```java
//in db like this:select eid,ent_name from ent_search where enttype_code is NULL;
//in es,wo can use that:
 GET ent_search/_search
{
  "_source": ["eid","ent_name"], 
    "query": {
        "bool": {
            "must_not": {
                "exists": {
                    "field": "enttype_code"
                }
            }
        }
    }
}  
```

## 高级查询

### 聚合查询

```java
{

  　　"size": 0,

 　　 "aggs":{

   　　 "group_by_state": {

   　　  "terms": {

     　　　   "field": "state"

   　　  }

   　　 }

  　　}

　　}
//等同于 　　SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC
```


　　

```java
{

  　　"size": 0,

  　　"aggs": {

   　　 "group_by_state": {

  　　  "terms": {

      　　 "field": "state"

     　　 },

   　　  "aggs": {

      　　 "average_balance": {

       　　  "avg": {

        　　   "field": "balance"

         　　 }

      　　 }

     　　 }

   　　}

  　　}

　　}
//按照state分组，降序排序，返回balance的平均值：
```



### 滚动查询

## 小结

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







### 谈论query和filter的效率

   一般认为filter的速度快于query的速度 
   \- filter不会计算相关度得分，效率高 
   \- filter的结果可以缓存到内存中，方便再用

```java
//找性别是女，所在的州是PA，过滤条件是年龄是39岁，balance大于等于10000的文档
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "gender": "F"
          }
        },
        {
          "match": {
            "state": "PA"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "age": "39"
          }
        },
        {
          "range": {
            "balance": {
              "gte": "10000"
            }
          }
        }
      ]
    }
  }
}
```

```java
GET /index_mdm_his_emp_v1/_search
{
  "query": {
    "bool": {
      "must": {
        "term": {
          "snapshot": {
            "value": "20190106"
          }
        }
      },
      "filter": {
        "range": {
          "join_date": {
            "lt": "2018-08-01",
            "gte": "2018-03-01"
          }
        }
      }
    }
  }
}
```

```java
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }}, 
        { "match": { "content": "Elasticsearch" }}  
      ],
      "filter": [ 
        { "term":  { "status": "published" }}, 
        { "range": { "publish_date": { "gte": "2015-01-01" }}} 
      ]
    }
  }

}
```

## 参考

1.[Elasticsearch 常用基本语法](https://www.cnblogs.com/sunfie/p/6653778.html)

2.