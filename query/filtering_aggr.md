# 过滤查询以及聚合



A natural extension to aggregation scoping is filtering. Because the aggregation operates in the context of the query scope, any filter applied to the query will also apply to the aggregation. 过滤是聚合作用域的一个很自然的扩展。因为聚合工作在查询作用域的上下文中，那么适用于查询的任何过滤器也同样能够适用于聚合。





## filtered查询



如果你想要找到所有售价高于10000美刀的车，同时也对这些车计算其平均价格，那么可以使用一个filtered查询：

```
GET /cars/transactions/_search?search_type=count
{
    "query" : {
        "filtered": {
            "filter": {
                "range": {
                    "price": {
                        "gte": 10000
                    }
                }
            }
        }
    },
    "aggs" : {
        "single_avg_price": {
            "avg" : { "field" : "price" }
        }
    }
}
```

从本质上而言，使用filtered查询和使用match查询并无区别，正如我们在上一章所讨论的那样。该查询(包含了一个过滤器)返回文档的一个特定子集，然后聚合工作在该子集上。





## 过滤桶(Filter Bucket)



如果你只想过滤聚合结果呢？假设我们正在创建针对汽车交易的搜索页面，我们想要根据用户搜索内容来展示对应结果。但是我们也想通过包含上个月出售的汽车的平均价格(匹配搜索的汽车)来让页面更加丰富。

此时我们不能使用简单的作用域，因为有两个不同搜索条件。搜索结果必须要匹配ford，但是聚合结果必须要匹配ford以及售出时间为上个月。

为了解决这一问题，我们使用一个名为filter的特殊桶。通过制定一个过滤器，当文档匹配了该过滤器的规则时，它就会被添加到桶中。

以下是得到的查询：

```
GET /cars/transactions/_search?search_type=count
{
   "query":{
      "match": {
         "make": "ford"
      }
   },
   "aggs":{
      "recent_sales": {
         "filter": { 
            "range": {
               "sold": {
                  "from": "now-1M"
               }
            }
         },
         "aggs": {
            "average_price":{
               "avg": {
                  "field": "price" 
               }
            }
         }
      }
   }
}
```

因为过滤器桶和任何其它桶以相似的方式工作，你可以任意地将其它桶和指标包含在其中。所有的嵌套组建都会"继承"该过滤器。从而使你能够根据需要对聚合中的内容进行过滤。





## 后置过滤器(Post Filter)



目前，我们有了用于过滤搜索结果和聚合的过滤器(filtered查询)，也有了用于过滤聚合中某一部分的过滤器(filter桶)。

你也许会好奇，“是否有一种过滤器只过滤搜索结果，而不过滤聚合呢？”这个问题的答案就是使用post_filter。

它是搜索请求内能够接受一个过滤器作为参数的顶层元素。该过滤器会在查询执行完毕后生效(后置因此得名：在查询执行之后运行)。正因为它在查询执行后才会运行，所以它并不会影响查询作用域 - 因此就不会对聚合有所影响。

我们可以利用这一行为在搜索条件中添加额外的过滤器，而不影响用户界面中类似于类别分面(Categorical Facets)的元素。让我们设计另一个针对汽车交易的搜索页面。该页面允许用户对汽车进行搜索，同时还能够根据颜色进行过滤。颜色通过聚合提供：

```
GET /cars/transactions/_search?search_type=count
{
    "query": {
        "match": {
            "make": "ford"
        }
    },
    "post_filter": {    
        "term" : {
            "color" : "green"
        }
    },
    "aggs" : {
        "all_colors": {
            "terms" : { "field" : "color" }
        }
    }
}
```

post_filter元素是一个顶层元素，只会对搜索结果进行过滤。

查询部分呢用来找到所有ford汽车。然后我们根据一个terms聚合来得到颜色列表。因为聚合是在查询作用域中进行的，得到的颜色列表会反映出ford汽车的各种颜色。

最后，post_filter会对搜索结果进行过滤，只显示绿色的ford汽车。这一步发生在执行查询之后，因此聚合是不会被影响的。

这一点对于维持一致的用户界面而言是非常重要的。假设一个用户在界面上点击了一个分类(比如，绿色)。期望的结果是搜索结果被过滤了，而用户界面上的分类选项是不会变化的。如果你使用了一个filtered查询，用户界面上也立即会对分类进行更新，此时绿色就变成了唯一的选项 - 这显然不是用户想要的！

> 警告：性能考量
>
> 只有当你需要对搜索结果和聚合使用不同的过滤方式时才考虑使用post_filter。有时一些用户会直接在常规搜索中使用post_filter。
>
> 不要这样做！post_filter会在查询之后才会被执行，因此会失去过滤在性能上帮助(比如缓存)。
>
> post_filter应该只和聚合一起使用，并且仅当你使用了不同的过滤条件时。





## 总结



选择合适类型的过滤 - 搜索结果(Search Hits)，聚合(Aggregations)，或两者 - 通常都取决于你的用户界面的行为。过滤器的选择(或者组合)取决于你想要如何向用户展示结果数据。

- A filtered query affects both search results and aggregations.filtered查询会影响搜索结果和聚合。
- filter桶只影响聚合。
- post_filter只影响搜索结果。