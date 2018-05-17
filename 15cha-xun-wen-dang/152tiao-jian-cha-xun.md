## 基础查询

了解以下查询关键词：
match_all 
match 
multi_match 
match_phrase
range
term和terms
bool和must、must_not、should
wildcards 
regexp 
prefix 
Phrase Matching

#### match_all 查询
>可以查询到所有文档，是没有查询条件下的默认语句。

```
POST /order_index/order_type/_search
{
  "query": {
    "match_all": {}
  }
}

```

#### match 查询
>match查询是一个标准查询，不管你需要全文本查询还是精确查询基本上都要用到它。
>但只能针对一个字段。
>如果你使用 match 查询一个全文本字段，它会在真正查询之前用分析器先分析match一下查询字符

```
POST /order_index/order_type/_search
{
  "query": {
    "match": {
      "address": "鼓楼区"
    }
  }
}
```
>如果用match下指定了一个确切值，在遇到数字，日期，布尔值或者not_analyzed 的字符串时，它将为你搜索你给定的值：
如：

```
{ "match": { "totalPrice": 26 }} 
{ "match": { "createTime": "2014-09-01" }} 
{ "match": { "public": true }} 
{ "match": { "userName": "Jane" }}
```
#### multi_match 查询
>multi_match查询允许你做match查询的基础上同时多个字段搜索同一个，相当于or查询

```
POST /order_index/order_type/_search
{
  "query": {
    "multi_match": {
      "query": "Jane",
      "fields": [
        "userName",
        "address"
      ]
    }
  }
}
```
#### match_phrase查询
>短语查询，采用match_phrase匹配，结果会非常严格
```
POST /order_index/order_type/_search
{
  "query": {
    "match_phrase": {
      "address": "鼓楼区"
    }
  }
}
```


#### range 查询
>range 过滤器， 让你可以根据范围过滤
gt :: 大于
gte:: 大于等于
lt :: 小于
lte:: 小于等于

```
POST /order_index/order_type/_search
{
"query": {
"range": {
"createTime": {
"gte": "2018-05-16 14:31:00"
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
"term": {
"totalPrice": 100
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
"terms": {
"totalPrice": [100,80]
}
}
}
```
#### bool 查询

>bool 过滤可以直接给出是否匹配成功，而bool 查询要计算每一个查询子句的 _score （相关性分值）。
>bool 过滤可以用来合并多个过滤条件查询结果的布尔逻辑，它包含一下操作符：
must :: 多个查询条件的完全匹配,相当于 and。 
must_not :: 多个查询条件的相反匹配，相当于 not。 
should :: 至少有一个查询条件匹配, 相当于 or。 

```
POST /order_index/order_type/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "address": "南京市"
        }
      },
      "must_not": {
        "match": {
          "totalPrice": "50"
        }
      },
      "should": [
        {
          "match": {
            "totalPrice": "80"
          }
        },
        {
          "range": {
            "createTime": {
              "gte": "2018-05-15 14:31:00"
            }
          }
        }
      ]
    }
  }
}
```

