1.修改**elasticsearch.yml** 

​	**#cluster.name: elasticsearch**

​	**#node.name: "Franz Kafka"**

2.启动服务，验证

**curl -X GET 'http://192.168.1.131:9200/?preey'**

**curl -X GET 'http://192.168.1.133:9200/_cat/nodes'**

3.查看主节点

**curl -X GET 'http://192.168.1.133:9200/_cat/master?v'**

参考：http://blog.51cto.com/sihua/1888410

