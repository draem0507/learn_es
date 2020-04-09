```xml
//最小化拆分
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "中中华人民共和国人民大会堂"
}

//最大化拆分 中华人民共和国、人民大会堂
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国人民大会堂"
}
```

## 分词器实践

两种分词器使用的最佳实践是：索引时用ik_max_word，在搜索时用ik_smart。
即：索引时最大化的将文章内容分词，搜索时更精确的搜索到想要的结果。

```
PUT my_index_0307
{
  "mappings": {
    "my_type": {
      "properties": {
        "full_text": {
          "type":  "text" ,
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_smart"
        },
        "exact_value": {
          "type":  "text" 
        }
      }
    }
  }
}



PUT my_index_0307/my_type/1
{
  "full_text":   "手机", 
  "exact_value": "手机"  
}

PUT my_index_0307/my_type/2
{
  "full_text":   "手机壳", 
  "exact_value": "手机壳"  
}

PUT my_index_0307/my_type/3
{
  "full_text":   "中华人民共和国人民大会堂", 
  "exact_value": "中华人民共和国人民大会堂"  
}



GET my_index_0307/my_type/_search
{
  "query": {
    "match": {
      "full_text": "中华人民共和国"
    }
  }
}

GET my_index_0307/my_type/_search
{
  "query": {
    "match": {
      "exact_value": "中华人民共和国"
    }
  }
}
```





## 自定义词库

```shell
touch   elasticsearch/plugins/ik/config/custom/authTest.dic
vim   authTest.dic
# content for authTest.dic
#浦下村
#包爷
#手机壳
:wq
vim IKAnalyzer.cfg.xml
<entry key ="ext_dict">custom/authTest.dic</entry>
# restart es
```







## 参考

https://blog.csdn.net/ldllovegyh/article/details/82820653

https://www.jianshu.com/p/f178e59ffaf2

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-dict-decomp-tokenfilter.html#analysis-dict-decomp-tokenfilter