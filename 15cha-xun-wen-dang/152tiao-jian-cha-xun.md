## 条件查询

了解以下查询关键词：
match_all 
match 
multi_match 
bool 和must、must_not、should
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

```
POST /order_index/order_type/_search
{
  "query": {
    "match": {
      "address": "鼓楼"
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
#### bool 查询和bool 过滤

>bool 过滤可以直接给出是否匹配成功，而bool 查询要计算每一个查询子句的 _score （相关性分值）。
>bool 过滤可以用来合并多个过滤条件查询结果的布尔逻辑，它包含一下操作符：
must :: 多个查询条件的完全匹配,相当于 and。 
must_not :: 多个查询条件的相反匹配，相当于 not。 
should :: 至少有一个查询条件匹配, 相当于 or。 
