# filter 语法
查询DSL有2种：查询DSL（query DSL）和过滤DSL（filter DSL）
为什么filter会快？

>查询在Query查询上下文和Filter过滤器上下文中，执行的操作是不一样的：

>Query查询上下文：

>在查询上下文中，查询会回答这个问题——“这个文档匹不匹配这个查询，它的相关度高么？”
如何验证匹配很好理解，如何计算相关度呢？之前说过，ES中索引的数据都会存储一个_score分值，分值越高就代表越匹配。另外关于某个搜索的分值计算还是很复杂的，因此也需要一定的时间。
查询上下文 是在 使用query进行查询时的执行环境，比如使用search的时候。

>Filter过滤器上下文：
在过滤器上下文中，查询会回答这个问题——“这个文档匹不匹配？”
答案很简单，是或者不是。它不会去计算任何分值，也不会关心返回的排序问题，因此效率会高一点。
过滤上下文 是在使用filter参数时候的执行环境，比如在bool查询中使用Must_not或者filter。

另外，经常使用过滤器，ES会自动的缓存过滤器的内容，这对于查询来说，会提高很多性能。


#### range 过滤
>range过滤允许我们按照指定范围查找一批数据
gt :: 大于
gte:: 大于等于
lt :: 小于
lte:: 小于等于

```
POST /order_index/order_type/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "totalPrice": {
            "gt": 20,
            "lt": 80
          }
        }
      }
    }
  }
}
```

#### term 过滤
>term主要用于精确匹配哪些值，比如数字，日期，布尔值或 not_analyzed 的字符串(未经分词的文本数据类型)

```
POST /order_index/order_type/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "totalPrice": 100
        }
      }
    }
  }
}
```


#### terms 过滤
>terms 跟 term 有点类似，但 terms 允许指定多个匹配条件。 如果某个字段指定了多个值，那么文档需要一起去做匹配

```
POST /order_index/order_type/_search
{
  "query": {
    "bool": {
      "filter": {
        "terms": {
          "totalPrice": [
            100,
            80
          ]
        }
      }
    }
  }
}
```
