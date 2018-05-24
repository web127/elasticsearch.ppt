# ElasticSearch语法
### 索引操作

查看集群中索引？
```
GET /_cat/indices?v
```
创建索引？
```
PUT /index?pretty
```

删除索引
```
DELETE /index?pretty
```
### 文档操作

新增文档

```
PUT /index/type/id
{
"json数据"
}
```
es默认会对document每个field都建立倒排索引，让其可以被搜索

查询文档

```
GET /index/type/id
```

修改文档

```
PUT /index/type/id
{
"json数据"
}
```

更新文档

```
POST /index/type/id/_update
{
"doc": {
"field":""
}
}
```

删除文档

```
DELETE /index/type/id
```

查询所有文档
```
GET /index/type/_search
{
"query": { "match_all": {} }
}
```
match 查询名称包含xxx的文档，并按价格降序排序

```
GET /index/type/_search
{
    "query" : {
        "match" : {
            "name" : "xxx"
        }
    },
    "sort": [
        { "price": "desc" }
    ]
}
```

match_phrase 短语搜索文档
>要求输入的搜索词，必须在指定的字段文本中，完全包含一模一样的，才可以作为结果返回
```
GET /index/type/_search
{
    "query" : {
        "match_phrase" : {
            "producer" : "producer"
        }
    }
}
```

highlight 高亮搜索文档，会将搜索词高亮显示

```
GET /index/type/_search
{
    "query" : {
        "match" : {
            "producer" : "producer"
        }
    },
    "highlight": {
        "fields" : {
            "producer" : {}
        }
    }
}
```

分页查询文档

```
GET /index/type/_search
{
  "query": { "match_all": {} },
  "from": 1,
  "size": 1
}
```
指定某些字段查询文档

```

GET /index/type/_search
{
  "query": { "match_all": {} },
  "_source": ["name", "price"]
}
```



filter 过滤查询价格大于50的文档

```
GET /index/type/_search
{
    "query" : {
        "bool" : {
            "must" : {
                "match" : {
                    "name" : "xxx" 
                }
            },
            "filter" : {
                "range" : {
                    "price" : { "gt" : 50 } 
                }
            }
        }
    }
}
```

