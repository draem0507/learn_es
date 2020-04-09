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

#### 基本匹配查询

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

#### term&terms  精确（分词后的词根）

```java
 {
   "query":{
       "term":{
           "area":"GuangZhou"
       }
    }
 }

{
  "query": {
    "terms": {
      "commentContent": [
        "抖音",
        "锤子"
      ]
    }
  }
}
```

##### match   分词

```java
{
  "query": {
    "match": {
      "commentContent": "锤子"
    }
  }
}
```

#### match_phrase

文档同时满足下面两个条件才会被搜索到：

- （1）分词后所有词项都要出现在该字段中
- （2）字段中的词项顺序要一致

1. ```java
   
      {
     "query": {
      "match_phrase": {
       "content": "里皮恒大"
     	}
     }
     }
   
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
//对应字段中至少有一个非空值的文档
{
  "query": {
    "exists": {
      "field": "user"
    }
  }
}
```



### 聚合查询

#### max

```java
{
  "size": 0,
  "aggs": {
    "max_task_id": {
      "max": {
        "field": "taskId"
      }
    }
  }
}
//size不设置为0，除了返回聚合结果外，还会返回其它所有的数据。
/**
返回结果
 "aggregations": {
    "max_task_id": {
      "value": 454951685710676000
    }
  }
*/




```

　#### min

```
{
  "size": 0,
  "aggs": {
    "min_task_id": {
      "min": {
        "field": "taskId"
      }
    }
  }
}

```

#### avg

#### sum

同max

#### stats

返回各种统计数据

```
{
  "size": 0,
  "aggs": {
    "max_task_id": {
      "stats": {
        "field": "taskId"
      }
    }
  }
}
//结果
"aggregations": {
    "max_task_id": {
      "count": 15422,
      "min": 123,
      "max": 454951685710676000,
      "avg": 451931162244731800,
      "sum": 6.969682384138254e+21
    }
  }
```

#### group by

相当于MySQL的group by操作，所以不要尝试对es中text的字段进行桶聚合，否则会失败。

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
//在桶聚合的过程中还可以进行指标聚合，相当于mysql做group by之后，再做各种max、min、avg、sum、stats之类的： select avg(balance),state from xx group by state
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

Filter

```java
//相当于是MySQL根据where条件过滤出结果，然后再做各种max、min、avg、sum、stats操作。

{
  "size": 0,
  "aggs": {
    "gender_1_follower": {
      "filter": {
        "term": {
          "gender": 1
        }
      },
      "aggs": {
        "stats_follower": {
          "stats": {
            "field": "realFollowerCount"
          }
        }
      }
    }
  }
}
```

filters

```java
//在Filter的基础上，可以查询多个字段各自独立的各个指标，即对每个查询结果分别做指标聚合。
{
  "size": 0,
  "aggs": {
    "gender_1_2_follower": {
      "filters": {
        "filters": [
          {
            "term": {
              "gender": 1
            }
          },
          {
            "term": {
              "gender": 2
            }
          }
        ]
      },
      "aggs": {
        "stats_follower": {
          "stats": {
            "field": "realFollowerCount"
          }
        }
      }
    }
  }
}


```

range

```java
{
  "size": 0,
  "aggs": {
    "follower_ranges": {
      "range": {
        "field": "realFollowerCount",
        "ranges": [
          {
            "to": 500
          },
          {
            "from": 500,
            "to": 1000
          },
          {
            "from": 1000,
            "to": 1500
          },
          {
            "from": "1500",
            "to": 2000
          },
          {
            "from": 2000
          }
        ]
      }
    }
  }
}
```

#### Date Histogram Aggregation

```java
PUT my_blog
{
  "mappings": {
    "article":{
      "properties": {
        "title":{"type": "text"},
        "postdate":{
          "type": "date"
          , "format": "yyyy-MM-dd HH:mm:ss"
        }
      }
    }
  }
}
 
PUT my_blog/article/1
{
  "title":"Elasticsearch in Action",
  "postdate":"2014-09-23 23:34:12"
}

GET my_blog/article/_search
{
  "size": 0, 
  "aggs": {
    "agg_year": {
      "date_histogram": {
        "field": "postdate",
        "interval": "year",
        "order": {
          "_key": "asc"
        }
      }
    }
  }
}
// interval:month  day
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



## 查询删除

```java
POST infra_customer_comment_task_info/_delete_by_query
{
  "query": {
      	"match_all": {}
      }
}
```





## 参考

1.[Elasticsearch 常用基本语法](https://www.cnblogs.com/sunfie/p/6653778.html)

2.