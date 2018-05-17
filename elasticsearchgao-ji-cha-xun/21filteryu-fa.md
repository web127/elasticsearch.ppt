# filter 语法

为什么filter会快？
query的时候，会先比较查询条件，然后计算分值，最后返回文档结果
而filter则是先判断是否满足查询条件，如果不满足，会缓存查询过程（记录该文档不满足结果）；满足的话，就直接缓存结果。
综上所述，filter快在两个方面：

1 对结果进行缓存
2 避免计算分值

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
