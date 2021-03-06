



### 字段属性说明

```java


"status": {
	"type":  "string", //字符串类型
	"index": "analyzed"//分词，不分词是：not_analyzed ，设置成no，字段将不会被索引
	"analyzer":"ik"//指定分词器
	"boost":1.23//字段级别的分数加权
	"doc_values":false//对not_analyzed字段，默认都是开启，分词字段不能使用，对排序和聚合能提升较大性能，节约内存
	"fielddata":{"format":"disabled"}//针对分词字段，参与排序或聚合时能提高性能，不分词字段统一建议使用doc_value
	"fields":{"raw":{"type":"string","index":"not_analyzed"}} //可以对一个字段提供多种索引模式，同一个字段的值，一个分词，一个不分词
	"ignore_above":100 //超过100个字符的文本，将会被忽略，不被索引
	"include_in_all":ture//设置是否此字段包含在_all字段中，默认是true，除非index设置成no选项
	"index_options":"docs"//4个可选参数docs（索引文档号） ,freqs（文档号+词频），positions（文档号+词频+位置，通常用来距离查询），offsets（文档号+词频+位置+偏移量，通常被使用在高亮字段）分词字段默认是position，其他的默认是docs
	"norms":{"enable":true,"loading":"lazy"}//分词字段默认配置，不分词字段：默认{"enable":false}，存储长度因子和索引时boost，建议对需要参与评分字段使用 ，会额外增加内存消耗量
	"null_value":"NULL"//设置一些缺失字段的初始化值，只有string可以使用，分词字段的null值也会被分词
	"position_increament_gap":0//影响距离查询或近似查询，可以设置在多值字段的数据上火分词字段上，查询时可指定slop间隔，默认值是100
	"store":false//是否单独设置此字段的是否存储而从_source字段中分离，默认是false，只能搜索，不能获取值
	"search_analyzer":"ik"//设置搜索时的分词器，默认跟ananlyzer是一致的，比如index时用standard+ngram，搜索时用standard用来完成自动提示功能
	"similarity":"BM25"//默认是TF/IDF算法，指定一个字段评分策略，仅仅对字符串型和分词类型有效
	"term_vector":"no"//默认不存储向量信息，支持参数yes（term存储），with_positions（term+位置）,with_offsets（term+偏移量），with_positions_offsets(term+位置+偏移量) 对快速高亮fast vector highlighter能提升性能，但开启又会加大索引体积，不适合大数据量用

}
```

#### string类型

 text,keyword(string类型在es5已经被弃用)

1. ES默认的分词器为standard， 想要改变默认分词器为ik分词器， 可以在config/elasticsearch.yml文件中添加如下配置即可： index.analysis.analyzer.default.type:ik
2. **验证en_name分词的效果**：get index_his_emp_v1/_analyze?field=en_name&text=lewis



#### 数字类型：

long：64位存储 
integer：32位存储 
short：16位存储 
byte：8位存储 
double：64位双精度存储 
float：32位单精度存储

支持参数：

> coerce：true/false 如果数据不是干净的，将自动会将字符串转成合适的数字类型，字符串会被强转成数字，浮点型会被转成整形，经纬度会被转换为标准类型 
> boost：索引时加权因子 
> doc_value：是否开启doc_value 
> ignore_malformed：false（错误的数字类型会报异常）true（将会忽略） 
> include_in_all：是否包含在_all字段中 
> index:not_analyzed默认不分词 
> null_value：默认替代的数字值 
> precision_step：16 额外存储对应的term，用来加快数值类型在执行范围查询时的性能，索引体积相对变大 
>
> store：是否存储具体的值



#### 日期类型： date

#### bool类型： boolean

#### binary类型： binary　　

#### 复杂类型

数组类型：没有明显的字段类型设置，任何一个字段的值，都可以被添加0个到多个，要求，他们的类型必须一致： 
对象类型（object）：存储类似json具有层级的数据 
嵌套类型(nested)：支持数组类型的对象Aarray[Object]，可层层嵌套



#### 地理类型

1. geo-point类型： 支持经纬度存储和距离范围检索 
2. geo-shape类型：支持任意图形范围的检索，例如矩形和平面多边形

#### 专用类型 

1. ipv4类型：用来存储IP地址，es内部会转换成long存储 
2. completion类型：使用fst有限状态机来提供suggest前缀查询功能 
3. token_count类型：提供token级别的计数功能 
4. mapper-murmur3类型：安装sudo bin/plugin install mapper-size插件，可支持_size统计_source数据的大小 

附件类型：需要 https://github.com/elastic/elasticsearch-mapper-attachments开源es插件支持，可存储office，html等类型

#### 多值字段 

一个字段的值，可以通过多种分词器存储，使用fields参数，支持大多数es数据类型

```java
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
//或则
  "created": {
                "type":   "multi_field",
                "fields": {
                    "created": { "type": "string" },
                    "date":    { "type": "date"   }
                }
            }


```

#### 别名的操作的操作

```java
POST localhost:9200/_aliases -d '
{
    "actions": [
       { "remove": {
           "alias": "my_index",
           "index": "my_index_v1"
        }},
       { "add": {
            "alias": "my_index",
           "index": "my_index_v2"
       }}
   ]

}
```



### mapping 限制的type

![](/Users/lwx/Downloads/1120422-20171129202455761-838163003.png)



### 分词器

```java
get /website/blogs/_anazyze?field=title&text=welcom to china

get /_analyze?analyzer=standar&text=Text to analyze
```

###  字段

#### 保存完整的字符串作为一个词条，同时又分词索引

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

## 

 

