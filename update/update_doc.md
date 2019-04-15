

### 局部更新

```java
//修改文档  （这种采用的是覆盖的方式）   相当于sql中的 update job set title=?,salary_min=?city=? (....省略...) where id = 1
PUT lagou/job/1
{
"title":"linux运维工程师",
"salary_min":10000,
"city":"广州",
"company":{
"name":"腾讯",
"company_addr":"广州软件园"
},
"publish_date":"2017-4-12",
"comments":21
}

// update job set comments = 20 where id = 1 
POST lagou/job/1/_update
{
  "doc":{
    "comments":20
  }
}
```

