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
#无法删除表
