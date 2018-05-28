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
>对所有文档查询，默认查询前10条数据，按_score降序排序

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
>短语查询，查询首先解析查询字符串来产生一个词条列表。然后会搜索所有的词条，采用match_phrase匹配，相比match结果会非常严格
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

#### term 查询
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


#### terms 查询
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

>Bool查询对应Lucene中的BooleanQuery，它由一个或者多个子句组成，每个子句都有特定的类型。

>must
返回的文档必须满足must子句的条件，并且参与计算分值

>filter
返回的文档必须满足filter子句的条件。但是不会像Must一样，参与计算分值

>should
返回的文档可能满足should子句的条件。在一个Bool查询中，如果没有must或者filter，有一个或者多个should子句，那么只要满足一个就可以返回。minimum_should_match参数定义了至少满足几个子句。

>must_not
返回的文档必须不满足must_not定义的条件。

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
#### wildcards
>使用标准的shell通配符查询

```
POST /order_index/order_type/_search
{
  "query": {
    "wildcard": {
      "address": "南?市*"
    }
  }
}
```

![](/assets/19.png)

#### regexp 查询
>正则表达式查询

```
POST /order_index/order_type/_search
{
  "query": {
    "regexp": {
      "orderNo": "a[0-9]+"
    }
  }
}
```
![](/assets/20.png)

### prefix 查询
>以什么字符开头的，可以更简单地用 prefix

```
POST /order_index/order_type/_search
{
  "query": {
    "prefix": {
      "address": "江"
    }
  }
}
```
![](/assets/21.png)