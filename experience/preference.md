### 引言

elasticsearch可以使用preference参数来指定分片查询的优先级，使用时就是在请求url上加上preference参数，如：http://ip:host/index/_search?preference=_primary 
java的调用接口翻译为：client.prepareSearch(“index”).setPreference(“_primary”)。

### es有5种查询优先级： 

##### _primary: 

指查询只在主分片中查询 

##### _primary_first: 

指查询会先在主分片中查询，如果主分片找不到（挂了），就会在副本中查询。 

##### _local: 

指查询操作会优先在本地节点有的分片中查询，没有的话再在其它节点查询。 

##### _only_node:_

_指在指定id的节点里面进行查询，如果该节点只有要查询索引的部分分片，就只在这部分分片中查找，所以查询结果可能不完整。如_only_node:123在节点id为123的节点中查询。 

##### Custom (string) value:

用户自定义值，指在参数cluster.routing.allocation.awareness.attributes指定的值，如这个值设置为了zone，那么preference=zone的话就在awareness.attributes=zone*这样的节点搜索，如zone1、zone2。关于这个值作用可以参考下面文章。 

### 自定义PlainOperationRouting



```java
if(preference.startsWith("_only_nodes:"))  {  
return indexShard.onlyNodesActiveShardsIt(preference.substring("_only_nodes:".length()));  
}  
```



```

/** 
* Prefers execution on the provided nodes if applicable. 
 */  
public ShardIterator onlyNodesActiveShardsIt(String nodeIds) {  
String[] ids = nodeIds.split(",");  
ArrayList<ShardRouting> ordered = **new** ArrayList<ShardRouting>(shards.size());  
// fill it in a randomized fashion  
for (int i = 0; i < shards.size(); i++) {  
ShardRouting shardRouting = shards.get(i);  
for(String nodeId:ids){  
if(nodeId.equals(shardRouting.currentNodeId())) {  
 ordered.add(shardRouting);  
 }  
}  
 }  
return new PlainShardIterator(shardId, ordered);  
}  
```



### 参考

[elasticsearch的五种分片查询优先级](https://www.cnblogs.com/huangpeng1990/p/4364347.html)

[About preference](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/search-request-preference.html#search-request-preference)