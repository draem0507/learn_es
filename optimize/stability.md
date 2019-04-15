1. Elasticsearch最好运行在per node per box. 多node共享一个box会造成内存竞争, 而且难查错 (到底是哪个node crazy了呢?). 不推荐.
2. 大数据下, 请考虑使用Master, Data, Client 3类型node模式. 一般是3~5 master nodes, N data nodes, 3 client node. bulk索引时, 请求发给data nodes. 搜索时, 请求发给client node.
3. 我曾做过跟你类似的数据量, 1TB+, 分布在3 master nodes, 10 data nodes共13个box, 运行在AWS, 每box有60G内存, 其中30G分配给elasticsearch. bulk索引时未见性能下降. filter及aggs搜索时间依然在100ms左右.
4. 0副本节省空间, 但有损于搜索, 建议至少1 replica.
5. 磁盘建议使用SSD, 但是如果只能用spinning, 请用最快速的 (比如7200或10k转)
6. 建议定时抓取node status, 写入graphite, 转化为图表来分析病症, 做优化, 做对比. (Github的全code搜索ES cluster就是这么干的)

reference

<https://elasticsearch.cn/question/84>