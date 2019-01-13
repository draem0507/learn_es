#添加索引
PUT lagou
{
  "settings": {
    "index": {
      "number_of_shards":5,
      "number_of_replicas":1
    }
  }
}
#查看 索引设置
GET lagou/_settings
GET _all/_settings
GET .kibana,lagou/_settings
GET _settings


#修改settings
PUT lagou/_settings
{
  "number_of_replicas":2
}

#获取索引信息
GET _all
GET lagou

#保存文档
PUT lagou/job/1
{
  "title":"python分布式爬虫开发",
  "salary_min":15000,
  "city":"北京",
  "company":{
    "name":"百度",
    "company_addr":"北京市软件园"
  },
  "publish_date":"2017-4-16",
  "comments":15
}
#通过post方式不添加ID，去添加文档
POST lagou/job/
{
  "title":"python django 开发工程师",
  "salary_min":30000,
  "city":"上海",
  "company":{
    "name":"美团",
    "company_addr":"北京市软件A园"
  },
  "publish_date":"2017-4-16",
  "comments":20
}


#查看文档内容
GET lagou/job/1
GET lagou/job/1/?_source=title
GET lagou/job/1/?_source=title,city
GET lagou/job/1/?_source



#修改文档
PUT lagou/job/1
{
  "title":"python分布式爬虫开发",
  "salary_min":15000,
  "company":{
    "name":"百度",
    "company_addr":"北京市软件园"
  },
  "publish_date":"2017-4-16",
  "comments":15
}


#更新文档内容
POST lagou/job/1/_update
{
"doc": {
    "comments":20
 }
}


#删除
DELETE lagou/job/1
DELETE lagou

#_delete_by_query

_delete_by_query会删除所有query语句匹配上的文档，用法如下：

```java
curl -X POST "localhost:9200/twitter/_delete_by_query" -H 'Content-Type: application/json' -d' {   "query": {      "match": {       "name": "测试删除"    
}   
}
}
//返回数据格式，告诉你用时和删除多少数据等
{
  "took" : 147,
  "timed_out": false,
  "deleted": 119,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 119,
  "failures" : [ ]
}
//一次删除多个索引（即：数据库）中的多个类型（即表）中的数据，也是可以的。例如：
curl -X POST "localhost:9200/twitter,blog/_docs,post/_delete_by_query" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}
'

//参考：https://www.jianshu.com/p/60a6ad164035


```

