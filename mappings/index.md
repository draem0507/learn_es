1 整体结构
"mappings": {
    "<typeName>": {
        "<meta>": {
            ...
        },
        "<meta>": {
            ...
        },
        "properties": {
            "<fieldName>": {
                "type": "<type>",
                "<parameter>": ...
            },
            ...
        }
    }
}


2 Meta


详细的Meta说明，请参考ElasticSearch Reference > Mapping > Meta-Fields官网。



此处仅简单介绍几个比较常用的Meta字段。



(1)_all
"_all": {
    "enabled": true / false (default)
}
若激活该功能，ES会将所有字段值拼接在一起，以空格分隔，保存在一个大的字符串中。

在查询时，可以通过_all来进行全字段匹配，而不需要指定具体字段，例如：

"query": {
    "match": {
        "_all": "..."
    }
}


(2)_source
"_source": {
    "enabled": true (default) / false
}
指定是否保存用户传递的原始JSON文档数据，在查询时，可返回相应的数据。



3 Field Type


详细的类型说明，请参考ElasticSearch Reference > Mapping > Field Data Type官网。



4 Field Parameter


此处仅介绍store、doc_values和analyzer参数。



(1)store
"store": true / false (default)
指明该字段值是否被单独存储在ES中，从而，可以独立于_source存储，进行单独获取。

单独存储的字段，可以在查询时，通过以下方式指定获取：

"stored_fields": ["<fieldname>","<fieldname>",...]


(2)doc_values
"doc_values": true / false
指明该字段值是否以列的形式存储在document级，从而，有助于sort、aggregations或scripting操作。

对于不同类型字段，默认值不同。



(3)analyzer
对于text类型的字段，可以指定analyzer参数，指明用哪种方式对文本值进行分析及拆分。

在查询过程中，也可以指定analyzer参数，指明用哪种方式对查询条件值进行分析及拆分。

当文本内容被拆分成小文本数组，在查询匹配时，即可以根据小文本匹配数量，得到查询条件的匹配程度。
